# Java 17 vs Java 21: diferencias que sí importan al aprender

## Objetivo
Este complemento explica las diferencias reales entre Java 17 y Java 21 para que sepas qué cambia a nivel de lenguaje, runtime, rendimiento y estilo de programación. La idea no es memorizar novedades aisladas, sino entender cuándo usar cada versión y por qué Java 21 suele ser una mejor base formativa cuando ya se domina Java 17.

## 1. Qué tienen en común
Java 17 y Java 21 son versiones LTS. Eso significa que ambas reciben soporte prolongado y son buenas bases para estudiar, desarrollar y mantener proyectos.

En la práctica, compartirás gran parte del mismo ecosistema:
- Sintaxis base del lenguaje: clases, interfaces, herencia, encapsulación, polimorfismo.
- API estándar de Java: colecciones, excepciones, I/O, JDBC, streams, fechas.
- Compilación con Maven o Gradle.
- JVM, garbage collector, classpath, módulos y herramientas de línea de comandos.

Conclusión: aprender Java 17 no es aprender un lenguaje distinto de Java 21. Es aprender la misma base con mejoras acumuladas.

## 2. Qué aporta Java 21 sobre Java 17
Java 21 no rompe la compatibilidad con la mayoría del código Java 17, pero sí agrega capacidades que modernizan la forma de escribir programas.

### 2.1. Virtual Threads
Es la novedad más importante para entender concurrencia moderna.

Los hilos clásicos de Java son hilos del sistema operativo. Funcionan bien, pero cada uno tiene más coste de memoria y de planificación. Cuando una aplicación tiene muchísimas tareas bloqueantes, como llamadas a BD o red, los hilos tradicionales pueden convertirse en un cuello de botella.

Los virtual threads son hilos gestionados por la JVM que permiten manejar muchísimas más tareas concurrentes con menos coste. No sustituyen la lógica de programación, pero hacen más escalable el modelo “un hilo por tarea”.

Ideas clave:
- Un virtual thread sigue ejecutando código Java normal.
- Si una tarea se bloquea esperando I/O, la JVM puede suspenderla sin ocupar un hilo pesado del sistema operativo todo el tiempo.
- Son especialmente útiles para servidores web, accesos a BD y procesamiento concurrente de muchas solicitudes.

Ejemplo conceptual:
```java
try (var executor = java.util.concurrent.Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1000; i++) {
        int taskId = i;
        executor.submit(() -> procesarSolicitud(taskId));
    }
}
```

Lo importante aquí no es el código, sino la idea: Java 21 hace mucho más natural la concurrencia masiva.

### 2.2. Pattern Matching más maduro
Java 21 consolida varias mejoras de pattern matching que en Java 17 no estaban tan desarrolladas.

Esto permite escribir código más expresivo cuando evalúas tipos o estructuras. Es especialmente útil en `switch` más modernos y en transformaciones de objetos.

Qué ganas:
- Menos `instanceof` seguido de cast manual.
- Código más directo y legible.
- Mejor modelado de casos cuando trabajas con jerarquías.

### 2.3. Record Patterns y switch más expresivos
Java 21 mejora el manejo de desestructuración de objetos en ciertos escenarios. Esto se nota sobre todo cuando tienes modelos de datos inmutables y quieres leer sus partes de forma clara.

Si vienes de Java 17, piensa en esto como una evolución natural de `record` y `switch` más potente.

### 2.4. Secuencias y mejoras de API
Java 21 incluye pequeñas mejoras de API y runtime que hacen más cómodo el trabajo diario. No siempre son la razón principal para migrar, pero sí mejoran la experiencia general.

## 3. Qué deberías enseñar primero
Si una persona empieza desde cero, conviene este orden:
1. Java 17 como base conceptual.
2. Sintaxis, POO, colecciones, excepciones, I/O, JDBC.
3. Streams, lambdas, `Optional`, `enum`, `record`.
4. Concurrencia básica con `Thread`, `Runnable`, `ExecutorService`.
5. Concurrencia moderna en Java 21 con virtual threads.

Así evitas que alguien aprenda primero las novedades y luego no entienda el fundamento que las sostiene.

## 4. Diferencia práctica entre estudiar en 17 o 21
### Si estudias con Java 17
Ventajas:
- Es una base muy estable.
- Es compatible con mucho software empresarial existente.
- Tiene un ecosistema maduro.

Limitaciones:
- No tendrás el salto más moderno en concurrencia.
- Algunas técnicas nuevas del lenguaje estarán ausentes o menos desarrolladas.

### Si estudias con Java 21
Ventajas:
- Aprendes sobre la base LTS más reciente.
- Puedes introducir virtual threads y explicaciones más modernas de concurrencia.
- El material queda más alineado con desarrollo actual.

Limitaciones:
- Algunas novedades requieren más contexto para no confundir a alguien que empieza.

## 5. Recomendación pedagógica para este curso
La mejor estrategia es:
- Enseñar el curso principal con compatibilidad Java 17 y 21.
- Usar Java 17 para explicar la base común.
- Reservar Java 21 para temas modernos como virtual threads y mejoras de expresión.

Eso evita mezclar fundamentos con novedades demasiado pronto.

## 6. Resumen rápido
- Java 17 y Java 21 comparten casi toda la base de aprendizaje.
- Java 21 agrega mejoras muy importantes en concurrencia y expresividad.
- Si el curso quiere ser actual, Java 21 debe aparecer como referencia preferente.
- Si el objetivo es enseñar bien, no basta con listar novedades: hay que explicar qué problema resuelven.

## 7. Ruta sugerida
- Empezar con Java 17 para fundamentos.
- Añadir Java 21 como evolución natural.
- Usar ejercicios comparativos para mostrar qué cambia y qué no cambia.
- Relacionar las novedades con problemas reales: hilos, rendimiento, legibilidad y escalabilidad.
