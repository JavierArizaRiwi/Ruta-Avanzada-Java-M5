# Hilos y Concurrencia en Java: explicación profunda y práctica

## Objetivo
Este documento explica desde cero qué son los hilos, cómo funciona la concurrencia en Java, por qué cuesta tanto entender este tema y qué debes dominar para usarlo sin errores. Está pensado para que puedas pasar de la teoría a decisiones prácticas en código real.

## 1. El problema que resuelve la concurrencia
La concurrencia aparece cuando quieres que una aplicación haga más de una cosa al mismo tiempo o, al menos, parezca hacerlo.

Ejemplos reales:
- Una interfaz gráfica que no debe congelarse mientras carga datos.
- Un servidor que atiende muchas peticiones a la vez.
- Un proceso que descarga archivos mientras valida otros datos.
- Una app que consulta base de datos y calcula resultados en paralelo.

Si todo se ejecutara en un único flujo secuencial, la aplicación sería simple pero lenta e incómoda.

## 2. Qué es un hilo
Un hilo es una unidad de ejecución dentro de un proceso.

### Proceso vs hilo
- Proceso: programa en ejecución con su propia memoria y recursos.
- Hilo: camino de ejecución dentro de ese proceso.

Una aplicación Java normalmente arranca con un hilo principal. Desde ahí puedes crear más hilos para repartir trabajo.

## 3. Por qué los hilos son difíciles de entender
La dificultad no está solo en la sintaxis. Está en que varios hilos comparten memoria y recursos.

Eso provoca problemas como:
- Condiciones de carrera: dos hilos modifican el mismo dato al mismo tiempo.
- Inconsistencia de estado: un hilo ve datos a medio actualizar.
- Bloqueos: dos hilos se esperan mutuamente y nadie avanza.
- Hambruna: un hilo queda sin oportunidad de ejecutarse.

La clave es entender que concurrencia no significa automáticamente paralelismo ni seguridad.

## 4. Concurrencia, paralelismo y asincronía
### Concurrencia
Varias tareas progresan en el tiempo, aunque no necesariamente en el mismo instante.

### Paralelismo
Varias tareas se ejecutan literalmente al mismo tiempo en distintos núcleos.

### Asincronía
Una tarea se lanza y el hilo que la inició no espera bloqueado inmediatamente.

Estas tres ideas se mezclan mucho. En Java puedes tener concurrencia sin paralelismo, paralelismo sin buena sincronización, y asincronía sin múltiples hilos explícitos.

## 5. Cómo se crea un hilo en Java
### Opción 1: extender Thread
```java
public class MiHilo extends Thread {
    @Override
    public void run() {
        System.out.println("Hola desde el hilo");
    }
}
```

### Opción 2: implementar Runnable
```java
public class Tarea implements Runnable {
    @Override
    public void run() {
        System.out.println("Trabajo ejecutado por un hilo");
    }
}
```

### Opción 3: usar ExecutorService
Esta es la opción más recomendable en aplicaciones reales.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

ExecutorService pool = Executors.newFixedThreadPool(4);
pool.submit(() -> System.out.println("Tarea en segundo plano"));
pool.shutdown();
```

## 6. Qué pasa cuando llamas a start() y run()
Este punto confunde mucho a principiantes.

- `start()` crea un nuevo hilo y luego Java ejecuta `run()` en ese hilo.
- `run()` llamado directamente no crea un hilo nuevo; se ejecuta como un método normal.

Ejemplo mental:
- `start()` = abrir una nueva línea de ejecución.
- `run()` = ejecutar el trabajo en la línea actual.

## 7. Ciclo de vida de un hilo
Un hilo pasa por estados como:
- Nuevo
- Ejecutable
- En ejecución
- Bloqueado o esperando
- Terminado

No necesitas memorizar todos al principio, pero sí entender que un hilo no siempre está realmente trabajando.

## 8. Problema central: memoria compartida
Cuando dos hilos leen y escriben el mismo dato, aparece el gran reto.

Ejemplo:
```java
contador++;
```

Esta instrucción parece una sola cosa, pero en realidad implica leer, sumar y escribir. Si dos hilos la ejecutan a la vez, puedes perder incrementos.

Ese problema se llama condición de carrera.

## 9. Sincronización
La sincronización sirve para coordinar el acceso a recursos compartidos.

### `synchronized`
Protege una sección crítica para que solo un hilo entre a la vez.

```java
public synchronized void incrementar() {
    contador++;
}
```

### Bloques sincronizados
```java
public void incrementar() {
    synchronized (this) {
        contador++;
    }
}
```

La idea es evitar que varios hilos modifiquen simultáneamente el mismo estado sensible.

## 10. `volatile`
`volatile` no hace una operación atómica. Lo que hace es ayudar a que un hilo vea cambios hechos por otro.

Sirve para flags simples, no para operaciones complejas como incrementos.

Ejemplo típico:
```java
private volatile boolean detenido;
```

## 11. Locks y estructuras concurrentes
Java también ofrece herramientas más avanzadas:
- `Lock` y `ReentrantLock`
- `ReadWriteLock`
- `Semaphore`
- `CountDownLatch`
- `CyclicBarrier`
- `ConcurrentHashMap`
- `BlockingQueue`

Estas clases existen porque `synchronized` no cubre todos los escenarios con comodidad.

## 12. Executors y pools de hilos
En aplicaciones reales casi nunca gestionas hilos manualmente para todo.

Usas pools porque:
- Evitan crear y destruir hilos constantemente.
- Mejoran control y rendimiento.
- Permiten limitar carga.

Modelos comunes:
- Un hilo por tarea pequeña.
- Pool fijo.
- Pool cacheado.
- Programación programada con scheduler.

## 13. Futures y tareas asíncronas
Cuando envías una tarea a un executor, puedes recibir un `Future`.

Ese objeto representa un resultado que todavía puede no estar listo.

```java
Future<Integer> resultado = pool.submit(() -> 10 + 20);
```

Esto ayuda a trabajar con operaciones que tardan más tiempo.

## 14. Reglas de oro para no romper cosas
- No compartas estado mutable sin control.
- Prefiere inmutabilidad siempre que puedas.
- Usa `ExecutorService` antes que crear hilos a mano.
- Sincroniza solo lo necesario.
- No bloquees más de la cuenta.
- Diseña pensando en concurrencia desde el principio si el problema lo necesita.

## 15. Hilos en Java 21
Java 21 cambia mucho la experiencia con los virtual threads.

### Qué aportan
- Escalan mejor para muchas tareas bloqueantes.
- Reducen el coste de manejar muchísima concurrencia.
- Hacen más natural el modelo "una tarea, una ejecución".

### Qué no cambian
- No eliminan el problema de la memoria compartida.
- No sustituyen la necesidad de diseñar bien el acceso concurrente.
- No hacen mágicamente seguro cualquier código.

## 16. Qué debes aprender en orden
1. Entender proceso, hilo y memoria compartida.
2. Saber crear un hilo con `Runnable`.
3. Entender `start()` versus `run()`.
4. Aprender `synchronized` y `volatile`.
5. Usar `ExecutorService`.
6. Conocer estructuras concurrentes.
7. Entender virtual threads en Java 21.

## 17. Ejemplo mental completo
Imagina una app de pedidos:
- Un hilo recibe el pedido.
- Otro valida stock.
- Otro calcula impuestos.
- Otro guarda en base de datos.

Si todos tocan el mismo carrito sin control, aparecen errores.
Si cada uno trabaja sobre datos bien definidos y sincronizados cuando toca, el sistema escala correctamente.

## 18. Errores típicos de principiante
- Crear demasiados hilos manualmente.
- Confundir concurrencia con paralelismo.
- Usar `synchronized` para todo sin pensar.
- No cerrar recursos compartidos.
- Modificar objetos compartidos desde varios hilos sin protección.
- No entender que la visibilidad de memoria también importa.

## 19. Resumen rápido
- Un hilo es una línea de ejecución dentro de un proceso.
- La dificultad real está en la memoria compartida.
- `synchronized` coordina acceso, `volatile` ayuda con visibilidad, `ExecutorService` profesionaliza el manejo de tareas.
- Java 21 mejora muchísimo la concurrencia con virtual threads.
- La concurrencia bien hecha exige diseño, no solo sintaxis.
