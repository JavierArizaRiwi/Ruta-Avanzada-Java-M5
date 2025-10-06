# Arquitectura por Capas en Java SE

## 1. Introducción

La arquitectura por capas es un modelo de diseño estructural que divide una aplicación en módulos o capas, donde cada una tiene responsabilidades específicas y se comunica solo con la capa inmediata. Este enfoque facilita el mantenimiento, la escalabilidad y la reutilización del código.

### Objetivos
- Separar responsabilidades (principio de separación de intereses).
- Facilitar el mantenimiento y la extensión del sistema.
- Aumentar la reutilización y la claridad del código.

---

## 2. Capas principales

| Capa | Responsabilidad | Ejemplo |
|------|------------------|----------|
| **Presentación (View)** | Interactúa con el usuario o sistema externo | Consola, interfaz gráfica o API |
| **Negocio (Service / Logic)** | Contiene la lógica de negocio, validaciones y reglas | Métodos que procesan datos |
| **Datos (DAO / Repository)** | Gestiona la conexión y operaciones con la base de datos o archivos | JDBC, CRUD, SQL, archivos |

En Java SE, estas tres capas son suficientes para construir aplicaciones organizadas.

---

## 3. Estructura de carpetas sugerida

```
src/
 ├── com.mycompany.app
 │    ├── domain/
 │    │    └── Estudiante.java
 │    ├── dao/
 │    │    └── EstudianteDAO.java
 │    ├── service/
 │    │    └── EstudianteService.java
 │    └── ui/
 │         └── AppMain.java
```

Cada carpeta representa una capa con una función específica dentro de la aplicación.

---

## 4. Ejemplo práctico: Sistema de gestión de estudiantes

### Capa Domain (Modelo)
```java
package com.mycompany.app.domain;

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

### Capa DAO (Datos)
```java
package com.mycompany.app.dao;

import com.mycompany.app.domain.Estudiante;
import java.util.ArrayList;
import java.util.List;

public class EstudianteDAO {
    private List<Estudiante> estudiantes = new ArrayList<>();

    public void guardar(Estudiante e) {
        estudiantes.add(e);
    }

    public List<Estudiante> listar() {
        return estudiantes;
    }

    public Estudiante buscarPorId(int id) {
        for (Estudiante e : estudiantes) {
            if (e.getId() == id) return e;
        }
        return null;
    }
}
```

---

### Capa Service (Lógica de Negocio)
```java
package com.mycompany.app.service;

import com.mycompany.app.dao.EstudianteDAO;
import com.mycompany.app.domain.Estudiante;
import java.util.List;

public class EstudianteService {
    private EstudianteDAO dao = new EstudianteDAO();

    public void registrarEstudiante(int id, String nombre, double promedio) {
        if (promedio < 0 || promedio > 5) {
            System.out.println("Promedio inválido");
            return;
        }
        Estudiante e = new Estudiante(id, nombre, promedio);
        dao.guardar(e);
    }

    public List<Estudiante> obtenerTodos() {
        return dao.listar();
    }

    public Estudiante buscarPorId(int id) {
        return dao.buscarPorId(id);
    }
}
```

---

### Capa UI (Presentación)
```java
package com.mycompany.app.ui;

import com.mycompany.app.service.EstudianteService;
import com.mycompany.app.domain.Estudiante;
import java.util.Scanner;

public class AppMain {
    public static void main(String[] args) {
        EstudianteService service = new EstudianteService();
        Scanner sc = new Scanner(System.in);

        int opcion;
        do {
            System.out.println("\n--- MENÚ ESTUDIANTES ---");
            System.out.println("1. Registrar");
            System.out.println("2. Listar");
            System.out.println("3. Buscar por ID");
            System.out.println("0. Salir");
            System.out.print("Opción: ");
            opcion = sc.nextInt();

            switch (opcion) {
                case 1 -> {
                    System.out.print("ID: ");
                    int id = sc.nextInt();
                    sc.nextLine();
                    System.out.print("Nombre: ");
                    String nombre = sc.nextLine();
                    System.out.print("Promedio: ");
                    double prom = sc.nextDouble();
                    service.registrarEstudiante(id, nombre, prom);
                }
                case 2 -> {
                    for (Estudiante e : service.obtenerTodos()) {
                        System.out.println(e);
                    }
                }
                case 3 -> {
                    System.out.print("ID a buscar: ");
                    int id = sc.nextInt();
                    Estudiante e = service.buscarPorId(id);
                    System.out.println(e != null ? e : "No encontrado");
                }
            }
        } while (opcion != 0);
    }
}
```

---

## 5. Flujo de comunicación entre capas

```
┌─────────────┐       ┌────────────────┐       ┌──────────────┐
│  AppMain    │──────▶│EstudianteService│──────▶│EstudianteDAO │
│ (Present.)  │       │(Lógica negocio) │       │(Datos)       │
└─────────────┘       └────────────────┘       └──────────────┘
         │                          │                       │
         │                          ▼                       ▼
         │                    Validaciones              Lista de objetos
         │
         ▼
     Usuario
```

---

## 6. Buenas prácticas

- Mantener una separación estricta entre las capas.
- La capa de presentación no debe acceder directamente al DAO.
- Definir paquetes con nombres coherentes: `domain`, `dao`, `service`, `ui`.
- Implementar manejo de excepciones en la capa de servicio.
- En caso de usar JDBC, mantener las consultas SQL dentro del DAO.
- Documentar el flujo de datos y las responsabilidades de cada clase.

---

## 7. Ejercicios sugeridos

1. Agregar una opción para eliminar estudiantes por ID.
2. Guardar los datos en un archivo CSV o base de datos MySQL usando JDBC.
3. Implementar el menú usando `JOptionPane` en lugar de consola.
4. Crear una excepción personalizada `EstudianteNoEncontradoException`.
5. Aplicar principios SOLID en la capa de servicio (ej. principio de responsabilidad única).

---

## 8. Conclusión

La arquitectura por capas permite organizar y aislar las responsabilidades de una aplicación Java, haciendo que el sistema sea más limpio, flexible y fácil de mantener. Esta estructura es la base para comprender arquitecturas más avanzadas como MVC, hexagonal o microservicios.