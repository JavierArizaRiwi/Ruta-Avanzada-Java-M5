# Sesión de Entrenamiento – Arrays, Matrices y Colecciones en Java

**Nivel:** Principiante – Intermedio  
**Objetivo:** Practicar arrays (1D), matrices (2D) y colecciones en Java mediante ejercicios progresivos y ejemplos de aplicación real.

---

## Agenda

1. Arrays 1D – fundamentos  
2. Operaciones típicas sobre arrays  
3. Matrices 2D – recorridos y diagonales  
4. Transposición y rotaciones  
5. Multiplicación de matrices  
6. Colecciones en Java: ArrayList, LinkedList, Queue  
7. Desafío final

---

## 1) Arrays 1D – Fundamentos

### Ejercicio 1 – Crear y recorrer
Crea un arreglo de tamaño `n`, llénalo con valores de `1..n` y muestra cada elemento.  
**Firma:**  
```java
static int[] buildSequential(int n)
```
**Aplicación real:**  
Un array puede usarse para almacenar los puntajes de los jugadores en un juego, o los días del mes en un calendario.

### Ejercicio 2 – Suma y promedio
Calcula la suma y el promedio de un arreglo.  
**Firmas:**  
```java
static long sum(int[] a)
static double average(int[] a)
```
**Aplicación real:**  
Calcular el promedio de calificaciones de un estudiante o el total de ventas diarias.

---

## 2) Operaciones típicas sobre arrays

### Ejercicio 3 – Máximo, mínimo y posición
Encuentra el valor mínimo, máximo y sus posiciones.  
**Firma:**  
```java
static int[] minMaxAndPositions(int[] a) // {min, idxMin, max, idxMax}
```
**Aplicación real:**  
Encontrar el día con mayor o menor temperatura en un mes.

### Ejercicio 4 – Invertir arreglo
Invierte un arreglo **in-place**.  
**Firma:**  
```java
static void reverse(int[] a)
```
**Aplicación real:**  
Revertir el orden de una lista de tareas o historial de acciones.

---

## 3) Matrices 2D – Recorridos y diagonales

### Ejercicio 5 – Suma por filas y columnas
Devuelve un arreglo con la suma de cada fila y cada columna.  
**Firmas:**  
```java
static long[] rowSums(int[][] m)
static long[] colSums(int[][] m)
```
**Aplicación real:**  
Sumar ventas por producto (filas) y por día (columnas) en una tienda.

### Ejercicio 6 – Sumas de diagonales
En una matriz cuadrada, suma la diagonal principal y secundaria.  
**Firma:**  
```java
static long[] diagonalSums(int[][] m) // {principal, secundaria}
```
**Aplicación real:**  
En juegos como el Sudoku o el Tres en Raya, verificar sumas en diagonales.

---

## 4) Transposición y rotaciones

### Ejercicio 7 – Transponer matriz
Devuelve la transpuesta de una matriz.  
**Firma:**  
```java
static int[][] transpose(int[][] m)
```
**Aplicación real:**  
Girar una imagen (matriz de píxeles) o intercambiar filas y columnas en una hoja de cálculo.

### Ejercicio 8 – Rotar 90° clockwise
Rota una matriz cuadrada 90° a la derecha.  
**Firma:**  
```java
static int[][] rotate90Clockwise(int[][] m)
```
**Aplicación real:**  
Rotar piezas en un juego tipo Tetris o manipular imágenes.

---

## 5) Multiplicación de matrices

### Ejercicio 9 – Producto de matrices
Multiplica `A (r x c)` por `B (c x k)` y devuelve `C (r x k)`.  
**Firma:**  
```java
static int[][] multiply(int[][] A, int[][] B)
```
**Aplicación real:**  
Cálculos en álgebra lineal, gráficos por computadora, o sistemas de recomendación.

---

## 6) Colecciones en Java: ArrayList, LinkedList, Queue

### ArrayList
Una **ArrayList** es una lista dinámica que permite agregar, eliminar y acceder a elementos por índice.  
Se basa internamente en un array, pero su tamaño puede crecer automáticamente.

**Ejemplo:**
```java
import java.util.ArrayList;

ArrayList<String> nombres = new ArrayList<>();
nombres.add("Ana");
nombres.add("Luis");
nombres.add("Sofía");
System.out.println(nombres.get(1)); // Imprime "Luis"
```
**Aplicación real:**  
Gestión de una lista de usuarios conectados en una aplicación.

### LinkedList
Una **LinkedList** es una lista enlazada donde cada elemento apunta al siguiente (y al anterior, si es doblemente enlazada).  
Permite inserciones y eliminaciones eficientes en cualquier posición, pero el acceso por índice es más lento que en ArrayList.

**Ejemplo:**
```java
import java.util.LinkedList;

LinkedList<Integer> numeros = new LinkedList<>();
numeros.add(10);
numeros.addFirst(5);
numeros.addLast(20);
System.out.println(numeros); // [5, 10, 20]
```
**Aplicación real:**  
Implementar una lista de reproducción de canciones donde se agregan o eliminan canciones al inicio o final.

### Queue (Cola)
Una **Queue** es una estructura FIFO (First-In, First-Out), donde el primer elemento en entrar es el primero en salir.  
En Java, se puede implementar con `LinkedList` o con clases especializadas como `ArrayDeque`.

**Ejemplo:**
```java
import java.util.Queue;
import java.util.LinkedList;

Queue<String> cola = new LinkedList<>();
cola.add("Cliente1");
cola.add("Cliente2");
System.out.println(cola.poll()); // "Cliente1" sale de la cola
```
**Aplicación real:**  
Gestión de turnos en una fila de atención al cliente o procesamiento de tareas en orden de llegada.

---

## 7) Desafío final – Recorrido en espiral

### Ejercicio 10 – Recorrido en espiral
Devuelve un arreglo con los elementos de una matriz en orden espiral.  
**Firma:**  
```java
static int[] spiralOrder(int[][] m)
```
**Aplicación real:**  
Recorrer una imagen o matriz de sensores en un patrón específico, o resolver problemas de lógica en entrevistas técnicas.

---

## Plantilla de Proyecto

```java
public class Main {
    public static void main(String[] args) {
        int[] a = ArraysYMatrices.buildSequential(5); // [1,2,3,4,5]
        ArraysYMatrices.reverse(a);                   // [5,4,3,2,1]

        int[][] m = {
            {1, 2, 3},
            {4, 5, 6}
        };
        int[][] mt = ArraysYMatrices.transpose(m);    // 3x2

        // Ejemplo con ArrayList
        ArrayList<String> lista = new ArrayList<>();
        lista.add("Ejemplo");
        System.out.println(lista);

        // Ejemplo con Queue
        Queue<Integer> cola = new LinkedList<>();
        cola.add(100);
        cola.add(200);
        System.out.println(cola.poll()); // 100
    }
}
```

---

## Tips de evaluación

- Probar con arreglos vacíos y matrices rectangulares.  
- Extender con rotaciones 180° y 270°.  
- Implementar búsqueda en matriz ordenada.  
- Comparar el uso de ArrayList y LinkedList según el caso de uso.

---

## Reto Extra

Implementa una función que encuentre el valor más frecuente en una matriz 2D y su cantidad de repeticiones.

**Aplicación real:**  
Detectar el color dominante en una imagen digital o el valor más común en una encuesta.

---