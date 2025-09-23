# Patrones de Diseño en Java 

> Guía práctica y profundamente explicada para aplicar patrones de diseño clave en proyectos Java modernos.

---

## 1) ¿Qué son los patrones de diseño y por qué importan?

Los **patrones de diseño** son soluciones reutilizables a problemas comunes de diseño de software. No son librerías ni frameworks, sino **modelos** que orientan cómo estructurar clases, interfaces y colaboraciones entre objetos. Usarlos bien te ayuda a:

- **Hacer el código más mantenible** y fácil de evolucionar.
- **Reducir acoplamiento** y mejorar la cohesión.
- **Comunicarte mejor** con otros desarrolladores (un nombre compartido para una solución compartida).
- **Evitar “reinventar la rueda”** y caer en _anti‑patrones_ (por ejemplo, abuso de `new`, condicionales gigantes, clases Dios, etc.).

**Cuándo NO usarlos:** si el problema es trivial, si un patrón complica más de lo que simplifica, o si una característica del lenguaje/Framework ya resuelve el caso de forma más simple (p. ej., inyección de dependencias en Spring).

---

## 2) Clasificación (GoF) y mapa mental rápido

Los patrones clásicos del catálogo _Gang of Four (GoF)_ se agrupan en tres familias:

- **Creacionales:** se enfocan en **cómo crear objetos** aislando la creación del uso.
  - _Incluyen:_ Singleton, Factory Method, Abstract Factory, Builder, Prototype.
- **Estructurales:** definen **cómo componer** clases/objetos para formar estructuras flexibles.
  - _Incluyen:_ Adapter, Composite, Decorator, Facade, Proxy.
- **De Comportamiento:** organizan **cómo colaboran** los objetos y cómo reparten responsabilidades.
  - _Incluyen:_ Strategy, Template Method, Observer, Command, State, Iterator.

> En este README profundizamos en **Singleton, Factory Method, Strategy, Template Method, Observer, Adapter, Decorator y Facade**.

---

## 3) Patrones Creacionales

### 3.1 Singleton — **Una única instancia global, controlada**

**Problema:** necesitas exactamente **una** instancia compartida (p. ej., configuración, logger, caché simple).  
**Solución:** restringe la construcción (`constructor` privado) y expón un **punto de acceso** controlado.

**Implementación segura en entornos multi‑hilo (double‑checked locking):**

```java
public final class AppConfig {
    private static volatile AppConfig INSTANCE; // 'volatile' evita reordenamientos peligrosos

    private AppConfig() { /* cargar propiedades, etc. */ }

    public static AppConfig getInstance() {
        if (INSTANCE == null) {
            synchronized (AppConfig.class) {
                if (INSTANCE == null) {
                    INSTANCE = new AppConfig(); // Se crea la instancia solo una vez
                }
            }
        }
        return INSTANCE; // Siempre devuelve la misma instancia
    }
}
```
**Explicación:**  
- El constructor es privado, así nadie puede crear instancias fuera de la clase.
- El método `getInstance()` asegura que solo se cree una instancia, incluso si varios hilos acceden al mismo tiempo.
- El uso de `volatile` y `synchronized` evita problemas de concurrencia.

**Alternativa recomendada:** _Enum Singleton_ (simple, a prueba de serialización y reflexión en la práctica):

```java
public enum GlobalLogger {
    INSTANCE;
    public void log(String msg) { System.out.println("[LOG] " + msg); }
}
```
**Explicación:**  
- Usar un `enum` garantiza que solo exista una instancia.
- Es seguro ante serialización y reflexión, y muy fácil de usar.

**Pros:** acceso global controlado, ciclo de vida único.  
**Contras:** puede ocultar **estado global** (dificulta pruebas y paralelismo).  
**Buenas prácticas:** mantenerlo **sin estado** o con estado inmutable; aislar acceso con interfaces cuando sea viable; en Spring, preferir **beans singleton** sobre crear tu propio patrón.

---

### 3.2 Factory Method — **Centraliza y encapsula la creación**

**Problema:** el cliente no debería conocer clases concretas ni `new` por todas partes.  
**Solución:** mueve la **lógica de construcción** a un _factory_ que decide qué implementación devolver.

```java
interface Figura { void dibujar(); }

class Circulo implements Figura {
    public void dibujar() { System.out.println("Dibujando círculo"); }
}

class Rectangulo implements Figura {
    public void dibujar() { System.out.println("Dibujando rectángulo"); }
}

final class FiguraFactory {
    private FiguraFactory() {}

    public static Figura crear(String tipo) {
        return switch (tipo.toLowerCase()) {
            case "circulo"    -> new Circulo();
            case "rectangulo" -> new Rectangulo();
            default -> throw new IllegalArgumentException("Tipo no soportado: " + tipo);
        };
    }
}

// Uso
Figura f1 = FiguraFactory.crear("circulo");
Figura f2 = FiguraFactory.crear("rectangulo");
f1.dibujar();
f2.dibujar();
```
**Explicación:**  
- El cliente no usa `new` directamente, sino que llama a la fábrica.
- La fábrica decide qué clase concreta crear según el parámetro.
- Si se agrega una nueva figura, solo se modifica la fábrica, no el código cliente.

**Pros:** desacopla cliente de clases concretas; punto único para validar/parámetros/configuración.  
**Contras:** si la familia crece, el `switch` o el registro pueden crecer también.  
**Variantes útiles:** registro de creadores por nombre/clave, lectura desde configuración, reflexión con validaciones, o _provider_ por inyección de dependencias.

---

## 4) Patrones de Comportamiento

### 4.1 Strategy — **Algoritmos intercambiables**

**Problema:** cambiar el algoritmo sin modificar el cliente (p. ej., reglas de precio/descuento, métodos de pago, validaciones).  
**Solución:** define una **interfaz** para el algoritmo y múltiples **estrategias** concretas; el **contexto** delega en la estrategia actual.

```java
interface Descuento {
    double aplicar(double monto);
}

class DescuentoEstudiante implements Descuento {
    public double aplicar(double monto) { return monto * 0.80; } // -20%
}

class DescuentoTemporada implements Descuento {
    public double aplicar(double monto) { return monto * 0.90; } // -10%
}

class Carrito {
    private Descuento estrategia;

    public Carrito(Descuento estrategia) { this.estrategia = estrategia; }
    public void setEstrategia(Descuento nueva) { this.estrategia = nueva; }

    public double total(double subtotal) { return estrategia.aplicar(subtotal); }
}

// Uso
Carrito c = new Carrito(new DescuentoEstudiante());
double t1 = c.total(100.0);        // 80.0
c.setEstrategia(new DescuentoTemporada());
double t2 = c.total(100.0);        // 90.0
```
**Explicación:**  
- La interfaz `Descuento` define el método común.
- Cada clase concreta implementa el algoritmo específico.
- El `Carrito` puede cambiar la estrategia en tiempo de ejecución, permitiendo flexibilidad.

**Pros:** reemplazo en caliente; pruebas unitarias sencillas por clase concreta.  
**Contras:** más clases pequeñas; requiere orquestación para elegir la estrategia adecuada.  
**Buenas prácticas:** exponer estrategias por configuración/DI; documentar pre/postcondiciones.

---

### 4.2 Template Method — **Esqueleto de proceso con pasos personalizables**

**Problema:** varios procesos comparten la misma **secuencia** pero difieren en **algunos pasos** (p. ej., flujo de pago).  
**Solución:** una clase abstracta define el **algoritmo** (método `final`) y delega pasos variables a **métodos abstractos**/hook.

```java
abstract class ProcesoPago {
    public final void ejecutar(double monto) {
        validar(monto);
        autorizar(monto);
        capturar(monto);
        notificar(monto);
    }
    protected void validar(double monto) {
        if (monto <= 0) throw new IllegalArgumentException("Monto inválido");
    }
    protected abstract void autorizar(double monto);
    protected abstract void capturar(double monto);
    protected void notificar(double monto) {
        System.out.println("Notificación enviada por $" + monto);
    }
}

class PagoTarjeta extends ProcesoPago {
    protected void autorizar(double monto) { /* tokenizar, antifraude... */ }
    protected void capturar(double monto)  { /* captura en pasarela */ }
}
```
**Explicación:**  
- El método `ejecutar` define el flujo completo y no puede ser modificado por las subclases.
- Los pasos variables (`autorizar`, `capturar`) se implementan en las subclases.
- El método `notificar` puede ser sobrescrito si se requiere personalización.

**Pros:** evita duplicación del flujo; fuerza orden y contratos.  
**Contras:** herencia rígida vs. composición; muchos _hooks_ pueden confundir.  
**Alternativas:** **Strategy** si la variación es más sustancial y se requieren combinaciones libres.

---

### 4.3 Observer — **Notificaciones automáticas y desacopladas**

**Problema:** al cambiar el estado de un objeto, múltiples interesados deben ser **notificados** sin acoplarse fuertemente.  
**Solución:** un **Sujeto** mantiene sus **Observadores**; cuando cambia, los **notifica**.

```java
import java.util.ArrayList;
import java.util.List;

interface Observador {
    void actualizar(String evento);
}

class Sujeto {
    private final List<Observador> obs = new ArrayList<>();
    public void suscribir(Observador o) { obs.add(o); }
    public void desuscribir(Observador o) { obs.remove(o); }
    public void notificar(String evento) {
        obs.forEach(o -> o.actualizar(evento));
    }
}
```
**Explicación:**  
- El `Sujeto` mantiene una lista de observadores.
- Cuando ocurre un evento, llama al método `actualizar` de cada observador.
- Los observadores pueden suscribirse o desuscribirse en cualquier momento.

**Casos de uso:** eventos de dominio, UI, _pub/sub_, auditoría.  
**Buenas prácticas:** definir **payloads** claros; manejar errores de observadores para no bloquear al sujeto.  
**En ecosistemas modernos:** usa `ApplicationEventPublisher` en Spring para desacoplar eventos internos.

---

## 5) Patrones Estructurales

### 5.1 Adapter — **Compatibiliza interfaces incompatibles**

**Problema:** tu código cliente espera una interfaz, pero la biblioteca/sistema legacy ofrece otra distinta.  
**Solución:** un **Adapter** traduce llamadas del cliente hacia el proveedor real.

```java
// Cliente espera ReproductorWav
interface ReproductorWav { void reproducirWav(String archivoWav); }

// Biblioteca externa solo reproduce MP3
class ReproductorMp3 { void playMp3(String archivoMp3) { /* ... */ } }

// Adapter
class Mp3ToWavAdapter implements ReproductorWav {
    private final ReproductorMp3 mp3 = new ReproductorMp3();
    public void reproducirWav(String archivoWav) {
        String convertido = archivoWav.replace(".wav", ".mp3");
        mp3.playMp3(convertido);
    }
}
```
**Explicación:**  
- El cliente sigue usando su interfaz esperada (`ReproductorWav`).
- El adapter traduce la llamada y la adapta a la interfaz de la biblioteca externa (`ReproductorMp3`).
- Así se logra compatibilidad sin modificar el cliente ni la biblioteca.

**Pros:** integración sin tocar ni el cliente ni la lib externa.  
**Contras:** puede proliferar si hay muchas combinaciones.  
**Tip:** documenta supuestos de conversión (nombres de archivo, formatos, errores).

---

### 5.2 Decorator — **Añade responsabilidades dinámicamente**

**Problema:** quieres **extender comportamiento** de un objeto sin crear una jerarquía enorme de herencias.  
**Solución:** envuelve el objeto con **decoradores** que agregan pasos antes/después delegando al componente real.

```java
interface Notificador { void enviar(String mensaje); }

class NotificadorBase implements Notificador {
    public void enviar(String mensaje) { System.out.println("Notificación: " + mensaje); }
}

class DecoradorEmail implements Notificador {
    private final Notificador wrap;
    public DecoradorEmail(Notificador wrap) { this.wrap = wrap; }
    public void enviar(String mensaje) {
        wrap.enviar(mensaje);
        System.out.println("EMAIL -> " + mensaje);
    }
}

class DecoradorSms implements Notificador {
    private final Notificador wrap;
    public DecoradorSms(Notificador wrap) { this.wrap = wrap; }
    public void enviar(String mensaje) {
        wrap.enviar(mensaje);
        System.out.println("SMS -> " + mensaje);
    }
}

// Uso: composiciones dinámicas
Notificador n = new DecoradorSms(new DecoradorEmail(new NotificadorBase()));
n.enviar("Tu pedido fue enviado");
```
**Explicación:**  
- Cada decorador recibe el objeto base y puede agregar funcionalidad antes o después de delegar la llamada.
- Puedes combinar decoradores en cualquier orden, logrando flexibilidad y extensibilidad.
- El objeto base nunca se modifica, solo se envuelve.

**Pros:** composición flexible; combinaciones a demanda.  
**Contras:** demasiados envoltorios pueden afectar legibilidad/depuración.  
**Tip:** nombra bien cada decorador y documenta el orden esperado.

---

### 5.3 Facade — **Interfaz simple para un subsistema complejo**

**Problema:** múltiples servicios/objetos deben coordinarse; el cliente no debería lidiar con esa complejidad.  
**Solución:** una **fachada** orquesta llamadas y expone una API **simple y segura**.

```java
class Validador { void validar(double m) { if (m <= 0) throw new IllegalArgumentException(); } }
class Autorizador { void autorizar(double m) { /* ... */ } }
class Capturador { void capturar(double m) { /* ... */ } }

class PagoFacade {
    private final Validador validador = new Validador();
    private final Autorizador autorizador = new Autorizador();
    private final Capturador capturador = new Capturador();

    public void pagar(double monto) {
        validador.validar(monto);
        autorizador.autorizador(monto);
        capturador.capturar(monto);
    }
}
```
**Explicación:**  
- El cliente solo interactúa con la fachada (`PagoFacade`), que se encarga de coordinar los servicios internos.
- Esto simplifica el uso y reduce el acoplamiento.
- Si el subsistema cambia, solo se modifica la fachada, no el código cliente.

**Pros:** reduce complejidad para el cliente; punto central para **validaciones/transacciones**.  
**Contras:** riesgo de convertirse en “clase Dios” si no delega bien; mantén el subsistema coherente y testeable.

---

## 6) Consejos de uso en proyectos reales (Java/Spring)

- **Preferir composición a herencia**: Strategy/Decorator/Facade suelen envejecer mejor que jerarquías profundas.
- **Inyección de dependencias (DI)**: en Spring, la mayoría de _singletons_ los gestiona el contenedor; usa **interfaces** + **@Primary/@Qualifier** para elegir estrategias.
- **Transaccionalidad**: si usas Facade para orquestar casos de uso, encapsula límites con `@Transactional` donde corresponda.
- **Pruebas**: prueba cada estrategia/decorador de forma **unitaria** y la orquestación de forma **integrada**.
- **Observabilidad**: en Decorator, añade logging/metrics sin contaminar clases base.
- **Nomenclatura**: nombra los patrones en el código (p. ej., `PaymentFacade`, `DiscountStrategy`) para comunicar intención.

---

## 7) Checklist rápida para elegir patrón

- ¿Cambio de algoritmo en caliente? → **Strategy**  
- ¿Mismo flujo con variaciones puntuales? → **Template Method**  
- ¿Eventos y suscriptores desacoplados? → **Observer**  
- **API incompatible** que debo integrar → **Adapter**  
- ¿Sumar funcionalidades sin heredar? → **Decorator**  
- ¿Orquestar subsistema complejo en una API simple? → **Facade**  
- ¿Una sola instancia global controlada? → **Singleton**  
- ¿Evitar `new` dispersos y ocultar clases concretas? → **Factory Method**

---

## 8) Retos de práctica (hands‑on)

1. **Factory + Strategy**: crea una fábrica de métodos de pago que devuelva estrategias (`Tarjeta`, `Efectivo`, `Crypto`) y calcula totales con y sin comisiones.
2. **Template + Observer**: diseña un checkout que, al finalizar, dispare eventos para notificar a _email_, _auditoría_ y _analytics_.
3. **Decorator avanzado**: agrega **cifrado** y **reintentos** a un `Notificador` sin tocar su implementación base.
4. **Adapter real**: integra una librería externa (formato A) en un cliente que espera formato B, con manejo de errores y _timeouts_.

---

## 9) Glosario breve

- **Acoplamiento/Cohesión**: cuánto dependen entre sí los módulos / qué tan enfocada es una clase en una única responsabilidad.
- **DI/IoC**: inversión de control; el contenedor crea y te inyecta dependencias.
- **Contrato**: interfaz y garantías (pre/postcondiciones) que ofrece un componente.

---

## 10) Referencia

Esta guía se inspira en una **guía rápida de patrones de diseño en Java** y la amplía con ejemplos y consejos para proyectos actuales.