# Ruta de aprendizaje de Java: de cero a experto

Esta guía convierte el repositorio en un currículo. “Experto” no significa memorizar toda la API: significa comprender el modelo de ejecución, diseñar con criterio, diagnosticar problemas y aprender una biblioteca nueva leyendo su documentación.

## Antes de empezar

Necesitas un JDK, un editor o IDE, Git y Maven. Comprueba:

```bash
java --version
javac --version
mvn --version
git --version
```

Usa Java 21 LTS como base didáctica. Java 17 sigue siendo válido para entornos corporativos; Java 25 LTS se estudia en el bloque de evolución. No actives características preview al comenzar.

## Nivel 0 — Modelo mental y herramientas

### Aprende

- Diferencia entre lenguaje Java, Java SE, JDK, JVM y JRE.
- De `.java` a bytecode `.class`, carga de clases, JIT y ejecución.
- Terminal, variables de entorno, paquetes y classpath.
- Estructura Maven: `src/main/java`, `src/test/java` y `pom.xml`.
- Cómo leer un error de compilación y un stack trace.

### Estudia

1. [Fundamentos de Java](COMPLEMENTOS/README_Fundamentos_Java.md)
2. [Configuración inicial](SESIONES/Semana1/README_Sesion_S1Dia1.md)
3. [Variables de entorno](COMPLEMENTOS/README_VariablesEntorno.md)
4. [Maven y Gradle](SESIONES/Semana2/README_Sesion_S2Dia1.md)

### Puedes avanzar si

- Compilas y ejecutas una clase desde el IDE y desde terminal.
- Explicas por qué `javac` y `java` no son el mismo comando.
- Sabes localizar la primera línea relevante de un stack trace.

## Nivel 1 — Sintaxis y resolución de problemas

### Aprende

- Variables, tipos primitivos, referencias, conversiones y alcance.
- Operadores, precedencia, `if`, `switch`, `for`, `while` y `break`.
- Métodos, parámetros, retorno, sobrecarga y paso por valor.
- `String`, igualdad con `equals`, `StringBuilder` e inmutabilidad.
- Arrays, matrices y complejidad básica.
- Colecciones: `List`, `Set`, `Map`, `Queue` e iteración.

### Estudia

1. [Fundamentos de Java](COMPLEMENTOS/README_Fundamentos_Java.md)
2. [Arrays, matrices y colecciones](COMPLEMENTOS/README_Arrays.md)
3. [Actividad POO y arrays](SESIONES/Semana2/README_Sesion_S2Dia3.md)

### Práctica mínima

- Validador de notas con entrada de consola.
- Estadísticas de un array sin Streams.
- Conteo de palabras con `Map<String, Integer>`.
- Eliminación de duplicados justificando el tipo de `Set`.

## Nivel 2 — Programación orientada a objetos

### Aprende

- Identidad, estado, comportamiento, invariantes y ciclo de vida.
- Encapsulación real: operaciones de negocio, no setters indiscriminados.
- Abstracción mediante contratos y modelos que omiten detalles irrelevantes.
- Polimorfismo de subtipos, sobreescritura y despacho dinámico.
- Herencia frente a composición y el principio de sustitución.
- Interfaces, clases abstractas, enums, records y objetos de valor.
- Acoplamiento, cohesión, dependencias e inyección por constructor.

### Estudia

1. [POO explicada en profundidad](SESIONES/Semana2/README_Sesion_S2Dia2.md)
2. [Proyecto académico con POO](SESIONES/Semana2/README_Sesion_S2Dia3.md)
3. [Interfaces y repositorios intercambiables](SESIONES/Semana4/README_Sesion_S4Dia1.md)
4. [DTO, Mapper y componentes por capas](COMPLEMENTOS/README_Componentes_JAVA.md)

### Puedes avanzar si

- Una cuenta bancaria nunca puede quedar inválida aunque alguien use su API pública.
- Puedes reemplazar un repositorio en memoria por otro JDBC sin modificar el servicio.
- Sabes explicar por qué `Cuadrado extends Rectangulo` puede romper sustitución según el modelo mutable.

## Nivel 3 — Genéricos, lambdas y Streams

### Aprende

- Genéricos, límites, comodines y regla PECS.
- Interfaces funcionales: `Predicate`, `Function`, `Consumer`, `Supplier`.
- Lambdas, referencias a métodos, captura y variables efectivamente finales.
- Pipeline de Stream, evaluación perezosa y operaciones terminales.
- `map` frente a `flatMap`, reducción, collectors y agrupaciones.
- `Optional` como resultado posiblemente ausente, no como reemplazo universal de `null`.

### Estudia

1. [Streams y programación funcional](COMPLEMENTOS/README_Streams_Java.md)
2. [Records y modelado moderno](COMPLEMENTOS/README_Records_JavaModerno.md)

### Práctica mínima

- Agrupar estudiantes por estado y calcular promedio por grupo.
- Convertir pedidos con líneas anidadas a un resumen por producto usando `flatMap`.
- Escribir la misma transformación con bucle y con Stream; comparar claridad.

## Nivel 4 — Robustez, archivos y pruebas

### Aprende

- Checked y unchecked exceptions, traducción por capas y causa original.
- `try-with-resources`, `AutoCloseable` y supresión de excepciones.
- API `Path`/`Files`, codificación de texto y serialización controlada.
- Pruebas unitarias, pirámide de pruebas, dobles de prueba y casos límite.
- Logging, configuración y diferencia entre registrar y manejar un error.
- Javadoc enfocado en contratos, precondiciones y efectos.

### Estudia

1. [Manejo completo de excepciones](COMPLEMENTOS/README_ManejoExcepciones.md)
2. [Excepciones aplicadas por capas](SESIONES/Semana5/README_Sesion_S4Dia2.md)
3. [Códigos de error con enums](SESIONES/Semana5/README_Sesion_S4Dia3.md)
4. [Pruebas automatizadas con JUnit](COMPLEMENTOS/README_Pruebas_Java.md)
5. [Javadoc](COMPLEMENTOS/README_DocumentacionJavaDoc.md)
6. [JAR ejecutable](COMPLEMENTOS/README_GenerarEjecutable.md)

## Nivel 5 — Persistencia y arquitectura

### Aprende

- SQL parametrizado, `PreparedStatement`, mapeo de filas y recursos JDBC.
- Transacciones, atomicidad, commit, rollback y límites en el servicio.
- DAO/Repository, Service, DTO, Mapper, UI y dirección de dependencias.
- MVC frente a arquitectura por capas; patrón de diseño frente a arquitectura.
- SOLID como heurísticas con costos, no como reglas mecánicas.

### Estudia

1. [CRUD con JDBC](SESIONES/Semana3/README_Sesion_S3Dia2.md)
2. [Mini-tienda integradora](SESIONES/Semana3/README_Sesion_S3Dia3.md)
3. [MVC y patrones de arquitectura](SESIONES/Semana4/README_Sesion_S4Dia2.md)
4. [Arquitectura por capas](SESIONES/Semana5/README_Sesion_S4Dia1.md)
5. [Arquitectura final con contratos](FINAL/README_Final_Arquitectura.md)

## Nivel 6 — Diseño avanzado y concurrencia

### Aprende

- Patrones como vocabulario contextual; problema, fuerzas y consecuencias.
- Mutabilidad, Java Memory Model, condiciones de carrera y visibilidad.
- `ExecutorService`, `Future`, `CompletableFuture` y colecciones concurrentes.
- Hilos virtuales para alta concurrencia con I/O bloqueante.
- Medición con benchmarks y profiling antes de optimizar.
- JVM: heap, stack, garbage collection, class loading y JIT a nivel conceptual.

### Estudia

1. [Patrones de diseño](SESIONES/Semana3/README_Sesion_S3Dia1.md)
2. [Hilos y concurrencia](COMPLEMENTOS/README_Hilos_y_Concurrencia.md)
3. [Versiones y evolución de Java](COMPLEMENTOS/README_Versiones_Java.md)

## Nivel 7 — Ecosistema empresarial

Primero domina Java SE. Después compara plataformas, porque Spring Boot y Jakarta EE resuelven composición, configuración, web, persistencia, observabilidad y operación; no sustituyen los fundamentos del lenguaje.

### Estudia

1. [Puente a Spring Boot](COMPLEMENTOS/README_Puente_SpringBoot.md)
2. [Java SE, Jakarta EE y Spring Boot](COMPLEMENTOS/README_JAVAEE_SPRINGB.md)
3. [Arquitectura Java EE clásica](COMPLEMENTOS/README_JAVAEE.md)
4. [WildFly y Jakarta EE](COMPLEMENTOS/README_JAVAEE_Wildfly.md)

El material que usa el nombre histórico “Java EE” debe leerse entendiendo que la evolución moderna se llama **Jakarta EE** y que desde Jakarta EE 9 los paquetes principales cambiaron de `javax.*` a `jakarta.*`.

## Mapa del contenido existente

| Ubicación | Papel dentro de la ruta | Momento recomendado |
|---|---|---|
| `SESIONES/Semana1` | Setup, primer modelo, Swing e historias de usuario | Niveles 0–2 |
| `SESIONES/Semana2` | Build tools, POO y ejercicio con arrays | Niveles 1–2 |
| `SESIONES/Semana3` | Patrones y JDBC | Niveles 5–6 |
| `SESIONES/Semana4` | Interfaces, MVC y diseño | Niveles 2 y 5 |
| `SESIONES/Semana5` | Capas y excepciones | Niveles 4–5 |
| `COMPLEMENTOS` | Profundizaciones transversales | Según enlace de cada nivel |
| `PRACTICA` | Requisitos del ejercicio integrador | Después del nivel 5 |
| `FINAL` | Arquitectura de referencia completa | Cierre del nivel 5 |

> Nota de estructura: los tres documentos dentro de `Semana5` conservan `S4` en el nombre por compatibilidad histórica, pero curricularmente pertenecen a Semana 5.

## Qué distingue a un nivel experto

Un desarrollador experto no elige una herramienta porque sea moderna. Puede:

- explicar las restricciones y alternativas de una decisión;
- detectar invariantes, límites transaccionales y fuentes de acoplamiento;
- leer bytecode, thread dumps, planes de ejecución o perfiles cuando el problema lo exige;
- separar un error de lenguaje, diseño, datos, concurrencia o infraestructura;
- migrar entre versiones entendiendo compatibilidad de fuente, binaria y de ejecución;
- hacer que el camino correcto sea el más fácil mediante API, pruebas y automatización.

## Regla de progreso

Por cada tema produce cuatro evidencias: una explicación breve, un ejemplo ejecutable, una prueba automatizada y una refactorización. Si solo puedes reconocer el código, todavía no dominas el tema.
