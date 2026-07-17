# Fundamentos de Java desde cero

## 1. Java, JDK, JVM y bytecode

Java es a la vez un lenguaje y una plataforma. Conviene separar estas piezas:

| Concepto | Función |
|---|---|
| Lenguaje Java | Reglas con las que escribes clases, métodos y expresiones |
| Java SE | Especificación y API estándar: colecciones, archivos, concurrencia, red, etc. |
| JDK | Herramientas de desarrollo: compilador `javac`, lanzador `java`, Javadoc y otras |
| JVM | Máquina virtual que carga, verifica y ejecuta bytecode |
| JRE | Entorno de ejecución; en Java moderno normalmente se distribuye o genera junto con la aplicación |

El flujo básico es:

```text
Main.java --javac--> Main.class (bytecode) --JVM--> código ejecutado
```

La JVM interpreta y compila dinámicamente zonas frecuentes mediante JIT. “Escribe una vez, ejecuta en cualquier lugar” depende de tener una JVM compatible y de no depender accidentalmente del sistema operativo.

## 2. Primer programa y estructura

```java
package com.academia.inicio;

public class Main {
    public static void main(String[] args) {
        System.out.println("Hola, Java");
    }
}
```

- `package` evita colisiones y organiza tipos.
- `class` declara un tipo.
- `main` es el punto de entrada convencional.
- `public` permite que el lanzador acceda al método.
- `static` permite invocarlo sin construir `Main`.
- `void` indica que no devuelve valor.
- `String[] args` recibe argumentos de terminal.

Con paquetes, la ruta debe coincidir: `src/main/java/com/academia/inicio/Main.java`.

## 3. Variables, valores y tipos

### Tipos primitivos

| Tipo | Ejemplo | Uso habitual |
|---|---|---|
| `boolean` | `true` | condiciones |
| `byte`, `short`, `int`, `long` | `42`, `42L` | enteros; usa `int` por defecto |
| `float`, `double` | `3.14`, `3.14F` | aproximaciones; usa `double` por defecto |
| `char` | `'A'` | una unidad UTF-16, no necesariamente un carácter Unicode completo |

`String`, arrays y cualquier clase son tipos de referencia. Una variable de referencia contiene una referencia a un objeto o `null`, no el objeto “dentro” de la variable.

```java
int edad = 20;
double promedio = 4.5;
boolean activo = true;
String nombre = "Ana";
var mensaje = nombre + " tiene " + edad; // inferido localmente como String
```

`var` no hace Java dinámico: el tipo se infiere en compilación, solo para variables locales con inicializador. Úsalo cuando el tipo siga siendo evidente.

### Conversión

```java
int cantidad = 10;
long ampliado = cantidad;       // conversión segura implícita
int reducido = (int) 10_000L;   // cast explícito: puede perder información
int numero = Integer.parseInt("42");
String texto = String.valueOf(numero);
```

Para dinero no uses `double`: su representación binaria introduce errores. Usa `BigDecimal` construido desde texto.

```java
BigDecimal precio = new BigDecimal("19.90");
```

## 4. Operadores y errores frecuentes

```java
int suma = 4 + 2;
int resto = 7 % 3;
boolean permitido = edad >= 18 && activo;
boolean alternativo = esAdmin || tienePermiso;
```

- `=` asigna; `==` compara valores primitivos o identidad de referencias.
- Para contenido de objetos, normalmente usa `equals`.
- `&&` y `||` cortocircuitan: no evalúan la derecha cuando el resultado ya se conoce.
- `5 / 2` vale `2`; si necesitas decimales usa `5.0 / 2`.
- Un `int` desborda silenciosamente; considera `long` o `Math.addExact` si importa detectarlo.

## 5. Control de flujo

```java
if (nota >= 3.0) {
    System.out.println("Aprobó");
} else {
    System.out.println("No aprobó");
}
```

Un `switch` moderno puede producir un valor:

```java
String nivel = switch (codigo) {
    case 1 -> "básico";
    case 2 -> "intermedio";
    case 3 -> "avanzado";
    default -> "desconocido";
};
```

Bucles:

```java
for (int i = 0; i < notas.length; i++) {
    System.out.println(i + ": " + notas[i]);
}

for (double nota : notas) {
    System.out.println(nota);
}

while (hayTrabajo()) {
    procesarSiguiente();
}
```

Prefiere el `for-each` si no necesitas índice. Usa `break` y `continue` con moderación; muchos saltos suelen señalar que el método hace demasiado.

## 6. Métodos y paso por valor

```java
static double promedio(double a, double b) {
    return (a + b) / 2;
}
```

Java siempre pasa argumentos **por valor**. Para un objeto, el valor copiado es su referencia. El método puede mutar el objeto referenciado, pero reasignar el parámetro no cambia la variable del llamador.

```java
static void renombrar(Estudiante e) {
    e.cambiarNombre("Ana"); // puede modificar el mismo objeto
    e = new Estudiante("Luis"); // solo reasigna el parámetro local
}
```

La firma para sobrecarga considera nombre y parámetros, no el retorno. Evita métodos con muchos parámetros del mismo tipo; un objeto de parámetros o record expresa mejor la intención.

## 7. String, igualdad e inmutabilidad

```java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);      // false: referencias distintas
System.out.println(a.equals(b)); // true: mismo contenido
```

`String` es inmutable. Métodos como `toUpperCase()` crean otro valor; no cambian el original.

```java
nombre = nombre.trim().toUpperCase();
```

Para concatenar repetidamente en un bucle usa `StringBuilder`:

```java
var resultado = new StringBuilder();
for (String palabra : palabras) {
    resultado.append(palabra).append(' ');
}
String texto = resultado.toString().trim();
```

Para comparar aceptando `null`, `Objects.equals(a, b)` es seguro.

## 8. Arrays y colecciones

Un array tiene tamaño fijo y puede almacenar primitivos:

```java
int[] notas = {5, 4, 3};
notas[0] = 10;
```

Las colecciones crecen y modelan distintas necesidades:

| Interfaz | Pregunta que responde | Implementación inicial |
|---|---|---|
| `List<E>` | ¿Importan orden y duplicados? | `ArrayList` |
| `Set<E>` | ¿Necesito unicidad? | `HashSet` |
| `Map<K,V>` | ¿Busco un valor por clave? | `HashMap` |
| `Queue<E>` | ¿Proceso por orden de llegada/prioridad? | `ArrayDeque` |

Programa contra la interfaz:

```java
List<String> nombres = new ArrayList<>();
nombres.add("Ana");
nombres.add("Luis");

Map<String, Integer> conteo = new HashMap<>();
conteo.merge("java", 1, Integer::sum);
```

Para que `HashSet` y las claves de `HashMap` funcionen bien, `equals` y `hashCode` deben ser coherentes. No cambies campos que participan en el hash mientras el objeto esté dentro de una colección hash.

## 9. Genéricos esenciales

Los genéricos aportan seguridad de tipos en compilación:

```java
List<String> lenguajes = new ArrayList<>();
// lenguajes.add(42); // no compila
String primero = lenguajes.get(0);
```

Un tipo genérico propio:

```java
record Resultado<T>(T valor, boolean exitoso) {}
```

Los genéricos son invariantes: `List<Integer>` no es subtipo de `List<Number>`. Con comodines, recuerda PECS:

- Producer Extends: `List<? extends Number>` permite leer números.
- Consumer Super: `List<? super Integer>` permite añadir enteros.

```java
static double sumar(List<? extends Number> valores) {
    return valores.stream().mapToDouble(Number::doubleValue).sum();
}
```

## 10. Memoria y ciclo de vida

Como modelo inicial:

- cada invocación mantiene variables locales y marcos de llamada en una pila;
- los objetos suelen vivir en el heap;
- el garbage collector recupera objetos que ya no son alcanzables;
- una referencia `null` no es un objeto y acceder a ella causa `NullPointerException`.

Es una simplificación útil, no una promesa sobre la implementación física. La JVM puede optimizar asignaciones si conserva el comportamiento observable.

## 11. Cómo leer errores

Hay tres familias:

1. **Compilación:** sintaxis o tipos; el programa no se construye.
2. **Ejecución:** una excepción interrumpe el flujo.
3. **Lógica:** ejecuta, pero produce un resultado incorrecto.

En un stack trace, localiza el tipo y mensaje de excepción, luego la primera línea de tu propio paquete. Lee la cadena `Caused by` desde la causa más profunda hacia afuera.

## 12. Ejercicio integrador

Crea un programa de consola que:

1. reciba nombres y notas;
2. valide nombre no vacío y nota entre 0 y 5;
3. almacene estudiantes en una `List`;
4. calcule promedio, máximo y mínimo con bucles;
5. cuente aprobados por medio de un método;
6. maneje entrada inválida sin ocultar errores de programación.

Después de dominar [POO](../SESIONES/Semana2/README_Sesion_S2Dia2.md), reemplaza los datos sueltos por objetos. Después de dominar [Streams](README_Streams_Java.md), reescribe solo los cálculos donde el pipeline resulte más claro.

## Checklist

- [ ] Distingo primitivo, referencia y `null`.
- [ ] Comparo `String` con `equals`, no con `==`.
- [ ] Explico paso por valor incluso para objetos.
- [ ] Elijo `List`, `Set` o `Map` por su semántica.
- [ ] Divido un problema en métodos pequeños con nombres de intención.
- [ ] Puedo compilar, ejecutar y leer el primer error relevante.

