# Versiones de Java: de Java 8 a Java 25

## 1. Cómo se versiona Java

Desde Java 9 el JDK publica una versión de características aproximadamente cada seis meses. Una versión **LTS** recibe soporte prolongado de proveedores; no significa que el lenguaje sea distinto ni que las versiones intermedias sean betas.

A julio de 2026, las LTS modernas de referencia son Java 8, 11, 17, 21 y 25. Java 26 es la versión no LTS actual; Java 27 está planificada para septiembre de 2026. Las fechas y el significado comercial del soporte varían por proveedor.

Fuentes oficiales:

- [Oracle Java SE Support Roadmap](https://www.oracle.com/java/technologies/java-se-support-roadmap.html)
- [OpenJDK: JDK 21](https://openjdk.org/projects/jdk/21/)
- [OpenJDK: JDK 25](https://openjdk.org/projects/jdk/25/)

## 2. Qué versión usar en esta ruta

| Situación | Recomendación |
|---|---|
| Aprender hoy | Java 21 LTS: moderno, estable y ampliamente adoptado |
| Proyecto nuevo con stack ya compatible | Evaluar Java 25 LTS |
| Empresa fijada en Java 17 | Usar 17 y conocer qué funciones del curso requieren 21 |
| Mantener Java 8/11 | Aprender con el runtime real, pero practicar migración y evitar copiar sus limitaciones al diseño nuevo |
| Probar una versión no LTS | Solo con estrategia explícita de actualización semestral |

No elijas únicamente por “la versión más nueva”. Comprueba framework, servidor, librerías, herramientas de build, observabilidad, despliegue y soporte operativo.

## 3. Línea temporal práctica

| Versión | Tipo | Cambios que un desarrollador debe reconocer |
|---|---|---|
| 8 (2014) | LTS | lambdas, Streams, `Optional`, `java.time`, métodos `default` |
| 9 | no LTS | módulos JPMS, `List.of`, JShell, Stream API ampliada |
| 10 | no LTS | inferencia local con `var` |
| 11 (2018) | LTS | `HttpClient`, ejecución de archivo fuente, nuevos métodos de `String`, retiro de módulos Java EE del JDK |
| 12–14 | no LTS | evolución de `switch`; expresiones `switch` finales en 14 |
| 15 | no LTS | text blocks finales |
| 16 | no LTS | records finales, pattern matching para `instanceof` final, `Stream.toList()` |
| 17 (2021) | LTS | sealed classes finales; base LTS consolidada |
| 18–20 | no LTS | UTF-8 por defecto, preview de pattern matching y virtual threads |
| 21 (2023) | LTS | virtual threads, record patterns y pattern matching para `switch` finales; sequenced collections |
| 22–24 | no LTS | iteraciones de APIs preview/incubator y mejoras de runtime |
| 25 (2025) | LTS | scoped values, imports de módulos, archivos fuente compactos, constructores flexibles; mejoras AOT, JFR y GC |
| 26 (2026) | no LTS | versión de actualización semestral; no es la base del curso |

“Final” importa: una característica preview puede cambiar, requiere `--enable-preview` y no debe convertirse en dependencia de producción sin una decisión consciente.

## 4. Java 8: el cambio funcional

Java 8 introdujo buena parte del estilo moderno:

```java
List<String> activos = usuarios.stream()
        .filter(Usuario::activo)
        .map(Usuario::nombre)
        .collect(Collectors.toList());
```

También llegó `java.time`, que debe preferirse sobre `Date`/`Calendar` en código nuevo:

```java
LocalDate fecha = LocalDate.of(2026, 7, 17);
Instant ahora = Instant.now();
```

## 5. Java 9 a 11: módulos y API moderna

### Módulos

JPMS agrega un nivel por encima de paquetes:

```java
module com.academia.app {
    requires java.sql;
    exports com.academia.api;
}
```

No necesitas modularizar cada proyecto al aprender, pero debes distinguir classpath de module path y reconocer errores de acceso fuerte.

### Colecciones inmutables y `var`

```java
var cursos = List.of("Java", "SQL");
```

`List.of` no acepta `null` y no permite modificaciones. `var` infiere el tipo estático; no funciona en campos, parámetros o retornos.

### HTTP Client

```java
HttpClient cliente = HttpClient.newHttpClient();
HttpRequest solicitud = HttpRequest.newBuilder(URI.create(url)).build();
HttpResponse<String> respuesta = cliente.send(
        solicitud, HttpResponse.BodyHandlers.ofString());
```

## 6. Java 12 a 17: lenguaje expresivo

### Switch expression

```java
int dias = switch (mes) {
    case ABRIL, JUNIO, SEPTIEMBRE, NOVIEMBRE -> 30;
    case FEBRERO -> bisiesto ? 29 : 28;
    default -> 31;
};
```

### Text blocks

```java
String json = """
        {
          "curso": "Java"
        }
        """;
```

### Records, pattern matching y sealed classes

```java
record Usuario(String nombre) {}

if (objeto instanceof Usuario u) {
    System.out.println(u.nombre());
}
```

Estudia los detalles en [Records y modelado moderno](README_Records_JavaModerno.md).

## 7. Java 17 frente a Java 21

Java 17 ya incluye records, sealed classes, text blocks y pattern matching para `instanceof`. Java 21 añade como características finales:

- virtual threads para escalar tareas que esperan I/O;
- pattern matching para `switch`;
- record patterns para desestructurar datos;
- sequenced collections (`SequencedCollection`, `SequencedSet`, `SequencedMap`).

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    Future<String> futuro = executor.submit(this::consultarServicio);
    System.out.println(futuro.get());
}
```

Los hilos virtuales mejoran **throughput** en cargas con muchas esperas; no hacen más rápida una tarea intensiva en CPU y no deben agruparse en pools. Consulta [JEP 444](https://openjdk.org/jeps/444).

Un switch exhaustivo en Java 21:

```java
static double area(Figura figura) {
    return switch (figura) {
        case Circulo(var radio) -> Math.PI * radio * radio;
        case Rectangulo(var ancho, var alto) -> ancho * alto;
    };
}
```

## 8. Java 21 frente a Java 25

Java 25 conserva todo Java 21 y finaliza varias mejoras del lenguaje y concurrencia:

- **Scoped Values:** compartir datos inmutables dentro de una ejecución delimitada, especialmente útil con hilos virtuales.
- **Module Import Declarations:** importar de forma concisa los paquetes exportados por un módulo.
- **Compact Source Files and Instance Main Methods:** reduce ceremonia para programas iniciales y scripts; no elimina la sintaxis clásica.
- **Flexible Constructor Bodies:** permite validar o preparar argumentos antes de invocar explícitamente otro constructor, respetando reglas de inicialización.
- mejoras de runtime en AOT, JFR, headers de objetos y collectors.

Java 25 también contiene características que siguen siendo preview o incubator, como Structured Concurrency en su quinta preview y Vector API en incubación. No las presentes como API final. Revisa el estado exacto en [la lista oficial de JDK 25](https://openjdk.org/projects/jdk/25/).

Para un principiante, la sintaxis compacta de Java 25 puede facilitar el primer contacto, pero debe aprender pronto clases, visibilidad, `static` y la forma canónica: son necesarias para leer proyectos reales.

## 9. Compatibilidad: cuatro preguntas diferentes

Al migrar, “¿es compatible?” es ambiguo:

1. **Fuente:** ¿el código vuelve a compilar con el JDK nuevo?
2. **Binaria:** ¿clases ya compiladas enlazan con sus dependencias?
3. **Runtime:** ¿el bytecode puede ejecutarse en esa JVM?
4. **Comportamiento:** ¿produce los mismos resultados y rendimiento?

Un JDK nuevo suele ejecutar bytecode antiguo, pero un JDK viejo no entiende bytecode nuevo:

```text
UnsupportedClassVersionError
```

Compilar con JDK 21 para ejecutar en Java 17:

```bash
javac --release 17 Main.java
```

En Maven:

```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
</properties>
```

`--release` limita bytecode y API pública al objetivo; es preferible a configurar solo `source` y `target`.

## 10. Distribuciones y licencias

OpenJDK es el proyecto de referencia abierto; múltiples proveedores publican builds compatibles. “Java 21” no identifica por sí solo al proveedor ni los términos de soporte.

Al elegir distribución revisa:

- licencia y derecho de uso en tu organización;
- calendario de parches de seguridad;
- duración y costo de soporte;
- arquitecturas y sistemas operativos;
- componentes adicionales y certificación TCK.

No confundas LTS del estándar/ecosistema con un contrato gratuito e idéntico de todos los proveedores.

## 11. Estrategia de migración

1. Inventaría versión, distribución, flags JVM y dependencias.
2. Actualiza build, CI, IDE, analizadores y plugins.
3. Ejecuta pruebas en el JDK destino antes de cambiar código.
4. Corrige uso de APIs internas (`sun.*`) y reflexión bloqueada por módulos.
5. Actualiza librerías y frameworks según sus matrices oficiales.
6. Mide memoria, latencia, throughput y pausas de GC.
7. Despliega progresivamente con observabilidad y reversión.
8. Adopta nuevas características después de estabilizar la plataforma.

Herramientas del JDK como `jdeps` y `jdeprscan` ayudan a detectar dependencias y APIs obsoletas.

## 12. Errores conceptuales frecuentes

- “LTS es más rápida”: LTS describe soporte, no rendimiento.
- “Java 21 requiere reescribir Java 17”: normalmente no; primero recompila y prueba.
- “Records nacieron en Java 17”: fueron finales en Java 16; Java 17 los incluye.
- “Virtual threads hacen más CPU”: escalan espera bloqueante, no multiplican núcleos.
- “Todo lo anunciado en una versión es final”: revisa etiquetas Preview/Incubator/Experimental.
- “JRE y JDK son sinónimos”: cumplen roles distintos aunque la distribución moderna haya cambiado.

## Checklist de elección

- [ ] La versión está soportada por framework, servidor y librerías.
- [ ] La distribución y licencia son adecuadas.
- [ ] CI y producción usan el mismo objetivo `--release`.
- [ ] No dependemos accidentalmente de preview features.
- [ ] Existen pruebas y mediciones antes de migrar.
- [ ] El equipo conoce las diferencias de lenguaje que realmente usará.

