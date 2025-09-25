# Manejo de Excepciones en Java (Checked, Unchecked y Personalizadas)

**Objetivo:** Dominar el modelo de excepciones en Java: diferencias entre *checked* y *unchecked*, uso correcto de `try`/`catch`/`finally` y `try-with-resources`, buenas prácticas, y creación de excepciones personalizadas. Incluye ejemplos listos para copiar y pegar, ejercicios y escenarios de aplicación real.

---

## 1) Conceptos clave

- **Excepción:** Evento anómalo durante la ejecución que interrumpe el flujo normal del programa. Permite manejar errores de forma estructurada.
- **Checked (verificadas):** El compilador obliga a **declararlas o capturarlas**. Suelen representar fallos **recuperables** (E/S, BD, red). Ej.: `IOException`, `SQLException`.
- **Unchecked (no verificadas):** Heredan de `RuntimeException`. **No** están forzadas por el compilador. Representan errores de **programación** (precondiciones no cumplidas). Ej.: `NullPointerException`, `IllegalArgumentException`.
- **Error:** Condiciones severas del runtime (`OutOfMemoryError`) que **no** se deben capturar normalmente.

Jerarquía simplificada:
```
Throwable
 ├─ Error
 └─ Exception
    ├─ RuntimeException (unchecked)
    └─ (checked excepciones)
```

---

## 2) Excepciones comunes y escenarios

### Unchecked (RuntimeException y derivadas)
- `NullPointerException` – referencia nula.  
  **Escenario:** Intentar acceder a un método o campo de un objeto que no ha sido inicializado.
- `IllegalArgumentException` – argumento inválido a un método.  
  **Escenario:** Pasar un valor negativo a un método que espera solo positivos.
- `IllegalStateException` – estado interno no válido para la operación.  
  **Escenario:** Llamar a un método en un objeto que no está listo para usarse.
- `IndexOutOfBoundsException` / `ArrayIndexOutOfBoundsException` – índices inválidos.  
  **Escenario:** Acceder a una posición fuera de los límites de un array o lista.
- `ArithmeticException` – división por cero, etc.  
  **Escenario:** Dividir un número entre cero.
- `NumberFormatException` – errores al parsear números.  
  **Escenario:** Convertir una cadena no numérica a número.

### Checked (Exception no-Runtime)
- `IOException` – fallos de entrada/salida (archivos, red).  
  **Escenario:** Leer un archivo que no existe o no se puede acceder.
- `SQLException` – errores de base de datos.  
  **Escenario:** Error al ejecutar una consulta SQL.
- `ParseException` – errores de parsing (fecha, texto).  
  **Escenario:** Analizar una fecha con formato incorrecto.
- `ClassNotFoundException` – clase no encontrada en classpath.  
  **Escenario:** Cargar una clase por reflexión que no está disponible.
- `FileNotFoundException` – archivo inexistente.  
  **Escenario:** Intentar abrir un archivo que no existe.

> **Regla práctica:** Condiciones que puedes **manejar/recuperar** suelen ser *checked*. **Errores de programación** o de **validación** deberían ser *unchecked* (lanza `IllegalArgumentException`, `IllegalStateException`, etc.).

---

## 3) try / catch / finally (y multi-catch)

El bloque `try` permite ejecutar código que puede lanzar excepciones.  
El bloque `catch` captura y maneja la excepción.  
El bloque `finally` se ejecuta siempre, ocurra o no una excepción (ideal para liberar recursos).

```java
import java.util.logging.Logger;

class EjemploTryCatch {
    private static final Logger log = Logger.getLogger(EjemploTryCatch.class.getName());

    static int parseAndDivide(String a, String b) {
        try {
            int x = Integer.parseInt(a);
            int y = Integer.parseInt(b);
            return x / y;
        } catch (NumberFormatException | ArithmeticException e) { // multi-catch
            log.warning("Entrada inválida: " + e.getMessage());
            // re-lanzar como unchecked con más contexto
            throw new IllegalArgumentException("Parámetros inválidos: a=" + a + ", b=" + b, e);
        } finally {
            // Siempre se ejecuta (salvo System.exit / fallo JVM)
            log.info("Finalizando parseAndDivide");
        }
    }
}
```

**Consejos**  
- Usa **multi-catch** (`|`) para agrupar excepciones que se manejan igual.  
- En `finally` **no** alteres el valor de retorno (difícil de depurar).  
- Evita `catch (Exception e)` genérico, salvo en **fronteras** de la aplicación (controlador web, capa CLI) para loguear y responder.

**Escenario real:**  
En una calculadora, puedes capturar errores de formato y división por cero para mostrar un mensaje amigable al usuario.

---

## 4) try-with-resources (cierres seguros y `suppressed`)

Para recursos `AutoCloseable` (streams, conexiones), se recomienda `try-with-resources`:

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

class LecturaArchivo {
    static String primeraLinea(String path) throws IOException { // checked
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        } // cierra automáticamente incluso si hay excepción
    }
}
```

- Si ocurre una excepción al **cerrar**, queda registrada como **suppressed** en la principal.
- Cómo inspeccionarlas:
```java
try {
    LecturaArchivo.primeraLinea("datos.txt");
} catch (IOException e) {
    for (Throwable s : e.getSuppressed()) {
        System.err.println("Suppressed: " + s);
    }
    throw e;
}
```

**Escenario real:**  
Al leer o escribir archivos, siempre usa `try-with-resources` para evitar fugas de recursos y asegurar el cierre correcto.

---

## 5) `throws` vs `throw`

- `throws` en la **firma** del método para **declarar** excepciones *checked* que pueden salir.
- `throw` **lanza** la excepción en un punto específico.

```java
static void validarEdad(int edad) {
    if (edad < 0) throw new IllegalArgumentException("La edad no puede ser negativa");
}

static String leer(String path) throws IOException {
    // el método delega el manejo al llamador
    return LecturaArchivo.primeraLinea(path);
}
```

**Escenario real:**  
Un método que lee archivos puede declarar `throws IOException` para que el llamador decida cómo manejar el error.

---

## 6) Excepciones personalizadas

Crea **checked** extendiendo `Exception`, o **unchecked** extendiendo `RuntimeException`.

### 6.1 Unchecked de dominio (validaciones y reglas de negocio)
```java
class DominioInvalidoException extends RuntimeException {
    public DominioInvalidoException(String message) { super(message); }
    public DominioInvalidoException(String message, Throwable cause) { super(message, cause); }
}
```

### 6.2 Checked para operaciones recuperables (IO/Integraciones)
```java
class ServicioIndisponibleException extends Exception {
    public ServicioIndisponibleException(String message) { super(message); }
    public ServicioIndisponibleException(String message, Throwable cause) { super(message, cause); }
}
```

### Uso en una capa de servicio
```java
class BancoService {
    // Unchecked para precondiciones de negocio
    public void transferir(String origen, String destino, double monto) {
        if (origen == null || destino == null) {
            throw new DominioInvalidoException("Cuentas no pueden ser nulas");
        }
        if (monto <= 0) {
            throw new DominioInvalidoException("Monto debe ser positivo");
        }
        // ... lógica de transferencia
    }

    // Checked para fallos de integración que el llamador podría reintentar
    public String consultarSaldoRemoto(String cuenta) throws ServicioIndisponibleException {
        try {
            // simular llamada remota que puede fallar
            throw new IOException("Timeout consultando al proveedor");
        } catch (IOException e) {
            // agrega contexto y preserva la causa
            throw new ServicioIndisponibleException("No fue posible consultar saldo de " + cuenta, e);
        }
    }
}
```

**Escenario real:**  
En un sistema bancario, puedes lanzar excepciones personalizadas para reglas de negocio (como transferencias inválidas) y para errores de integración (como servicios externos caídos).

---

## 7) Encadenamiento (wrapping) y rethrow con causa

**Preserva la pila y el contexto** usando el segundo parámetro (*cause*).

```java
try {
    ejecutar();
} catch (SQLException e) {
    throw new IllegalStateException("Falló al ejecutar consulta [" + sqlId + "]", e);
}
```

> Nunca “tragues” excepciones: **loguea** y **propaga** con contexto si no puedes manejar.

**Escenario real:**  
En una aplicación web, puedes capturar excepciones de base de datos y lanzar una excepción de negocio con más contexto para la capa superior.

---

## 8) Buenas prácticas (checklist)

- ✅ **Usa unchecked** para **precondiciones** y errores de programación (`IllegalArgumentException`, `IllegalStateException`).  
- ✅ **Usa checked** cuando el llamador **se pueda recuperar** (E/S, red) y tenga sentido **forzar manejo**.  
- ✅ **Agrega contexto** al re-lanzar (qué operación, ids, parámetros).  
- ✅ **Preserva la causa** (`new MiExcepcion("msg", e)`).  
- ✅ **try-with-resources** siempre que cierres recursos.  
- ✅ **No uses excepciones para control de flujo** normal.  
- ✅ **Registra con logging**, no con `printStackTrace()` en producción.  
- ✅ **Mensajes claros** (sin datos sensibles).  
- ✅ **Capa frontera**: captura genérica para transformar en respuesta de usuario/API y loguear.  

---

## 9) Logging rápido (java.util.logging / SLF4J)

Ejemplo con `java.util.logging`:
```java
import java.util.logging.Level;
import java.util.logging.Logger;

class LogEjemplo {
    private static final Logger log = Logger.getLogger(LogEjemplo.class.getName());

    void procesar() {
        try {
            // ...
        } catch (Exception e) {
            log.log(Level.SEVERE, "Error procesando solicitud id={0}", 42);
            log.log(Level.FINE, "Detalle", e); // stack completo en nivel fino
            throw e;
        }
    }
}
```

Con **SLF4J** + Logback (recomendado en proyectos grandes):
```java
// build.gradle: implementation("org.slf4j:slf4j-api:2.0.13"), runtimeOnly("ch.qos.logback:logback-classic:1.5.6")
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

class LogSLF4J {
    private static final Logger log = LoggerFactory.getLogger(LogSLF4J.class);
    void procesar() {
        try {
            // ...
        } catch (Exception e) {
            log.error("Error procesando solicitud id={}", 42, e);
            throw e;
        }
    }
}
```

**Escenario real:**  
En aplicaciones empresariales, el logging estructurado permite rastrear errores y diagnosticar problemas en producción.

---

## 10) Casos prácticos completos

### 10.1 Lectura robusta de archivo
```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class LectorSeguro {
    public static int contarLineas(String path) throws IOException {
        int count = 0;
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            while (br.readLine() != null) count++;
        }
        return count;
    }

    public static void main(String[] args) {
        try {
            System.out.println("Líneas: " + contarLineas("data.txt"));
        } catch (IOException e) {
            System.err.println("No se pudo leer el archivo: " + e.getMessage());
        }
    }
}
```
**Escenario real:**  
Procesar archivos de datos donde puede haber errores de lectura o archivos faltantes.

### 10.2 Validación de negocio con excepciones personalizadas
```java
class SaldoInsuficienteException extends RuntimeException {
    public SaldoInsuficienteException(String msg) { super(msg); }
}

class Cuenta {
    private double saldo;

    public Cuenta(double saldoInicial) {
        if (saldoInicial < 0) throw new IllegalArgumentException("Saldo inicial negativo");
        this.saldo = saldoInicial;
    }

    public void debitar(double monto) {
        if (monto <= 0) throw new IllegalArgumentException("Monto inválido");
        if (monto > saldo) throw new SaldoInsuficienteException("Saldo insuficiente: " + saldo);
        saldo -= monto;
    }
}
```
**Escenario real:**  
Controlar operaciones bancarias y evitar que un usuario gire más dinero del que tiene disponible.

### 10.3 Conversión checked → unchecked en capa frontera
```java
class ControladorCLI {
    public static void main(String[] args) {
        BancoService svc = new BancoService();
        try {
            String saldo = svc.consultarSaldoRemoto("123-XYZ");
            System.out.println("Saldo=" + saldo);
        } catch (ServicioIndisponibleException e) {
            // Se decide política: log + mensaje amable
            System.err.println("Servicio temporalmente no disponible. Intente más tarde.");
            // Opcional: convertir a unchecked si tu capa superior no maneja checked
            throw new IllegalStateException("Fallo consultando saldo", e);
        }
    }
}
```
**Escenario real:**  
En una aplicación de línea de comandos o API, puedes capturar excepciones checked y transformarlas en mensajes de usuario o en excepciones unchecked para simplificar el manejo en capas superiores.

---

## 11) Ejercicios propuestos

1. **Refactorizar un parser:** Diseña un método `parsearEntero(String s)` que devuelva `OptionalInt` y **no lance excepción** ante entradas inválidas; registra con logging a nivel `FINE` el detalle.  
2. **Diseñar jerarquía de excepciones** para un módulo de pagos: `PagoException` (base), `MedioNoSoportadoException` (unchecked), `IntegracionPagoException` (checked, con `cause`). Implementa un servicio que use ambas.  
3. **try-with-resources:** Escribe un método que copie un archivo grande `copy(String src, String dst)` y mida el tiempo; captura `IOException`, re-lanza con contexto del path. Agrega impresión de `getSuppressed()`.  
4. **API frontera:** Crea un mini CLI que invoque un servicio que puede lanzar `SQLException`. En la capa CLI captura y transforma a mensaje de usuario sin exponer detalles sensibles; registra stack trace con SLF4J.  
5. **Test unitarios:** Usa JUnit 5 y `assertThrows` para validar que `Cuenta.debitar()` lance `SaldoInsuficienteException` cuando corresponde.

---

## 12) Snippets útiles (copiar y pegar)

**Multi-catch + rethrow con contexto**
```java
try {
    // ...
} catch (IOException | SQLException e) {
    throw new IllegalStateException("Falló proceso de importación", e);
}
```

**Validación de parámetros (unchecked)**
```java
static void requirePositive(int n) {
    if (n <= 0) throw new IllegalArgumentException("n debe ser > 0, fue " + n);
}
```

**Capturar y registrar `suppressed`**
```java
catch (Exception e) {
    for (Throwable s : e.getSuppressed()) {
        System.err.println("Suppressed: " + s);
    }
    throw e;
}
```

---

## 13) Tabla de referencia rápida

| Tipo | Jerarquía | Ejemplos | ¿El compilador fuerza manejo? | ¿Cuándo usar? |
|---|---|---|---|---|
| **Checked** | `Exception` (no `RuntimeException`) | `IOException`, `SQLException` | **Sí** (`throws`/`catch`) | Fallos recuperables de IO/BD/red |
| **Unchecked** | `RuntimeException` | `IllegalArgumentException`, `NPE`, `IndexOutOfBounds` | No | Precondiciones/errores de programación |
| **Error** | `Error` | `OutOfMemoryError`, `StackOverflowError` | No | No capturar salvo casos muy específicos |

---

## 14) Más escenarios de aplicación real

- **Aplicaciones web:**  
  Captura excepciones en controladores para devolver mensajes de error claros al usuario y registrar el error para diagnóstico.
- **Procesamiento por lotes:**  
  Usa excepciones para saltar registros inválidos y continuar procesando el resto del lote.
- **Integración con servicios externos:**  
  Lanza excepciones checked cuando una API externa falla, permitiendo reintentos o fallback.
- **Validación de formularios:**  
  Lanza excepciones unchecked para datos inválidos antes de guardar en base de datos.
- **Sistemas embebidos:**  
  Maneja errores de hardware (checked) y condiciones inesperadas (unchecked) para mantener la estabilidad del sistema.

---

## 15) Conclusiones

- Distingue entre **errores de programación** (unchecked) y **fallos recuperables** (checked).  
- **Preserva contexto y causa** al re-lanzar.  
- **Cierra recursos** con `try-with-resources`.  
- Define **excepciones personalizadas** para comunicar reglas de negocio claramente.  
- Centraliza el manejo en **capas frontera** (controladores, CLI) con buen **logging**.
- Aplica el manejo de excepciones para mejorar la robustez, mantenibilidad y experiencia de usuario en tus aplicaciones Java.

---