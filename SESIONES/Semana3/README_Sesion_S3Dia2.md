
# CRUD con JDBC (Guía clara y al grano)

Esta guía simplifica **JDBC** a lo esencial para crear un CRUD (Create, Read, Update, Delete) contra una base de datos relacional (usaremos **MySQL** de ejemplo). Incluye:
- Script SQL para crear la tabla.
- Clase `ConexionDB` (Factory) para obtener conexiones.
- DAO `ProductoDAO` con métodos `crear`, `buscarPorId`, `listar`, `actualizar`, `eliminar`.
- Consejos de transacciones, `PreparedStatement`, manejo de recursos y errores.
- Mini `Main` para probar todo.

> Puedes adaptar los nombres de paquete y credenciales a tu entorno.

---

## 1) Requisitos y esquema SQL

**Dependencia (Maven):**
```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-j</artifactId>
  <version>8.4.0</version>
</dependency>
```

**Base y tabla de ejemplo:**
```sql
CREATE DATABASE IF NOT EXISTS tienda;
USE tienda;

CREATE TABLE IF NOT EXISTS producto (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(120) NOT NULL,
  precio DECIMAL(10,2) NOT NULL,
  stock INT NOT NULL DEFAULT 0,
  creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 2) Conexión JDBC (Factory)

Usaremos **try-with-resources** siempre que sea posible y variables de entorno para no hardcodear credenciales.

```java
// src/main/java/com/ejemplo/db/ConexionDB.java
package com.ejemplo.db;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConexionDB {
    // Usa variables de entorno o propiedades (recomendado)
    private static final String URL  = System.getenv().getOrDefault("DB_URL",  "jdbc:mysql://localhost:3306/tienda");
    private static final String USER = System.getenv().getOrDefault("DB_USER", "root");
    private static final String PASS = System.getenv().getOrDefault("DB_PASS", "root");

    public static Connection getConexion() throws SQLException {
        // El driver de MySQL 8 se carga automáticamente al tener la dependencia
        return DriverManager.getConnection(URL, USER, PASS);
    }
}
```

> Si no usas variables de entorno, cambia los valores por los tuyos. En Linux/macOS:
> ```bash
> export DB_URL="jdbc:mysql://localhost:3306/tienda"
> export DB_USER="root"
> export DB_PASS="tu_password"
> ```

---

## 3) Entidad `Producto`

```java
// src/main/java/com/ejemplo/domain/Producto.java
package com.ejemplo.domain;

public class Producto {
    private Integer id;
    private String  nombre;
    private double  precio;
    private int     stock;

    public Producto() {}
    public Producto(Integer id, String nombre, double precio, int stock) {
        this.id = id; this.nombre = nombre; this.precio = precio; this.stock = stock;
    }
    public Producto(String nombre, double precio, int stock) {
        this(null, nombre, precio, stock);
    }

    // Getters/Setters
    public Integer getId() { return id; }
    public void setId(Integer id) { this.id = id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public double getPrecio() { return precio; }
    public void setPrecio(double precio) { this.precio = precio; }
    public int getStock() { return stock; }
    public void setStock(int stock) { this.stock = stock; }

    @Override public String toString() {
        return "Producto{id=%d, nombre='%s', precio=%.2f, stock=%d}".formatted(id, nombre, precio, stock);
    }
}
```

---

## 4) CRUD con `PreparedStatement` (DAO)

```java
// src/main/java/com/ejemplo/dao/ProductoDAO.java
package com.ejemplo.dao;

import com.ejemplo.db.ConexionDB;
import com.ejemplo.domain.Producto;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class ProductoDAO {

    // CREATE: insertar y devolver el ID generado
    public int crear(Producto p) throws SQLException {
        String sql = "INSERT INTO producto (nombre, precio, stock) VALUES (?, ?, ?)";
        try (Connection conn = ConexionDB.getConexion();
             PreparedStatement ps = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {

            ps.setString(1, p.getNombre());
            ps.setDouble(2, p.getPrecio());
            ps.setInt(3, p.getStock());

            int filas = ps.executeUpdate();
            if (filas == 0) throw new SQLException("No se insertó ningún registro");

            try (ResultSet rs = ps.getGeneratedKeys()) {
                if (rs.next()) {
                    int id = rs.getInt(1);
                    p.setId(id);
                    return id;
                } else {
                    throw new SQLException("No se obtuvo ID generado");
                }
            }
        }
    }

    // READ (uno): buscar por id
    public Producto buscarPorId(int id) throws SQLException {
        String sql = "SELECT id, nombre, precio, stock FROM producto WHERE id = ?";
        try (Connection conn = ConexionDB.getConexion();
             PreparedStatement ps = conn.prepareStatement(sql)) {

            ps.setInt(1, id);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    return mapRow(rs);
                }
                return null; // no encontrado
            }
        }
    }

    // READ (lista): traer todos (o podrías paginar)
    public List<Producto> listar() throws SQLException {
        String sql = "SELECT id, nombre, precio, stock FROM producto ORDER BY id DESC";
        List<Producto> out = new ArrayList<>();
        try (Connection conn = ConexionDB.getConexion();
             PreparedStatement ps = conn.prepareStatement(sql);
             ResultSet rs = ps.executeQuery()) {

            while (rs.next()) out.add(mapRow(rs));
        }
        return out;
    }

    // UPDATE: actualizar campos por id
    public boolean actualizar(Producto p) throws SQLException {
        if (p.getId() == null) throw new IllegalArgumentException("El producto debe tener id");
        String sql = "UPDATE producto SET nombre = ?, precio = ?, stock = ? WHERE id = ?";
        try (Connection conn = ConexionDB.getConexion();
             PreparedStatement ps = conn.prepareStatement(sql)) {

            ps.setString(1, p.getNombre());
            ps.setDouble(2, p.getPrecio());
            ps.setInt(3, p.getStock());
            ps.setInt(4, p.getId());

            return ps.executeUpdate() > 0;
        }
    }

    // DELETE: eliminar por id
    public boolean eliminar(int id) throws SQLException {
        String sql = "DELETE FROM producto WHERE id = ?";
        try (Connection conn = ConexionDB.getConexion();
             PreparedStatement ps = conn.prepareStatement(sql)) {

            ps.setInt(1, id);
            return ps.executeUpdate() > 0;
        }
    }

    // Utilidad: mapear una fila a objeto
    private Producto mapRow(ResultSet rs) throws SQLException {
        return new Producto(
                rs.getInt("id"),
                rs.getString("nombre"),
                rs.getDouble("precio"),
                rs.getInt("stock")
        );
    }
}
```

### Puntos clave del DAO
- **`PreparedStatement`** evita SQL Injection y permite pasar parámetros tipados.
- **`Statement.RETURN_GENERATED_KEYS`** para recuperar el **ID autoincrement** al crear.
- **`try-with-resources`** cierra `Connection`, `PreparedStatement` y `ResultSet` automáticamente.
- Métodos devuelven **boolean** o el objeto encontrado para comprobar resultados.

---

## 5) Transacciones (cuando combinas varios pasos)

Usa transacciones si hay **operaciones dependientes** (todo o nada).

```java
// Ejemplo: crear un producto y actualizar su stock como parte de la misma operación
public void crearYSumarStock(Producto p, int extra) throws SQLException {
    String insertSql = "INSERT INTO producto (nombre, precio, stock) VALUES (?, ?, ?)";
    String updateSql = "UPDATE producto SET stock = stock + ? WHERE id = ?";

    try (Connection conn = ConexionDB.getConexion()) {
        conn.setAutoCommit(false); // inicia transacción
        try (PreparedStatement psInsert = conn.prepareStatement(insertSql, Statement.RETURN_GENERATED_KEYS);
             PreparedStatement psUpdate = conn.prepareStatement(updateSql)) {

            // Insert
            psInsert.setString(1, p.getNombre());
            psInsert.setDouble(2, p.getPrecio());
            psInsert.setInt(3, p.getStock());
            psInsert.executeUpdate();

            try (ResultSet rs = psInsert.getGeneratedKeys()) {
                if (!rs.next()) throw new SQLException("Sin ID generado");
                int idGenerado = rs.getInt(1);
                p.setId(idGenerado);

                // Update dependiente
                psUpdate.setInt(1, extra);
                psUpdate.setInt(2, idGenerado);
                psUpdate.executeUpdate();
            }

            conn.commit(); // todo OK
        } catch (SQLException e) {
            conn.rollback(); // deshacer todo
            throw e;
        } finally {
            conn.setAutoCommit(true);
        }
    }
}
```

**Reglas de oro en transacciones:**
- Desactiva `autoCommit`, haz tus operaciones, **`commit()`** o **`rollback()`** ante error.
- Mantén transacciones **cortas** para no bloquear la BD.
- Maneja excepciones y vuelve a activar `autoCommit(true)` en el `finally`.

---

## 6) Mini `Main` para probar CRUD

```java
// src/main/java/com/ejemplo/Main.java
package com.ejemplo;

import com.ejemplo.dao.ProductoDAO;
import com.ejemplo.domain.Producto;

import java.util.List;

public class Main {
    public static void main(String[] args) {
        ProductoDAO dao = new ProductoDAO();
        try {
            // CREATE
            Producto p = new Producto("Teclado mecánico", 199.99, 10);
            int id = dao.crear(p);
            System.out.println("Creado con id = " + id);

            // READ (uno)
            Producto buscado = dao.buscarPorId(id);
            System.out.println("Buscado: " + buscado);

            // READ (lista)
            List<Producto> productos = dao.listar();
            System.out.println("Listado: " + productos);

            // UPDATE
            buscado.setPrecio(149.90);
            buscado.setStock(20);
            boolean okUpdate = dao.actualizar(buscado);
            System.out.println("Actualizado? " + okUpdate);

            // DELETE
            boolean okDelete = dao.eliminar(id);
            System.out.println("Eliminado? " + okDelete);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

---

## 7) Buenas prácticas y errores comunes

**Buenas prácticas**
- **Capas**: separa `domain` (POJOs), `dao` (SQL), `db` (conexión), `service` (reglas de negocio) y `ui`.
- Valida entradas antes de llegar al DAO (por ej., `precio >= 0`, `nombre` no vacío).
- Usa **paginación** en `listar` si hay muchos datos (`LIMIT ?, ?`).  
- Maneja **zonas horarias** y formatos numéricos según el locale.
- **Logs** con contexto (parámetros, tiempos). Evita loggear contraseñas.

**Errores típicos y cómo evitarlos**
- **`Communications link failure`**: revisa host/puerto, firewall, que MySQL está levantado.
- **`Access denied for user`**: credenciales incorrectas o el usuario no tiene permisos en la BD.
- **Tabla/columna no existe**: corre el **script SQL** de la sección 1 y sincroniza nombres.
- **Tipos incompatibles**: al mapear `ResultSet`, usa getters correctos (`getInt`, `getDouble`, etc.).
- **Fugas de recursos**: SOLUCIÓN: `try-with-resources` y no guardes `Connection` globales.
- **SQL Injection**: usa **`PreparedStatement`** siempre y nunca concatenes strings con datos de usuario.
- **AutoCommit/Transacciones**: si desactivas `autoCommit`, **recuerda** `commit()` o `rollback()` y volver a `true`.

---

## 8) Extensiones útiles

- **Búsquedas filtradas**: `listarPorNombre(String filtro)` con `WHERE nombre LIKE ?` (`%filtro%`).
- **Batch**: inserciones masivas con `addBatch()` y `executeBatch()`.
- **DAO genérico**: una interfaz común (`crear`, `actualizar`, `eliminar`, `buscarPorId`, `listar`) y distintas implementaciones.
- **Pool de conexiones**: HikariCP para producción (mejor rendimiento y menos latencia).

---

### Resumen mental (lo mínimo que debes recordar)
1. **Conexión** (`DriverManager.getConnection`).
2. **`PreparedStatement`** con parámetros (`?`), setea tipos correctos.
3. **`executeUpdate`** para `INSERT/UPDATE/DELETE`, **`executeQuery`** para `SELECT`.
4. **Lee resultados** con `ResultSet` y mapea a objetos.
5. **Cierra recursos** o usa `try-with-resources`.
6. **Transacciones** cuando sean varias operaciones dependientes.

¡Listo! Con esto puedes construir y explicar CRUD con JDBC con claridad profesional.