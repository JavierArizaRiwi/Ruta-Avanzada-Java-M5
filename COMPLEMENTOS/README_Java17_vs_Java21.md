# Java 17 vs Java 21: diferencias prácticas

Esta comparación breve se conserva para quienes deben elegir entre estas dos LTS. Para la evolución completa hasta Java 25 consulta [Versiones de Java](README_Versiones_Java.md).

## Resumen

| Tema | Java 17 | Java 21 |
|---|---|---|
| LTS | Sí, desde septiembre de 2021 | Sí, desde septiembre de 2023 |
| Records | Sí, finales desde Java 16 | Sí |
| Sealed classes | Sí, finales | Sí |
| Pattern matching `instanceof` | Sí | Sí |
| Pattern matching `switch` | Preview limitado en versiones posteriores a 17 | Final |
| Record patterns | No | Final |
| Virtual threads | No | Final |
| Sequenced collections | No | Sí |

## Qué no cambia

POO, genéricos, colecciones, lambdas, Streams, `Optional`, archivos, JDBC y módulos se aprenden con cualquiera. Java 21 es compatible con la enorme mayoría del código fuente de Java 17.

## Cuándo elegir Java 17

- Tu empresa, servidor o framework lo exige.
- Mantienes un sistema cuya matriz certificada todavía termina en 17.
- Necesitas producir bytecode 17 y no usarás APIs posteriores.

Compilar desde un JDK posterior con objetivo 17:

```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
</properties>
```

## Cuándo elegir Java 21

- Empiezas la ruta sin una restricción externa.
- Necesitas hilos virtuales para muchas tareas con I/O bloqueante.
- Quieres pattern matching y record patterns finales.
- Tus dependencias y entorno de despliegue ya están certificados.

## Virtual threads

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    var tareas = urls.stream()
            .map(url -> executor.submit(() -> descargar(url)))
            .toList();

    for (var tarea : tareas) {
        System.out.println(tarea.get());
    }
}
```

Son adecuados para gran cantidad de tareas que pasan tiempo esperando red, archivos o JDBC. No aceleran trabajo CPU-bound y no deben agruparse en un pool: se crea un hilo virtual por tarea. Fuente: [JEP 444](https://openjdk.org/jeps/444).

## Pattern matching

En Java 17:

```java
if (valor instanceof String texto) {
    System.out.println(texto.length());
}
```

En Java 21, un `switch` puede trabajar con tipos y records de forma exhaustiva:

```java
static String describir(Figura figura) {
    return switch (figura) {
        case Circulo(var radio) -> "Círculo de radio " + radio;
        case Rectangulo(var ancho, var alto) -> ancho + " x " + alto;
    };
}
```

## Recomendación didáctica

Usa Java 21 LTS para el curso, pero señala cada característica posterior a 17. Así los ejemplos modernos compilan sin flags preview y el estudiante aprende a leer proyectos corporativos en 17.

Java 25 es también LTS desde septiembre de 2025. Si vas a iniciar un proyecto real, evalúala con la matriz de compatibilidad del stack; para aprender, Java 21 sigue siendo una base estable y menos distractora. Consulta la [hoja de ruta oficial de soporte](https://www.oracle.com/java/technologies/java-se-support-roadmap.html).
