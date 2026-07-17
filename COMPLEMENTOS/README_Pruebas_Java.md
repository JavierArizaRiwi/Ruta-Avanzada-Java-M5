# Pruebas automatizadas en Java con JUnit

## 1. Qué demuestra una prueba

Una prueba ejecutable aporta evidencia de que un comportamiento cumple un contrato bajo condiciones concretas. No demuestra ausencia total de errores. Su valor está en:

- detectar regresiones;
- aclarar el comportamiento esperado;
- permitir refactorizar con seguridad;
- obligar a controlar dependencias y efectos;
- acortar el diagnóstico de fallos.

Prueba comportamiento observable, no líneas privadas. Una prueba que se rompe al renombrar un método privado, sin cambiar comportamiento, está demasiado acoplada a la implementación.

## 2. Vocabulario actual de JUnit

- **JUnit Platform:** descubre y ejecuta motores de pruebas.
- **JUnit Jupiter:** API y motor usados para escribir pruebas modernas.
- **JUnit Vintage:** compatibilidad para pruebas antiguas JUnit 3/4; úsalo solo durante migraciones.

JUnit 6 requiere Java 17 o superior en runtime. Esta ruta usa JUnit 6.1.2 como referencia fechada en julio de 2026; verifica la versión estable antes de iniciar un proyecto nuevo en la [guía oficial](https://docs.junit.org/current/user-guide/).

## 3. Configuración Maven

Usa el BOM para alinear los módulos JUnit:

```xml
<properties>
    <maven.compiler.release>21</maven.compiler.release>
    <junit.version>6.1.2</junit.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.junit</groupId>
            <artifactId>junit-bom</artifactId>
            <version>${junit.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

JUnit 6 necesita Maven Surefire/Failsafe 3.0.0 o posterior. Fija una versión reciente en proyectos reproducibles siguiendo el [soporte de build oficial](https://docs.junit.org/current/running-tests/build-support.html).

Estructura:

```text
src/main/java/com/academia/Promedio.java
src/test/java/com/academia/PromedioTest.java
```

Ejecuta:

```bash
mvn test
```

## 4. Primera prueba: Arrange, Act, Assert

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

class CalculadoraTest {

    @Test
    void sumaDosEnteros() {
        // Arrange
        var calculadora = new Calculadora();

        // Act
        int resultado = calculadora.sumar(2, 3);

        // Assert
        assertEquals(5, resultado);
    }
}
```

AAA ayuda al comenzar, pero no requiere comentarios si el código ya se lee con claridad.

Un nombre útil describe escenario y resultado:

```java
@Test
void retirar_rechazaMontoMayorAlSaldo() {}
```

Evita `test1`, `funciona` o nombres que repiten el método sin expresar la regla.

## 5. Aserciones fundamentales

```java
assertEquals(esperado, real);
assertTrue(resultado.isPresent());
assertFalse(cuenta.estaBloqueada());
assertNull(valor);
assertNotNull(valor);
assertIterableEquals(esperados, reales);
```

Para decimales aproximados:

```java
assertEquals(0.3, resultado, 0.000_001);
```

Para dinero, compara según el contrato. `BigDecimal.equals` considera escala (`10.0` no es igual a `10.00`), mientras `compareTo` compara valor numérico:

```java
assertEquals(0, esperado.compareTo(real));
```

Agrupa comprobaciones independientes cuando quieres ver todos los fallos:

```java
assertAll(
        () -> assertEquals("Ana", estudiante.nombre()),
        () -> assertEquals(3, estudiante.notas().size()),
        () -> assertTrue(estudiante.aprobo())
);
```

No pongas veinte comportamientos distintos en una sola prueba.

## 6. Probar excepciones

```java
@Test
void retirar_fallaConSaldoInsuficiente() {
    var cuenta = new Cuenta("001", new BigDecimal("100.00"));

    SaldoInsuficienteException error = assertThrows(
            SaldoInsuficienteException.class,
            () -> cuenta.retirar(new BigDecimal("150.00"))
    );

    assertEquals("001", error.numeroCuenta());
    assertEquals(new BigDecimal("100.00"), cuenta.saldo());
}
```

Además del tipo de error, verifica que el estado no cambió. No hagas una aserción frágil sobre todo el texto si el mensaje no es parte del contrato.

## 7. Pruebas parametrizadas

Evitan duplicar la misma regla con entradas distintas:

```java
@ParameterizedTest(name = "nota {0} -> aprobado {1}")
@CsvSource({
        "0.0, false",
        "2.99, false",
        "3.0, true",
        "5.0, true"
})
void determinaAprobacion(double nota, boolean esperado) {
    assertEquals(esperado, new Nota(nota).aprobada());
}
```

Para casos complejos usa `@MethodSource`:

```java
static Stream<Arguments> montosInvalidos() {
    return Stream.of(
            Arguments.of((BigDecimal) null),
            Arguments.of(BigDecimal.ZERO),
            Arguments.of(new BigDecimal("-0.01"))
    );
}
```

Incluye particiones y fronteras: mínimo, justo debajo, valor válido, máximo y justo encima.

## 8. Ciclo de vida y aislamiento

```java
class PedidoServiceTest {
    private PedidoRepositoryFake repositorio;
    private PedidoService servicio;

    @BeforeEach
    void preparar() {
        repositorio = new PedidoRepositoryFake();
        servicio = new PedidoService(repositorio);
    }
}
```

Cada prueba debe poder ejecutarse sola, en cualquier orden y repetidamente. Evita campos `static` mutables, bases compartidas sin limpieza y dependencia de la hora real.

`@BeforeAll` sirve para recursos costosos compartidos, no para esconder dependencia entre pruebas.

## 9. Fakes, stubs y mocks

- **Fake:** implementación funcional simplificada, como repositorio en memoria.
- **Stub:** devuelve respuestas preparadas.
- **Mock:** además verifica interacciones esperadas.
- **Spy:** envuelve un objeto real y registra o sustituye partes.

Empieza con un fake manual cuando el contrato es pequeño:

```java
final class EstudianteRepositoryFake implements EstudianteRepository {
    private final Map<Long, Estudiante> datos = new HashMap<>();

    @Override
    public Optional<Estudiante> buscarPorId(long id) {
        return Optional.ofNullable(datos.get(id));
    }

    @Override
    public void guardar(Estudiante estudiante) {
        datos.put(estudiante.id(), estudiante);
    }
}
```

Verifica **estado o resultado** cuando sea posible. Verifica interacciones cuando ellas formen parte del contrato, por ejemplo “no guardar si la validación falla”. Demasiados mocks hacen que la prueba copie la implementación.

## 10. Controlar tiempo, azar y servicios externos

Código difícil de probar:

```java
if (LocalDate.now().isAfter(vencimiento)) { ... }
```

Inyecta `Clock`:

```java
public final class SuscripcionService {
    private final Clock clock;

    public SuscripcionService(Clock clock) {
        this.clock = clock;
    }

    boolean vencida(LocalDate fecha) {
        return LocalDate.now(clock).isAfter(fecha);
    }
}
```

En la prueba:

```java
Clock fijo = Clock.fixed(
        Instant.parse("2026-07-17T12:00:00Z"),
        ZoneOffset.UTC);
```

Aplica el mismo principio a UUID, azar, red, archivos y correo: coloca el efecto detrás de una dependencia controlable.

## 11. Archivos temporales

```java
@Test
void exportaCsv(@TempDir Path directorio) throws IOException {
    Path destino = directorio.resolve("pedidos.csv");

    exportador.exportar(destino, pedidos);

    List<String> lineas = Files.readAllLines(destino, StandardCharsets.UTF_8);
    assertEquals("id,cliente,total", lineas.getFirst());
}
```

`@TempDir` aísla la prueba y limpia recursos. No escribas en rutas absolutas ni dentro del repositorio.

## 12. Unitarias, integración y end-to-end

| Tipo | Alcance | Velocidad | Ejemplo |
|---|---|---|---|
| Unitaria | una regla con dependencias controladas | muy alta | total de pedido |
| Integración | colaboración con tecnología real | media | repository contra base de prueba |
| End-to-end | flujo desplegado completo | baja | petición HTTP hasta base de datos |

Una prueba con JDBC real no es unitaria aunque pruebe una sola clase. El nombre describe dependencias y aislamiento, no la cantidad de clases.

La mayoría debe ser rápida y determinista; conserva menos pruebas costosas para fronteras críticas.

## 13. Qué probar

Prioriza:

- reglas de negocio e invariantes;
- límites y particiones de entrada;
- traducción de errores;
- transacciones y rollback;
- serialización y contratos públicos;
- consultas con mapeo complejo;
- errores que ya ocurrieron: cada bug corregido merece una prueba de regresión.

No suele aportar probar getters triviales, código de framework o detalles privados sin comportamiento.

## 14. Tests deterministas

Una prueba flaky falla a veces sin cambio de código. Causas comunes:

- tiempo real y zonas horarias;
- orden no garantizado de `HashMap`/`HashSet`;
- puertos, red o archivos compartidos;
- concurrencia coordinada con `Thread.sleep`;
- datos de base no aislados;
- locale y codificación implícitos.

Corrige la causa; no agregues reintentos para ocultarla. En concurrencia sincroniza con primitivas explícitas y condiciones observables, no con esperas arbitrarias.

## 15. Cobertura y calidad

La cobertura indica qué código ejecutaron las pruebas, no si las aserciones son buenas. 100 % de líneas puede ignorar requisitos enteros.

Usa cobertura para encontrar zonas no ejercitadas, junto con:

- calidad de casos límite;
- mutation testing cuando el proyecto lo justifique;
- velocidad y estabilidad de la suite;
- legibilidad y diagnóstico del fallo;
- riesgo del comportamiento cubierto.

## 16. Diseño guiado por pruebas

El ciclo TDD es:

```text
Rojo: escribir una prueba que falla por la razón esperada
Verde: implementar lo mínimo correcto
Refactor: mejorar diseño manteniendo la suite verde
```

TDD es una herramienta de diseño y feedback, no una obligación para cada línea. Funciona especialmente bien en reglas, parsers y algoritmos; para exploración visual o integración desconocida puede convenir un spike y luego pruebas.

## 17. Errores comunes

1. Compartir estado entre pruebas.
2. Usar solo el camino feliz.
3. Atrapar la excepción dentro de la prueba y olvidar fallar.
4. Comparar decimales o fechas sin semántica clara.
5. Mockear cada objeto, incluso valores simples.
6. Probar métodos privados mediante reflexión.
7. Depender del orden natural de una colección no ordenada.
8. Dormir hilos para “esperar” asincronía.
9. Ejecutar tests solo desde el IDE y no desde Maven/CI.
10. Medir éxito solo por porcentaje de cobertura.

## 18. Checklist

- [ ] La prueba expresa escenario y resultado.
- [ ] Se ejecuta sola, rápido y repetidamente.
- [ ] Cubre fronteras y fallos, no solo camino feliz.
- [ ] Controla tiempo, azar, red y archivos.
- [ ] Comprueba que el estado permanece válido tras un error.
- [ ] Distingue unitarias de integración.
- [ ] La suite corre con `mvn test` en un entorno limpio.
- [ ] Una refactorización interna no rompe pruebas de comportamiento.
