# Programación orientada a objetos en Java: de conceptos a buen diseño

## Objetivo

Comprender la POO como una forma de modelar responsabilidades e interacciones, no como una lista de palabras clave. Al terminar podrás diseñar objetos válidos, elegir entre composición, herencia e interfaces y usar polimorfismo sin crear jerarquías artificiales.

## 1. Antes de los cuatro principios: objeto, clase e identidad

Una **clase** define un tipo. Un **objeto** es una instancia concreta creada durante la ejecución.

```java
public final class Cuenta {
    private final String numero;
    private BigDecimal saldo;

    public Cuenta(String numero, BigDecimal saldoInicial) {
        this.numero = Objects.requireNonNull(numero);
        if (saldoInicial.signum() < 0) {
            throw new IllegalArgumentException("El saldo inicial no puede ser negativo");
        }
        this.saldo = saldoInicial;
    }
}
```

Un objeto combina:

- **estado:** valores que conserva, como `numero` y `saldo`;
- **comportamiento:** operaciones permitidas, como `depositar` y `retirar`;
- **identidad:** dos cuentas pueden tener el mismo saldo y seguir siendo cuentas distintas;
- **invariantes:** reglas que siempre deben cumplirse, como saldo no negativo.

Una buena clase hace difícil crear o dejar un objeto en estado inválido.

## 2. Encapsulación: proteger invariantes

Encapsular no es añadir getters y setters a cada campo. Es mantener datos y reglas dentro de una frontera y ofrecer operaciones que preservan la validez.

### Diseño débil

```java
public class Cuenta {
    public BigDecimal saldo;
}
```

Cualquier consumidor puede asignar un saldo negativo o `null`.

Un setter tampoco resuelve el modelo si expone una operación que el negocio no permite:

```java
cuenta.setSaldo(new BigDecimal("1000000")); // ¿de dónde salió ese dinero?
```

### Diseño encapsulado

```java
public final class Cuenta {
    private final String numero;
    private BigDecimal saldo;

    // constructor validado omitido

    public BigDecimal saldo() {
        return saldo;
    }

    public void depositar(BigDecimal monto) {
        validarMontoPositivo(monto);
        saldo = saldo.add(monto);
    }

    public void retirar(BigDecimal monto) {
        validarMontoPositivo(monto);
        if (saldo.compareTo(monto) < 0) {
            throw new SaldoInsuficienteException(numero, saldo, monto);
        }
        saldo = saldo.subtract(monto);
    }

    private static void validarMontoPositivo(BigDecimal monto) {
        if (monto == null || monto.signum() <= 0) {
            throw new IllegalArgumentException("El monto debe ser positivo");
        }
    }
}
```

La API habla el lenguaje del dominio. `depositar` y `retirar` explican la intención; `setSaldo` solo revela almacenamiento.

### Encapsulación de colecciones

No devuelvas una colección interna mutable:

```java
public final class Curso {
    private final List<Estudiante> estudiantes = new ArrayList<>();

    public void matricular(Estudiante estudiante) {
        Objects.requireNonNull(estudiante);
        if (estudiantes.contains(estudiante)) {
            throw new IllegalArgumentException("Ya está matriculado");
        }
        estudiantes.add(estudiante);
    }

    public List<Estudiante> estudiantes() {
        return List.copyOf(estudiantes);
    }
}
```

La copia no modificable impide que un consumidor salte `matricular` y rompa reglas.

## 3. Abstracción: modelar lo relevante

Abstraer es representar un concepto conservando los detalles relevantes y ocultando los irrelevantes para el problema actual.

`BigDecimal` abstrae una representación decimal; `List` abstrae una secuencia; una clase `Email` abstrae validación y normalización. No necesitas una interfaz o clase abstracta para que exista abstracción.

```java
public record Email(String valor) {
    public Email {
        Objects.requireNonNull(valor);
        valor = valor.trim().toLowerCase(Locale.ROOT);
        if (!valor.contains("@")) {
            throw new IllegalArgumentException("Email inválido");
        }
    }
}
```

Quien usa `Email` trabaja con un valor válido y no repite cómo se normaliza.

### Abstracción mediante contrato

Una interfaz es útil cuando varios componentes deben cumplir el mismo rol:

```java
public interface EstudianteRepository {
    Optional<Estudiante> buscarPorId(long id);
    void guardar(Estudiante estudiante);
}
```

El servicio depende de la capacidad “guardar estudiantes”, no de SQL, memoria o archivos.

```java
public final class MatriculaService {
    private final EstudianteRepository estudiantes;

    public MatriculaService(EstudianteRepository estudiantes) {
        this.estudiantes = Objects.requireNonNull(estudiantes);
    }
}
```

No crees una interfaz para cada clase por rutina. Créala donde represente un contrato útil, exista variación real o necesites aislar una frontera tecnológica.

## 4. Herencia: una relación de subtipo

Herencia significa que una instancia de la subclase puede usarse correctamente donde se espera la superclase: relación **es-un** y principio de sustitución.

```java
public abstract class Empleado {
    private final String nombre;

    protected Empleado(String nombre) {
        this.nombre = Objects.requireNonNull(nombre);
    }

    public final String nombre() {
        return nombre;
    }

    public abstract BigDecimal calcularPago();
}

public final class EmpleadoMensual extends Empleado {
    private final BigDecimal salario;

    public EmpleadoMensual(String nombre, BigDecimal salario) {
        super(nombre);
        this.salario = salario;
    }

    @Override
    public BigDecimal calcularPago() {
        return salario;
    }
}
```

Java permite extender una sola clase e implementar múltiples interfaces. Los constructores no se heredan. `super(...)` inicializa la parte de la superclase.

### Riesgos

- La subclase queda fuertemente acoplada a detalles `protected` y al ciclo de vida del padre.
- Un cambio en la superclase puede romper subclases de manera sutil.
- Jerarquías profundas dificultan saber qué implementación se ejecuta.
- Reutilizar código no demuestra una relación de subtipo.

### Composición antes que herencia

Si la relación es **tiene-un** o **usa-un**, compón:

```java
public final class FacturaService {
    private final CalculadorImpuestos impuestos;
    private final FacturaRepository repositorio;

    public FacturaService(CalculadorImpuestos impuestos,
                          FacturaRepository repositorio) {
        this.impuestos = impuestos;
        this.repositorio = repositorio;
    }
}
```

La composición permite sustituir colaboradores sin heredar estado o implementación.

### El problema cuadrado–rectángulo

Si `Rectangulo` es mutable y permite cambiar ancho y alto independientemente, hacer `Cuadrado extends Rectangulo` puede romper la expectativa de quien modifica solo un lado. Geométricamente un cuadrado es rectángulo; el modelo de objetos mutable no necesariamente conserva sustitución. El diseño depende del contrato, no solo del mundo real.

## 5. Polimorfismo: un contrato, múltiples comportamientos

El polimorfismo de subtipos permite enviar el mismo mensaje a objetos diferentes:

```java
public interface Notificador {
    void enviar(Mensaje mensaje);
}

public final class NotificadorEmail implements Notificador {
    @Override
    public void enviar(Mensaje mensaje) {
        // envío por email
    }
}

public final class NotificadorSms implements Notificador {
    @Override
    public void enviar(Mensaje mensaje) {
        // envío por SMS
    }
}
```

```java
static void notificar(Notificador canal, Mensaje mensaje) {
    canal.enviar(mensaje); // despacho dinámico según el objeto real
}
```

El tipo de la referencia decide qué operaciones puedes invocar; el tipo real del objeto decide qué método sobrescrito se ejecuta.

```java
Notificador canal = new NotificadorEmail();
canal.enviar(mensaje); // ejecuta NotificadorEmail.enviar
```

### Sobreescritura vs sobrecarga

| Concepto | Momento | Regla |
|---|---|---|
| Overriding / sobreescritura | ejecución | una subclase redefine un método heredado compatible |
| Overloading / sobrecarga | compilación | mismo nombre, lista de parámetros diferente |

Usa `@Override`: permite al compilador detectar errores de firma.

```java
void imprimir(String texto) {}
void imprimir(int numero) {} // sobrecarga
```

La sobrecarga no es despacho dinámico por el tipo real de los argumentos; el compilador elige una firma según los tipos visibles.

## 6. Interfaz o clase abstracta

| Pregunta | Interfaz | Clase abstracta |
|---|---|---|
| ¿Expresa un rol que tipos no relacionados pueden cumplir? | Sí | No suele ser la primera opción |
| ¿Necesita estado de instancia compartido? | No | Sí |
| ¿Una clase debe asumir varios roles? | Sí, implementa varias | Solo puede extender una |
| ¿Hay algoritmo base y pasos variables? | Posible con `default`, pero limitado | Adecuada |
| ¿Es frontera de arquitectura? | Muy adecuada | Depende |

Los métodos `default` evolucionan interfaces y comparten comportamiento pequeño, pero no deben convertir una interfaz en una clase base disfrazada.

## 7. Visibilidad y paquetes

| Modificador | Acceso |
|---|---|
| `private` | solo la clase |
| sin modificador | mismo paquete |
| `protected` | mismo paquete y subclases con reglas específicas |
| `public` | cualquier consumidor que pueda acceder al paquete/módulo |

Usa la visibilidad mínima necesaria. `public` amplía el contrato que tendrás que mantener. Organiza paquetes por responsabilidad o característica, no como contenedores arbitrarios.

## 8. `static`, instancia y dependencia global

Un miembro de instancia pertenece a un objeto; uno `static` pertenece a la clase.

Buenos usos de `static`:

- constantes inmutables;
- fábricas sin estado;
- funciones puras de utilidad;
- punto de entrada `main`.

Evita estado global mutable `static`: acopla pruebas, orden de ejecución y concurrencia.

```java
public final class Conversor {
    private Conversor() {}

    public static BigDecimal porcentaje(BigDecimal base, BigDecimal tasa) {
        return base.multiply(tasa);
    }
}
```

## 9. Igualdad, identidad y hash

`==` compara identidad de referencias; `equals` expresa igualdad lógica.

Si sobrescribes `equals`, sobrescribe `hashCode` con los mismos campos. Dos objetos iguales deben tener igual hash. Los records los generan usando todos sus componentes.

Entidades con ID generado requieren cuidado: el ID cambia de `null` a asignado y puede volver inestable el hash. Define la igualdad según el ciclo de vida y no guardes en `HashSet` objetos cuyos campos de igualdad cambiarán.

## 10. Inmutabilidad

Un objeto inmutable no cambia su estado observable después de construirse. Beneficios:

- razonar y probar es más sencillo;
- compartir entre hilos es más seguro;
- funciona bien como clave;
- reduce estados intermedios inválidos.

Para una clase inmutable:

- campos privados y finales;
- sin setters;
- valida en construcción;
- copia defensivamente valores mutables;
- evita que subclases introduzcan mutabilidad (`final` o control de extensión).

Un record es superficialmente inmutable, no profundamente inmutable. Consulta [Records y Java moderno](../../COMPLEMENTOS/README_Records_JavaModerno.md).

## 11. Relaciones entre objetos

- **Asociación:** un servicio usa un repositorio.
- **Agregación:** un equipo agrupa jugadores que pueden existir aparte.
- **Composición:** una orden posee líneas cuyo ciclo de vida pertenece a la orden.
- **Herencia:** un subtipo cumple todo el contrato del tipo base.

En código Java, agregación y composición pueden verse igual como campos; la diferencia está en propiedad, invariantes y ciclo de vida.

## 12. POO y SOLID no son lo mismo

Abstracción, encapsulación, herencia y polimorfismo describen mecanismos/ideas de POO. SOLID son heurísticas de diseño:

- **SRP:** una unidad tiene una razón coherente para cambiar.
- **OCP:** se puede extender comportamiento estable sin editar muchos consumidores.
- **LSP:** un subtipo cumple expectativas del contrato base.
- **ISP:** consumidores no dependen de operaciones que no necesitan.
- **DIP:** políticas de alto nivel no dependen directamente de detalles volátiles.

No cuentes métodos ni crees capas vacías para “cumplir SOLID”. Evalúa acoplamiento, cohesión, volatilidad y costo real.

## 13. Ejemplo integrado

```java
public interface PoliticaDescuento {
    BigDecimal aplicar(BigDecimal subtotal);
}

public record SinDescuento() implements PoliticaDescuento {
    @Override
    public BigDecimal aplicar(BigDecimal subtotal) {
        return subtotal;
    }
}

public record DescuentoPorcentaje(BigDecimal tasa) implements PoliticaDescuento {
    public DescuentoPorcentaje {
        if (tasa.signum() < 0 || tasa.compareTo(BigDecimal.ONE) > 0) {
            throw new IllegalArgumentException("Tasa fuera de rango");
        }
    }

    @Override
    public BigDecimal aplicar(BigDecimal subtotal) {
        return subtotal.multiply(BigDecimal.ONE.subtract(tasa));
    }
}

public final class Pedido {
    private final List<LineaPedido> lineas = new ArrayList<>();
    private PoliticaDescuento descuento;

    public Pedido(PoliticaDescuento descuento) {
        this.descuento = Objects.requireNonNull(descuento);
    }

    public void agregar(Producto producto, int cantidad) {
        if (cantidad <= 0) throw new IllegalArgumentException("Cantidad positiva requerida");
        lineas.add(new LineaPedido(producto, cantidad));
    }

    public BigDecimal total() {
        BigDecimal subtotal = lineas.stream()
                .map(LineaPedido::subtotal)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
        return descuento.aplicar(subtotal);
    }
}
```

Aquí hay encapsulación de líneas, abstracción mediante `PoliticaDescuento`, polimorfismo en `aplicar` y composición entre pedido, líneas y política. No fue necesaria herencia de clases.

## 14. Antipatrones frecuentes

- Modelo anémico: solo getters/setters y todas las reglas en un “service dios”.
- Herencia por reutilización: subtipos que no cumplen el contrato del padre.
- Campo público mutable.
- Constructor que permite estado inválido.
- Una interfaz por clase sin frontera ni variación.
- `instanceof` repetido donde el polimorfismo expresaría mejor el comportamiento.
- Jerarquía profunda para evitar tres líneas duplicadas.
- Dependencias creadas con `new` dentro del servicio, imposibles de sustituir.

## 15. Ejercicios de dominio

1. Diseña `CuentaBancaria` sin `setSaldo` y prueba todos sus límites.
2. Modela envíos con una estrategia de costo intercambiable.
3. Refactoriza una jerarquía `Ave` donde no todas vuelan; evita que `Pinguino` herede una operación inválida.
4. Crea `Email` y `Dinero` como objetos de valor inmutables.
5. Implementa repositorios en memoria y JDBC detrás de la misma interfaz.
6. Explica qué principio rompe cada antipatrón de la sección anterior.

## Checklist de dominio

- [ ] La API pública expresa operaciones del dominio, no solo acceso a campos.
- [ ] Los constructores establecen invariantes.
- [ ] Las colecciones internas no se filtran como mutables.
- [ ] Uso herencia solo si existe sustitución real.
- [ ] Prefiero composición para ensamblar comportamiento.
- [ ] Distingo sobrecarga de sobreescritura.
- [ ] `equals` y `hashCode` son coherentes.
- [ ] Las interfaces representan contratos útiles.
- [ ] Puedo probar la lógica sin interfaz gráfica ni base de datos.
