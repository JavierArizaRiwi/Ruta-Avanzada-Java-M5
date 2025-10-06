# Manejo de Excepciones en Java SE (con JOptionPane) 

## 1. Introducción

Las excepciones son eventos anómalos que ocurren durante la ejecución y que pueden interrumpir el flujo normal del programa. En Java, el manejo de excepciones permite controlar errores sin que la aplicación termine abruptamente. En esta guía trabajaremos **exclusivamente con entradas y mensajes mediante `JOptionPane`**, manteniendo el estilo usado en tus materiales.

### Objetivos
- Comprender el flujo del manejo de excepciones (`try` / `catch` / `finally`).
- Distinguir entre excepciones **verificadas (checked)** y **no verificadas (unchecked)**.
- Crear y usar **excepciones personalizadas** para reglas de negocio.
- Integrar el manejo de errores en una app Java SE basada en **JOptionPane**.

---

## 2. Conceptos clave

| Tipo | Ejemplos | ¿Obliga el compilador a capturar o propagar? |
|---|---|---|
| **Checked (verificadas)** | `IOException`, `SQLException`, `FileNotFoundException` | **Sí** |
| **Unchecked (no verificadas)** | `NullPointerException`, `ArithmeticException`, `IllegalArgumentException` | **No necesariamente** |
| **Error** | `OutOfMemoryError`, `StackOverflowError` | No deben manejarse en lógica de negocio |

**Regla práctica**: usa checked para condiciones que **se esperan** y pueden tratarse (I/O, BD); usa unchecked para **programming errors** o validaciones de argumentos (`IllegalArgumentException`).

---

## 3. Estructura básica del manejo de excepciones

```java
try {
    // Código que puede fallar
} catch (TipoDeExcepcion e) {
    // Manejo específico
} catch (Exception e) {
    // Manejo genérico (evítalo si puedes)
} finally {
    // Siempre se ejecuta (liberar recursos, mensajes finales, etc.)
}
```

---

## 4. Proyecto de ejemplo (con JOptionPane)

### 4.1 Estructura de paquetes

```
src/
 └── com.mycompany.excepciones
      ├── domain/
      │    └── Estudiante.java
      ├── exceptions/
      │    ├── PromedioInvalidoException.java
      │    └── EstudianteNoEncontradoException.java
      ├── service/
      │    └── EstudianteService.java
      └── ui/
           └── AppMain.java
```

> Nota: el almacenamiento será en memoria (lista). Puedes cambiarlo luego por JDBC y mantener la misma estrategia de excepciones.

---

### 4.2 Domain

```java
package com.mycompany.excepciones.domain;

public class Estudiante {
    private int id;
    private String nombre;
    private double promedio;

    public Estudiante(int id, String nombre, double promedio) {
        this.id = id;
        this.nombre = nombre;
        this.promedio = promedio;
    }

    public int getId() { return id; }
    public String getNombre() { return nombre; }
    public double getPromedio() { return promedio; }

    @Override
    public String toString() {
        return id + " - " + nombre + " - Promedio: " + promedio;
    }
}
```

---

### 4.3 Excepciones personalizadas

```java
package com.mycompany.excepciones.exceptions;

// Checked: obliga a manejar o propagar
public class PromedioInvalidoException extends Exception {
    public PromedioInvalidoException(String mensaje) {
        super(mensaje);
    }
}
```

```java
package com.mycompany.excepciones.exceptions;

// Unchecked: útil para búsquedas que fallan si la lógica espera el recurso
public class EstudianteNoEncontradoException extends RuntimeException {
    public EstudianteNoEncontradoException(String mensaje) {
        super(mensaje);
    }
}
```

---

### 4.4 Service (reglas de negocio + validaciones)

```java
package com.mycompany.excepciones.service;

import com.mycompany.excepciones.domain.Estudiante;
import com.mycompany.excepciones.exceptions.PromedioInvalidoException;
import com.mycompany.excepciones.exceptions.EstudianteNoEncontradoException;

import java.util.ArrayList;
import java.util.List;

public class EstudianteService {
    private final List<Estudiante> estudiantes = new ArrayList<>();

    public void agregarEstudiante(int id, String nombre, double promedio) throws PromedioInvalidoException {
        if (nombre == null || nombre.isBlank()) {
            throw new IllegalArgumentException("El nombre no puede estar vacío");
        }
        if (promedio < 0 || promedio > 5) {
            throw new PromedioInvalidoException("El promedio debe estar entre 0 y 5");
        }
        estudiantes.add(new Estudiante(id, nombre, promedio));
    }

    public List<Estudiante> listar() {
        return new ArrayList<>(estudiantes);
    }

    public Estudiante buscarPorId(int id) {
        return estudiantes.stream()
                .filter(e -> e.getId() == id)
                .findFirst()
                .orElseThrow(() -> new EstudianteNoEncontradoException("No existe estudiante con id " + id));
    }

    public void eliminarPorId(int id) {
        Estudiante e = buscarPorId(id); // puede lanzar EstudianteNoEncontradoException
        estudiantes.remove(e);
    }
}
```

---

### 4.5 UI (JOptionPane)

```java
package com.mycompany.excepciones.ui;

import com.mycompany.excepciones.service.EstudianteService;
import com.mycompany.excepciones.domain.Estudiante;
import com.mycompany.excepciones.exceptions.PromedioInvalidoException;

import javax.swing.JOptionPane;
import java.util.stream.Collectors;

public class AppMain {
    public static void main(String[] args) {
        EstudianteService service = new EstudianteService();
        boolean continuar = true;

        while (continuar) {
            String opcion = JOptionPane.showInputDialog(null,
                "MENÚ ESTUDIANTES\n" +
                "1. Agregar\n" +
                "2. Listar\n" +
                "3. Buscar por ID\n" +
                "4. Eliminar por ID\n" +
                "0. Salir", "Menú", JOptionPane.QUESTION_MESSAGE);

            if (opcion == null) break; // Cancelar

            try {
                switch (opcion) {
                    case "1" -> agregar(service);
                    case "2" -> listar(service);
                    case "3" -> buscar(service);
                    case "4" -> eliminar(service);
                    case "0" -> continuar = false;
                    default -> JOptionPane.showMessageDialog(null, "Opción inválida");
                }
            } catch (PromedioInvalidoException e) {
                JOptionPane.showMessageDialog(null, "Error de negocio: " + e.getMessage(), "Validación", JOptionPane.WARNING_MESSAGE);
            } catch (IllegalArgumentException e) {
                JOptionPane.showMessageDialog(null, "Argumento inválido: " + e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            } catch (Exception e) {
                JOptionPane.showMessageDialog(null, "Error inesperado: " + e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            }
        }

        JOptionPane.showMessageDialog(null, "Aplicación finalizada.");
    }

    private static void agregar(EstudianteService service) throws PromedioInvalidoException {
        String idStr = JOptionPane.showInputDialog("ID:");
        if (idStr == null) return;
        int id = parseEntero(idStr);

        String nombre = JOptionPane.showInputDialog("Nombre:");
        if (nombre == null) return;

        String promStr = JOptionPane.showInputDialog("Promedio (0 a 5):");
        if (promStr == null) return;
        double promedio = parseDouble(promStr);

        service.agregarEstudiante(id, nombre, promedio);
        JOptionPane.showMessageDialog(null, "Estudiante agregado.");
    }

    private static void listar(EstudianteService service) {
        String listado = service.listar().stream()
                .map(Estudiante::toString)
                .collect(Collectors.joining("\n"));
        JOptionPane.showMessageDialog(null, listado.isEmpty() ? "Sin registros" : listado);
    }

    private static void buscar(EstudianteService service) {
        String idStr = JOptionPane.showInputDialog("ID a buscar:");
        if (idStr == null) return;
        int id = parseEntero(idStr);

        var e = service.buscarPorId(id); // Puede lanzar EstudianteNoEncontradoException (unchecked)
        JOptionPane.showMessageDialog(null, e.toString());
    }

    private static void eliminar(EstudianteService service) {
        String idStr = JOptionPane.showInputDialog("ID a eliminar:");
        if (idStr == null) return;
        int id = parseEntero(idStr);

        service.eliminarPorId(id); // Puede lanzar EstudianteNoEncontradoException (unchecked)
        JOptionPane.showMessageDialog(null, "Eliminado correctamente.");
    }

    // Utilidades de parseo con manejo de NumberFormatException
    private static int parseEntero(String valor) {
        try {
            return Integer.parseInt(valor.trim());
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException("Se esperaba un número entero");
        }
    }

    private static double parseDouble(String valor) {
        try {
            return Double.parseDouble(valor.trim());
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException("Se esperaba un número decimal");
        }
    }
}
```

**Claves didácticas del ejemplo**  
- `try/catch` en el **loop de menú** centraliza el manejo de errores de UI.  
- Excepciones de **negocio** (`PromedioInvalidoException`) son **checked** y obligan a tratarlas.  
- Validaciones de **argumentos** (`IllegalArgumentException`) son **unchecked** (errores de entrada).  
- Búsquedas fallidas arrojan **unchecked** (`EstudianteNoEncontradoException`) y se resuelven arriba.  
- El usuario recibe mensajes claros usando `JOptionPane` con diferentes íconos (informativo, warning, error).

---

## 5. Buenas prácticas

- Manejar excepciones **en el nivel adecuado** (UI muestra mensajes; Service valida reglas; Domain no debería contener lógica de manejo).
- **No abuses** de `catch (Exception e)`; prefiere capturas específicas.
- Lanza `IllegalArgumentException` para argumentos inválidos y **documenta** en Javadoc.
- Usa excepciones personalizadas para **reglas de negocio** (legibilidad y trazabilidad).
- Centraliza el logging/mensajes en un punto (aquí, el loop del menú).
- Para I/O o BD, usa **try-with-resources** y traduce excepciones técnicas a mensajes de negocio.

---

## 6. Ejercicios propuestos

1. Agrega una excepción personalizada `NombreDuplicadoException` cuando se intente registrar un estudiante con **ID ya existente** o nombre duplicado (elige la regla). Integra su manejo en UI.
2. Implementa una opción de **actualizar promedio** que valide el rango y use `PromedioInvalidoException`.
3. Simula persistencia en archivo CSV. Maneja `IOException` con **try-with-resources** y muestra mensajes con `JOptionPane`.
4. Añade una búsqueda por nombre (case-insensitive) y arroja `EstudianteNoEncontradoException` si no hay coincidencias.
5. Integra una capa DAO con JDBC; traduce `SQLException` a una `DataAccessException` personalizada y maneja el mensaje en UI.

---

## 7. Conclusión

El manejo adecuado de excepciones con `JOptionPane` permite construir aplicaciones de escritorio sencillas **robustas y amigables** para el usuario. Las **excepciones personalizadas** hacen explícitas las reglas de negocio y simplifican el soporte y la evolución del sistema.