# 📘 Guía Completa: Patrón MVC, Patrones de Arquitectura y Patrones de Diseño

---

## 1. Patrón MVC (Model–View–Controller)

### Definición
MVC es un **patrón de arquitectura de software** que separa una aplicación en tres capas principales:
- **Modelo (Model)**: gestiona datos y lógica de negocio.
- **Vista (View)**: maneja la presentación (UI o consola).
- **Controlador (Controller)**: actúa como intermediario entre la vista y el modelo.

### Flujo
```
Usuario → Vista (input) → Controlador → Modelo → Controlador → Vista (output) → Usuario
```

### Beneficios
- ✅ Separación de responsabilidades.  
- ✅ Escalabilidad: migrar fácilmente de consola a web o escritorio.  
- ✅ Mantenibilidad: cambios en una capa no afectan a otras.  
- ✅ Reutilización: modelos y controladores se pueden usar con distintas vistas.

### Ejemplo práctico en Java

#### Modelo (`Usuario.java`)
```java
package model;

public class Usuario {
    private int id;
    private String nombre;
    private String email;

    public Usuario() {}
    public Usuario(int id, String nombre, String email) {
        this.id = id;
        this.nombre = nombre;
        this.email = email;
    }

    // Getters y setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

#### DAO (`UsuarioDAO.java`)
```java
package model;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class UsuarioDAO {
    private Connection conn;

    public UsuarioDAO(Connection conn) {
        this.conn = conn;
    }

    public void guardar(Usuario u) throws SQLException {
        String sql = "INSERT INTO usuario (nombre, email) VALUES (?, ?)";
        try (PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setString(1, u.getNombre());
            ps.setString(2, u.getEmail());
            ps.executeUpdate();
        }
    }

    public List<Usuario> listar() throws SQLException {
        List<Usuario> lista = new ArrayList<>();
        String sql = "SELECT * FROM usuario";
        try (Statement st = conn.createStatement();
             ResultSet rs = st.executeQuery(sql)) {
            while (rs.next()) {
                Usuario u = new Usuario(rs.getInt("id"), rs.getString("nombre"), rs.getString("email"));
                lista.add(u);
            }
        }
        return lista;
    }
}
```

#### Controlador (`UsuarioController.java`)
```java
package controller;

import model.Usuario;
import model.UsuarioDAO;
import java.sql.SQLException;
import java.util.List;

public class UsuarioController {
    private UsuarioDAO dao;

    public UsuarioController(UsuarioDAO dao) {
        this.dao = dao;
    }

    public void crearUsuario(String nombre, String email) throws SQLException {
        Usuario u = new Usuario();
        u.setNombre(nombre);
        u.setEmail(email);
        dao.guardar(u);
    }

    public List<Usuario> obtenerUsuarios() throws SQLException {
        return dao.listar();
    }
}
```

#### Vista (`Main.java`)
```java
package view;

import controller.UsuarioController;
import model.UsuarioDAO;
import model.Usuario;

import java.sql.*;
import java.util.List;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) throws Exception {
        Connection conn = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/academia", "root", "1234");

        UsuarioDAO dao = new UsuarioDAO(conn);
        UsuarioController controller = new UsuarioController(dao);

        Scanner sc = new Scanner(System.in);
        int opcion;

        do {
            System.out.println("=== MENU USUARIOS ===");
            System.out.println("1. Crear usuario");
            System.out.println("2. Listar usuarios");
            System.out.println("3. Salir");
            opcion = sc.nextInt();
            sc.nextLine();

            switch (opcion) {
                case 1:
                    System.out.print("Nombre: ");
                    String nombre = sc.nextLine();
                    System.out.print("Email: ");
                    String email = sc.nextLine();
                    controller.crearUsuario(nombre, email);
                    System.out.println("Usuario creado!");
                    break;

                case 2:
                    List<Usuario> lista = controller.obtenerUsuarios();
                    for (Usuario u : lista) {
                        System.out.println(u.getId() + " - " + u.getNombre() + " - " + u.getEmail());
                    }
                    break;

                case 3:
                    System.out.println("Saliendo...");
                    break;

                default:
                    System.out.println("Opción inválida.");
            }
        } while (opcion != 3);

        sc.close();
        conn.close();
    }
}
```

---

## 2. Diferencia entre Patrones de Arquitectura y Patrones de Diseño

### Patrones de Arquitectura
- **Nivel**: Macro (cómo organizar todo un sistema).
- **Qué resuelven**: estructura y organización general de la aplicación.
- **Ejemplos**:  
  - MVC (Model–View–Controller)  
  - Capas (Layered Architecture)  
  - Hexagonal / Puertos y Adaptadores  
  - Clean Architecture  
  - Microservicios  

👉 Son como el **plano general de una ciudad**.

### Patrones de Diseño (GoF)
- **Nivel**: Micro (cómo organizar clases y objetos dentro de esa arquitectura).
- **Qué resuelven**: problemas recurrentes a nivel de código.
- **Ejemplos**:  
  - **Creacionales** → Singleton, Factory, Builder  
  - **Estructurales** → Adapter, Decorator, Composite  
  - **Comportamiento** → Strategy, Observer, Command  

👉 Son como las **reglas de construcción de las casas** en la ciudad.

### Relación entre ambos
- El **patrón de arquitectura** define el esqueleto del sistema.  
- Los **patrones de diseño** resuelven problemas puntuales dentro de ese esqueleto.  

#### Ejemplo integrador
Un **sistema bancario** puede usar:  
- **Arquitectura en capas**.  
- Dentro de la capa de negocio, un **Strategy** para calcular intereses.  
- En la capa de datos, un **Singleton** para la conexión DB.  

### Comparación

| Aspecto                  | Patrones de Arquitectura               | Patrones de Diseño (GoF)            |
|---------------------------|----------------------------------------|--------------------------------------|
| **Nivel**                | Macro (sistema completo)               | Micro (clases y objetos)             |
| **Problemas que resuelven** | Organización global del software        | Problemas puntuales de diseño OO     |
| **Ejemplo**              | MVC, Capas, Microservicios             | Singleton, Observer, Strategy        |
| **Escalabilidad**        | Alta (impacta a todo el sistema)       | Local (impacta a módulos/clases)     |

---

## 3. Conclusión
- **Patrones de Arquitectura** → cómo se organiza **todo el sistema**.  
- **Patrones de Diseño** → cómo se organizan **los bloques internos** (clases/objetos).  
- **Ambos se complementan**: MVC puede usar patrones GoF como Singleton o Strategy dentro de su implementación.
