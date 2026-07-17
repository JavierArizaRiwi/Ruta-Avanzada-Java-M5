# Streams, lambdas y programación funcional en Java

## 1. Qué problema resuelve un Stream

Un `Stream<T>` describe una **secuencia de operaciones** sobre datos. No es una estructura que almacena elementos y no es un `InputStream` de archivos. Permite expresar “qué transformación quiero” con un pipeline.

```java
List<String> aprobados = estudiantes.stream()
        .filter(Estudiante::aprobo)
        .map(Estudiante::nombre)
        .sorted()
        .toList();
```

Se lee: toma estudiantes, conserva aprobados, transforma cada estudiante en su nombre, ordena y materializa una lista.

Un bucle sigue siendo correcto. Usa Stream cuando mejora la intención de una transformación; usa bucle cuando hay control de flujo complejo, mutación deliberada o el pipeline sería críptico.

## 2. De colección a pipeline

Un pipeline tiene:

1. una fuente: colección, array, rango, archivo o generador;
2. cero o más operaciones intermedias;
3. una operación terminal.

```java
long cantidad = pedidos.stream()     // fuente
        .filter(Pedido::pagado)      // intermedia
        .map(Pedido::cliente)        // intermedia
        .distinct()                  // intermedia stateful
        .count();                    // terminal
```

Las operaciones intermedias son perezosas: no procesan hasta que existe una terminal. Un Stream se consume una sola vez.

```java
Stream<String> stream = nombres.stream();
long total = stream.count();
// stream.toList(); // IllegalStateException: ya fue operado/cerrado
```

## 3. Interfaces funcionales y lambdas

Una interfaz funcional tiene un único método abstracto. Las más usadas son:

| Interfaz | Firma mental | Ejemplo |
|---|---|---|
| `Predicate<T>` | `T -> boolean` | filtrar aprobados |
| `Function<T,R>` | `T -> R` | estudiante a nombre |
| `Consumer<T>` | `T -> void` | imprimir o enviar |
| `Supplier<T>` | `() -> T` | construir un valor |
| `UnaryOperator<T>` | `T -> T` | normalizar texto |
| `BinaryOperator<T>` | `(T,T) -> T` | combinar valores |

Formas equivalentes:

```java
Predicate<Estudiante> p1 = e -> e.nota() >= 3.0;
Predicate<Estudiante> p2 = Estudiante::aprobo;
```

Una lambda puede capturar variables locales solo si son finales o efectivamente finales. Evita efectos secundarios: hacen que el resultado dependa del orden y dificultan paralelizar o probar.

```java
// Evitar: mutación externa dentro del pipeline
List<String> salida = new ArrayList<>();
nombres.stream().filter(n -> !n.isBlank()).forEach(salida::add);

// Preferir
List<String> salidaSegura = nombres.stream()
        .filter(n -> !n.isBlank())
        .toList();
```

`Stream.toList()` devuelve una lista no modificable. Si necesitas una lista mutable, usa `collect(Collectors.toCollection(ArrayList::new))`.

## 4. Operaciones intermedias esenciales

### `filter`

Conserva elementos que cumplen un predicado:

```java
pedidos.stream().filter(p -> p.total().signum() > 0);
```

### `map`

Transforma un elemento en exactamente otro:

```java
estudiantes.stream().map(Estudiante::email);
```

### `flatMap`

Transforma cada elemento en cero o más y aplana el resultado:

```java
List<LineaPedido> lineas = pedidos.stream()
        .flatMap(p -> p.lineas().stream())
        .toList();
```

Si `map` produce `Stream<Stream<T>>` o `List<List<T>>` cuando buscas elementos, probablemente necesitas `flatMap`.

### `distinct`, `sorted`, `limit`, `skip`, `peek`

```java
List<String> pagina = nombres.stream()
        .map(String::trim)
        .filter(s -> !s.isEmpty())
        .distinct()
        .sorted(String.CASE_INSENSITIVE_ORDER)
        .skip(20)
        .limit(10)
        .toList();
```

`distinct` depende de `equals`/`hashCode`. `sorted` y `distinct` necesitan recordar estado. Usa `peek` principalmente para diagnóstico; no conviertas un pipeline en una secuencia oculta de mutaciones.

## 5. Operaciones terminales

```java
boolean hayReprobados = estudiantes.stream().anyMatch(e -> !e.aprobo());
boolean todosActivos = estudiantes.stream().allMatch(Estudiante::activo);
Optional<Estudiante> primero = estudiantes.stream().findFirst();
long total = estudiantes.stream().count();
```

Las operaciones de cortocircuito como `anyMatch`, `findFirst` y `limit` pueden detener el procesamiento temprano.

### `reduce`

Combina elementos en un valor:

```java
int suma = numeros.stream().reduce(0, Integer::sum);
Optional<Integer> maximo = numeros.stream().reduce(Integer::max);
```

La identidad debe ser neutra y la operación asociativa, especialmente si el Stream puede ser paralelo. Para sumar números usa streams primitivos cuando sea claro:

```java
double promedio = estudiantes.stream()
        .mapToDouble(Estudiante::nota)
        .average()
        .orElse(0.0);
```

`IntStream`, `LongStream` y `DoubleStream` evitan boxing y ofrecen `sum`, `average` y `summaryStatistics`.

## 6. Collectors

Un collector acumula elementos en un resultado.

```java
Set<String> ciudades = personas.stream()
        .map(Persona::ciudad)
        .collect(Collectors.toSet());
```

### Agrupar

```java
Map<String, List<Estudiante>> porCurso = estudiantes.stream()
        .collect(Collectors.groupingBy(Estudiante::curso));
```

Collector descendente:

```java
Map<String, Double> promedioPorCurso = estudiantes.stream()
        .collect(Collectors.groupingBy(
                Estudiante::curso,
                Collectors.averagingDouble(Estudiante::nota)
        ));
```

### Particionar

`partitioningBy` produce dos grupos booleanos:

```java
Map<Boolean, List<Estudiante>> porResultado = estudiantes.stream()
        .collect(Collectors.partitioningBy(Estudiante::aprobo));
```

### Crear mapas y resolver claves repetidas

```java
Map<String, Estudiante> mejorPorEmail = estudiantes.stream()
        .collect(Collectors.toMap(
                Estudiante::email,
                Function.identity(),
                BinaryOperator.maxBy(Comparator.comparingDouble(Estudiante::nota))
        ));
```

Sin función de combinación, una clave duplicada lanza `IllegalStateException`. Decide explícitamente si conservar primero, último, mejor o agrupar.

### Componer resultados

```java
String listado = nombres.stream()
        .sorted()
        .collect(Collectors.joining(", ", "[", "]"));
```

## 7. Optional bien usado

`Optional<T>` expresa que un **resultado** puede estar ausente:

```java
Optional<Estudiante> encontrado = repositorio.buscarPorId(id);
String nombre = encontrado
        .map(Estudiante::nombre)
        .orElse("Desconocido");
```

Evita:

- usar `Optional` como campo de entidad o parámetro por defecto;
- llamar `get()` sin comprobar;
- devolver `null` dentro de un `Optional`;
- `isPresent()` seguido de `get()` cuando `map`, `orElseGet` o `orElseThrow` expresan mejor el flujo.

`orElse` evalúa su argumento siempre; `orElseGet` construye el alternativo solo si falta el valor.

```java
Usuario usuario = encontrado.orElseGet(this::crearInvitado);
```

## 8. Orden, estado y efectos secundarios

Un pipeline funcional ideal transforma valores sin modificar fuente ni estado externo. Esto mejora razonamiento y seguridad concurrente.

```java
List<EstudianteDto> dtos = estudiantes.stream()
        .filter(Estudiante::activo)
        .map(mapper::aDto)
        .toList();
```

`forEach` no garantiza el orden en Streams paralelos; `forEachOrdered` lo conserva con un costo. Si el orden forma parte del resultado, represéntalo mediante `sorted`, una colección ordenada o una operación secuencial clara.

## 9. Streams paralelos: no por defecto

`parallelStream()` reparte trabajo en el common `ForkJoinPool`. No garantiza que sea más rápido.

Puede ayudar cuando:

- hay muchos elementos;
- cada operación consume CPU suficiente;
- las operaciones son independientes y asociativas;
- se mide una mejora en un entorno parecido a producción.

Suele perjudicar con colecciones pequeñas, I/O bloqueante, estado compartido, orden estricto o servidores que ya administran concurrencia. Para I/O concurrente en Java 21 considera hilos virtuales, no Streams paralelos.

## 10. Errores comunes

1. Encadenar veinte operaciones sin nombres intermedios.
2. Usar `map` solo para ejecutar un efecto secundario.
3. Capturar y mutar una colección externa.
4. Convertir a lista entre cada operación.
5. Usar `parallelStream()` sin benchmark.
6. Hacer consultas de base de datos dentro de cada `map` y provocar N+1 llamadas.
7. Suponer que `toList()` devuelve una lista mutable.
8. Ignorar duplicados al usar `toMap`.

Divide pipelines complejos en funciones nombradas:

```java
Predicate<Estudiante> elegible = e -> e.activo() && e.nota() >= 3.0;
Comparator<Estudiante> porNotaDesc = Comparator.comparingDouble(Estudiante::nota).reversed();

List<Estudiante> seleccionados = estudiantes.stream()
        .filter(elegible)
        .sorted(porNotaDesc)
        .limit(10)
        .toList();
```

## 11. Ejemplo completo

```java
record Estudiante(String nombre, String curso, double nota, boolean activo) {
    boolean aprobo() {
        return nota >= 3.0;
    }
}

record Resumen(long cantidad, double promedio, double maxima) {}

Map<String, Resumen> resumenPorCurso = estudiantes.stream()
        .filter(Estudiante::activo)
        .collect(Collectors.groupingBy(
                Estudiante::curso,
                Collectors.collectingAndThen(
                        Collectors.summarizingDouble(Estudiante::nota),
                        s -> new Resumen(s.getCount(), s.getAverage(), s.getMax())
                )
        ));
```

## 12. Ejercicios

1. Normaliza una lista de correos, elimina vacíos y duplicados, y ordénala.
2. Agrupa ventas por mes y suma `BigDecimal` sin convertir a `double`.
3. Obtén los tres estudiantes con mejor nota de cada curso.
4. Convierte `List<Pedido>` a un mapa de unidades totales por producto usando `flatMap`.
5. Implementa cada solución primero con bucle y luego con Stream; escribe cuál comunica mejor la intención.

## Checklist

- [ ] Distingo fuente, operación intermedia y terminal.
- [ ] Explico pereza y consumo único.
- [ ] Elijo correctamente entre `map` y `flatMap`.
- [ ] Manejo claves duplicadas en `toMap`.
- [ ] Evito efectos secundarios en pipelines.
- [ ] No paralelizo sin medir.
- [ ] Uso `Optional` como retorno ausente, no como decoración universal.
