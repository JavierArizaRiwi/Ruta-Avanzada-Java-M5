# JDBC en Java — Guía completa con paso a paso, con y sin patrones de diseño

> Todo lo esencial para configurar JDBC rápidamente, y dos enfoques de implementación: **directo (sin patrones)** y **modular/profesional (con patrones)**.

---

## 0) Requisitos previos

- **Java 17+** (o al menos 11).
- Un motor de base de datos relacional (ej.: **MySQL 8** o **PostgreSQL 14+**).
- **Maven** o **Gradle** para gestionar dependencias.
- Cliente SQL opcional: **MySQL Workbench** o **pgAdmin**.

---

## 1) Paso a paso (simple) de configuración

### 1.1 Instalar y crear base de datos
1. Instala el servidor (MySQL o PostgreSQL).
2. Crea una base de datos y un usuario con permisos:
   - **MySQL**
     ```sql
     CREATE DATABASE demo_jdbc CHARACTER SET utf8mb4;
     CREATE USER 'app_user'@'%' IDENTIFIED BY 'app_pwd';
     GRANT ALL PRIVILEGES ON demo_jdbc.* TO 'app_user'@'%';
     FLUSH PRIVILEGES;
     ```
   - **PostgreSQL**
     ```sql
     CREATE DATABASE demo_jdbc;
     CREATE USER app_user WITH PASSWORD 'app_pwd';
     GRANT ALL PRIVILEGES ON DATABASE demo_jdbc TO app_user;
     ```

### 1.2 Agregar dependencia del driver (Maven)
- **MySQL**
  ```xml
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.33</version>
  </dependency>
  ```
- **PostgreSQL**
  ```xml
  <dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.3</version>
  </dependency>
  ```

### 1.3 Definir variables de conexión (application.properties)
Crea `src/main/resources/application.properties`:
```properties
db.vendor=mysql          # mysql | postgres
db.host=localhost
db.port=3306             # 5432 para Postgres
db.name=demo_jdbc
db.user=app_user
db.password=app_pwd
db.useSSL=false          # true si tu servidor exige SSL

# Opcional: pool (si usas HikariCP)
pool.enabled=false
pool.maxPoolSize=5
```

### 1.4 URL JDBC típica
- **MySQL**: `jdbc:mysql://{host}:{port}/{db}?useSSL=false&serverTimezone=UTC`
- **PostgreSQL**: `jdbc:postgresql://{host}:{port}/{db}`

### 1.5 Probar conexión mínima
1. Crea una tabla sencilla:
   ```sql
   CREATE TABLE usuarios (
     id INT PRIMARY KEY AUTO_INCREMENT,           -- SERIAL en Postgres
     nombre VARCHAR(100) NOT NULL,
     email VARCHAR(120) UNIQUE NOT NULL,
     creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   -- PostgreSQL: id SERIAL PRIMARY KEY
   ```
2. Ejecuta un programa que haga `SELECT 1` para validar que todo responde (ver ejemplos abajo).

> **Listo.** Con esto ya puedes conectar y consultar.

---

## 2) JDBC básico — **SIN patrones de diseño** (enfoque directo)

> Útil para demos, pruebas rápidas o scripts internos. No recomendado para proyectos grandes.

```java
// src/main/java/demo/basico/MainSinPatrones.java
package demo.basico;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class MainSinPatrones {
    public static void main(String[] args) {
        // Definición de parámetros de conexión (esto normalmente debería ir en un archivo de configuración)
        String vendor = "mysql"; // o "postgres"
        String host = "localhost";
        String port = vendor.equals("postgres") ? "5432" : "3306";
        String db   = "demo_jdbc";
        String user = "app_user";
        String pass = "app_pwd";

        // Construcción de la URL JDBC según el motor de base de datos
        String url = vendor.equals("postgres")
                ? String.format("jdbc:postgresql://%s:%s/%s", host, port, db)
                : String.format("jdbc:mysql://%s:%s/%s?useSSL=false&serverTimezone=UTC", host, port, db);

        // 1) Conexión
        try (Connection conn = DriverManager.getConnection(url, user, pass)) {
            System.out.println("Conexión OK");

            // 2) INSERT con PreparedStatement (evita inyección SQL)
            // Usar PreparedStatement es fundamental para seguridad y performance
            String insertSql = "INSERT INTO usuarios (nombre, email) VALUES (?, ?)";
            try (PreparedStatement ps = conn.prepareStatement(insertSql)) {
                ps.setString(1, "Ana");
                ps.setString(2, "ana@mail.com");
                int filas = ps.executeUpdate();
                System.out.println("Filas insertadas: " + filas);
            }

            // 3) SELECT y mapeo manual
            // Aquí se consulta la tabla y se recorre el ResultSet para mostrar los datos
            String selectSql = "SELECT id, nombre, email, creado_en FROM usuarios";
            List<String> filas = new ArrayList<>();
            try (Statement st = conn.createStatement();
                 ResultSet rs = st.executeQuery(selectSql)) {
                while (rs.next()) {
                    filas.add(
                        rs.getInt("id") + " | " +
                        rs.getString("nombre") + " | " +
                        rs.getString("email") + " | " +
                        rs.getTimestamp("creado_en")
                    );
                }
            }
            filas.forEach(System.out::println);

            // 4) Transacción manual (ejemplo simple)
            // Las transacciones permiten agrupar varias operaciones y asegurar que todas se ejecuten correctamente
            conn.setAutoCommit(false);
            try (PreparedStatement ps = conn.prepareStatement("UPDATE usuarios SET nombre=? WHERE email=?")) {
                ps.setString(1, "Ana Actualizada");
                ps.setString(2, "ana@mail.com");
                ps.executeUpdate();
                conn.commit();
            } catch (SQLException ex) {
                conn.rollback(); // Si ocurre un error, se revierte la transacción
                throw ex;
            }

        } catch (SQLException e) {
            // Manejo básico de errores: en producción, deberías loggear y mostrar mensajes claros
            e.printStackTrace();
        }
    }
}
```

**Pros:** directo, pocos archivos.  
**Contras:** SQL “pegado” al código, duplicación, difícil de testear/escale, manejo de transacciones y recursos disperso.

---

## 3) JDBC profesional — **CON patrones de diseño**

### 3.1 Objetivo del diseño
- **Factory Method**: centralizar la creación de `Connection` según el vendor (MySQL/Postgres) y configuración.
- **Strategy**: desacoplar el **mapeo** de `ResultSet` a entidades (`RowMapper<T>`), permitiendo reutilizar consultas y pruebas.
- **Template Method**: encapsular el flujo repetitivo (abrir conexión → preparar sentencia → bind → ejecutar → mapear → cerrar) en una clase base (`JdbcTemplateLight`).
- **DAO/Repository**: interfaces limpias (`UsuarioRepository`) con implementaciones JDBC; el servicio no conoce SQL.
- **(Opcional) Facade/Service**: caso de uso de negocio que orquesta varias llamadas a repositorios bajo una transacción.

> El resultado es parecido a **Spring JDBC** pero hecho “a mano” para entender los principios.

### 3.2 Código con comentarios explicativos

```java
// src/main/java/demo/patrones/config/AppConfig.java
package demo.patrones.config;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

// Esta clase carga la configuración desde application.properties
public class AppConfig {
    private final Properties props = new Properties();

    public AppConfig() {
        try (InputStream in = getClass().getResourceAsStream("/application.properties")) {
            if (in == null) throw new IllegalStateException("No se encontró application.properties");
            props.load(in);
        } catch (IOException e) {
            throw new RuntimeException("Error cargando configuración", e);
        }
    }

    // Permite obtener cualquier propiedad por clave
    public String get(String key) { return props.getProperty(key); }
}
```

```java
// src/main/java/demo/patrones/db/ConnectionFactory.java
package demo.patrones.db;

import demo.patrones.config.AppConfig;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

// Factory Method: centraliza la creación de conexiones JDBC
public class ConnectionFactory {
    private final AppConfig cfg;
    public ConnectionFactory(AppConfig cfg) { this.cfg = cfg; }

    // Método que abre una conexión según el motor configurado
    public Connection open() throws SQLException {
        String vendor = cfg.get("db.vendor");
        String host = cfg.get("db.host");
        String port = cfg.get("db.port");
        String name = cfg.get("db.name");
        String user = cfg.get("db.user");
        String pass = cfg.get("db.password");

        String url;
        if ("postgres".equalsIgnoreCase(vendor)) {
            url = String.format("jdbc:postgresql://%s:%s/%s", host, port, name);
        } else {
            String useSSL = cfg.get("db.useSSL");
            url = String.format("jdbc:mysql://%s:%s/%s?useSSL=%s&serverTimezone=UTC", host, port, name, useSSL);
        }
        return DriverManager.getConnection(url, user, pass);
    }
}
```

```java
// src/main/java/demo/patrones/jdbc/RowMapper.java
package demo.patrones.jdbc;

import java.sql.ResultSet;
import java.sql.SQLException;

// Strategy: define cómo convertir una fila de ResultSet en un objeto Java
public interface RowMapper<T> {
    T map(ResultSet rs) throws SQLException;
}
```

```java
// src/main/java/demo/patrones/jdbc/JdbcTemplateLight.java
package demo.patrones.jdbc;

import demo.patrones.db.ConnectionFactory;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.function.Consumer;

// Template Method: encapsula el flujo común para ejecutar SQL
public class JdbcTemplateLight {
    private final ConnectionFactory factory;
    public JdbcTemplateLight(ConnectionFactory factory) { this.factory = factory; }

    // Ejecuta una consulta y mapea los resultados usando RowMapper
    public <T> List<T> query(String sql, Consumer<PreparedStatement> binder, RowMapper<T> mapper) throws SQLException {
        try (Connection c = factory.open();
             PreparedStatement ps = c.prepareStatement(sql)) {
            if (binder != null) binder.accept(ps); // Permite parametrizar la consulta
            try (ResultSet rs = ps.executeQuery()) {
                List<T> out = new ArrayList<>();
                while (rs.next()) out.add(mapper.map(rs)); // Mapea cada fila
                return out;
            }
        }
    }

    // Ejecuta una actualización (INSERT, UPDATE, DELETE)
    public int update(String sql, Consumer<PreparedStatement> binder) throws SQLException {
        try (Connection c = factory.open();
             PreparedStatement ps = c.prepareStatement(sql)) {
            if (binder != null) binder.accept(ps);
            return ps.executeUpdate();
        }
    }

    // Ejecuta varias operaciones en una transacción
    public <T> T txExecute(SqlTxCallback<T> cb) throws SQLException {
        try (Connection c = factory.open()) {
            boolean prev = c.getAutoCommit();
            c.setAutoCommit(false);
            try {
                T result = cb.doInTx(c);
                c.commit();
                return result;
            } catch (SQLException ex) {
                c.rollback();
                throw ex;
            } finally {
                c.setAutoCommit(prev);
            }
        }
    }

    // Callback funcional para transacciones
    @FunctionalInterface
    public interface SqlTxCallback<T> {
        T doInTx(Connection conn) throws SQLException;
    }
}
```

```java
// src/main/java/demo/patrones/domain/Usuario.java
package demo.patrones.domain;

import java.time.Instant;

// POJO simple para representar un usuario
public class Usuario {
    private Integer id;
    private String nombre;
    private String email;
    private Instant creadoEn;

    // Constructor y métodos de acceso
    public Usuario(Integer id, String nombre, String email, Instant creadoEn) {
        this.id = id; this.nombre = nombre; this.email = email; this.creadoEn = creadoEn;
    }
    public Integer getId() { return id; }
    public String getNombre() { return nombre; }
    public String getEmail() { return email; }
    public Instant getCreadoEn() { return creadoEn; }

    @Override public String toString() {
        return "Usuario{id=%d, nombre='%s', email='%s'}".formatted(id, nombre, email);
    }
}
```

```java
// src/main/java/demo/patrones/repo/UsuarioRepository.java
package demo.patrones.repo;

import demo.patrones.domain.Usuario;
import java.util.List;
import java.util.Optional;

// DAO/Repository: define las operaciones de acceso a datos para Usuario
public interface UsuarioRepository {
    int crear(String nombre, String email);
    int actualizarNombrePorEmail(String nombre, String email);
    Optional<Usuario> buscarPorEmail(String email);
    List<Usuario> listar();
}
```

```java
// src/main/java/demo/patrones/repo/jdbc/UsuarioJdbcRepository.java
package demo.patrones.repo.jdbc;

import demo.patrones.domain.Usuario;
import demo.patrones.jdbc.JdbcTemplateLight;
import demo.patrones.jdbc.RowMapper;
import demo.patrones.repo.UsuarioRepository;

import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.time.ZoneId;
import java.util.List;
import java.util.Optional;

// Implementación JDBC del repositorio de usuarios
public class UsuarioJdbcRepository implements UsuarioRepository {
    private final JdbcTemplateLight jdbc;

    public UsuarioJdbcRepository(JdbcTemplateLight jdbc) { this.jdbc = jdbc; }

    // Mapeador de filas: convierte ResultSet en Usuario
    private static final RowMapper<Usuario> MAPPER = rs -> new Usuario(
            rs.getInt("id"),
            rs.getString("nombre"),
            rs.getString("email"),
            rs.getTimestamp("creado_en").toInstant()
    );

    @Override
    public int crear(String nombre, String email) {
        String sql = "INSERT INTO usuarios(nombre, email) VALUES(?, ?)";
        try {
            return jdbc.update(sql, ps -> {
                try {
                    ps.setString(1, nombre);
                    ps.setString(2, email);
                } catch (SQLException e) { throw new RuntimeException(e); }
            });
        } catch (SQLException e) { throw new RuntimeException(e); }
    }

    @Override
    public int actualizarNombrePorEmail(String nombre, String email) {
        String sql = "UPDATE usuarios SET nombre=? WHERE email=?";
        try {
            return jdbc.update(sql, ps -> {
                try {
                    ps.setString(1, nombre);
                    ps.setString(2, email);
                } catch (SQLException e) { throw new RuntimeException(e); }
            });
        } catch (SQLException e) { throw new RuntimeException(e); }
    }

    @Override
    public Optional<Usuario> buscarPorEmail(String email) {
        String sql = "SELECT id, nombre, email, creado_en FROM usuarios WHERE email=?";
        try {
            List<Usuario> list = jdbc.query(sql, ps -> {
                try { ps.setString(1, email); } catch (SQLException e) { throw new RuntimeException(e); }
            }, MAPPER);
            return list.stream().findFirst();
        } catch (SQLException e) { throw new RuntimeException(e); }
    }

    @Override
    public List<Usuario> listar() {
        String sql = "SELECT id, nombre, email, creado_en FROM usuarios";
        try {
            return jdbc.query(sql, null, MAPPER);
        } catch (SQLException e) { throw new RuntimeException(e); }
    }
}
```

```java
// src/main/java/demo/patrones/service/UsuarioService.java
package demo.patrones.service;

import demo.patrones.domain.Usuario;
import demo.patrones.repo.UsuarioRepository;

import java.util.List;
import java.util.Optional;

// Facade/Service: orquesta casos de uso y oculta detalles de acceso a datos
public class UsuarioService {
    private final UsuarioRepository repo;
    public UsuarioService(UsuarioRepository repo) { this.repo = repo; }

    public void registrar(String nombre, String email) {
        repo.crear(nombre, email);
    }

    public void renombrar(String nuevoNombre, String email) {
        repo.actualizarNombrePorEmail(nuevoNombre, email);
    }

    public Optional<Usuario> buscarPorEmail(String email) { return repo.buscarPorEmail(email); }
    public List<Usuario> listar() { return repo.listar(); }
}
```

```java
// src/main/java/demo/patrones/MainConPatrones.java
package demo.patrones;

import demo.patrones.config.AppConfig;
import demo.patrones.db.ConnectionFactory;
import demo.patrones.jdbc.JdbcTemplateLight;
import demo.patrones.repo.UsuarioRepository;
import demo.patrones.repo.jdbc.UsuarioJdbcRepository;
import demo.patrones.service.UsuarioService;

// Punto de entrada: aquí se conectan todas las piezas y se ejecutan los casos de uso
public class MainConPatrones {
    public static void main(String[] args) {
        // Wiring manual (podrías usar Spring más adelante)
        AppConfig cfg = new AppConfig(); // Carga configuración
        ConnectionFactory factory = new ConnectionFactory(cfg); // Crea conexiones
        JdbcTemplateLight jdbc = new JdbcTemplateLight(factory); // Encapsula el flujo JDBC
        UsuarioRepository repo = new UsuarioJdbcRepository(jdbc); // Implementa acceso a datos
        UsuarioService service = new UsuarioService(repo); // Orquesta casos de uso

        // Uso: registrar, renombrar y listar usuarios
        service.registrar("Ana", "ana@mail.com");
        service.renombrar("Ana Actualizada", "ana@mail.com");
        service.listar().forEach(System.out::println);
    }
}
```

**Ventajas (con patrones)**  
- Código **modular y testeable** (puedes _mockear_ `UsuarioRepository`).  
- **Reutilización** del flujo JDBC (Template) y del mapeo (Strategy).  
- Creación de conexiones centralizada (Factory Method), lista para agregar pool.  
- Fácil evolución hacia **Spring JDBC** o **JPA** sin reescribir tu dominio.

**Trade-offs**  
- Más clases/archivos.  
- Pequeña infraestructura propia (plantillas, mapeadores) que debes mantener (o sustituir por Spring cuando estés listo).

---

## 4) Pool de conexiones (opcional, recomendado)

Si el tráfico crece, usa un pool como **HikariCP**.

**Maven:**
```xml
<dependency>
  <groupId>com.zaxxer</groupId>
  <artifactId>HikariCP</artifactId>
  <version>5.1.0</version>
</dependency>
```

**Factory alternativa con Hikari:**
```java
// src/main/java/demo/patrones/db/HikariConnectionFactory.java
package demo.patrones.db;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import demo.patrones.config.AppConfig;

import java.sql.Connection;
import java.sql.SQLException;
import javax.sql.DataSource;

// Esta clase extiende ConnectionFactory y usa HikariCP para gestionar el pool de conexiones
public class HikariConnectionFactory extends ConnectionFactory {
    private final DataSource ds;

    public HikariConnectionFactory(AppConfig cfg) {
        super(cfg);
        String vendor = cfg.get("db.vendor");
        String host = cfg.get("db.host");
        String port = cfg.get("db.port");
        String name = cfg.get("db.name");
        String user = cfg.get("db.user");
        String pass = cfg.get("db.password");

        String jdbcUrl = "postgres".equalsIgnoreCase(vendor)
            ? String.format("jdbc:postgresql://%s:%s/%s", host, port, name)
            : String.format("jdbc:mysql://%s:%s/%s?useSSL=%s&serverTimezone=UTC",
                host, port, name, cfg.get("db.useSSL"));

        HikariConfig hc = new HikariConfig();
        hc.setJdbcUrl(jdbcUrl);
        hc.setUsername(user);
        hc.setPassword(pass);
        hc.setMaximumPoolSize(Integer.parseInt(cfg.get("pool.maxPoolSize")));
        this.ds = new HikariDataSource(hc);
    }

    @Override
    public Connection open() throws SQLException { return ds.getConnection(); }
}
```

> Con esto, cambiar a pool es **transparente** para `JdbcTemplateLight` y tus repositorios.

---

## 5) Checklist de buenas prácticas

- **PreparedStatement** siempre que haya parámetros (seguridad y performance).
- **try-with-resources** para cerrar `Connection/Statement/ResultSet` automáticamente.
- **Transacciones** con `setAutoCommit(false)` y `commit/rollback` apropiados.
- No expongas **credenciales** en el repositorio (usa variables de entorno/gestor de secretos).
- Loggea SQL de negocio crítico y errores con contexto (valores, tamaños de lote, etc.).
- Evita **select * **; especifica columnas. Agrega **índices** según consultas frecuentes.
- Si el proyecto crece, evalúa **Spring JDBC** o **JPA** para ganar productividad.

---

## 6) Ejercicios sugeridos

1. Implementa `ProductoRepository` con el mismo enfoque de patrones.
2. Añade paginación y ordenamiento; compara MySQL vs Postgres (p. ej., `LIMIT/OFFSET`).
3. Implementa un **RowMapper genérico** con reflexión para POJOs simples.
4. Integra **transacciones** que afecten a 2–3 tablas con `txExecute`.
5. Cambia de `ConnectionFactory` a `HikariConnectionFactory` sin tocar repositorios.

---

## 7) Estructura sugerida del proyecto

```
src/
 └─ main/
    ├─ java/
    │   └─ demo/
    │       ├─ basico/
    │       │   └─ MainSinPatrones.java
    │       ├─ patrones/
    │       │   ├─ MainConPatrones.java
    │       │   ├─ config/AppConfig.java
    │       │   ├─ db/ConnectionFactory.java
    │       │   ├─ db/HikariConnectionFactory.java    (opcional)
    │       │   ├─ jdbc/JdbcTemplateLight.java
    │       │   ├─ jdbc/RowMapper.java
    │       │   ├─ domain/Usuario.java
    │       │   ├─ repo/UsuarioRepository.java
    │       │   └─ repo/jdbc/UsuarioJdbcRepository.java
    │       └─ service/UsuarioService.java
    └─ resources/
        └─ application.properties
```

---

## 8) Conclusión

- **Sin patrones**: perfecto para pruebas rápidas, pero difícil de escalar.  
- **Con patrones**: obtienes modularidad, testabilidad y un camino natural hacia **Spring**.  
- Los patrones **Factory Method**, **Strategy**, **Template Method**, **DAO/Repository** y **Service/Facade** se complementan para que la capa de datos sea **limpia, segura y mantenible**.

---

**Recuerda comentar tu código explicando el propósito de cada clase, método y bloque importante. Esto facilita el aprendizaje y el mantenimiento