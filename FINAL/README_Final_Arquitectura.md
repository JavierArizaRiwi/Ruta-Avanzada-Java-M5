
# Arquitectura por Capas con Contratos (Interfaces) — `com.mycompany.academic`

> **Objetivo:** Dejar tu proyecto organizado por capas (domain, repository, service, ui, infra, exceptions), con **contratos** basados en **interfaces** para lograr **bajo acoplamiento**, **alta cohesión**, **testabilidad** y **mantenibilidad**.  
> **Tecnologías:** Java SE + JDBC (sin frameworks), Swing/Consola para UI.  
> **Patrones y principios:** **Repository**, **Service**, **DTO/Value Objects**, **Excepciones personalizadas**, **Inversión de Dependencias**, **SOLID** (especialmente **ISP** y **DIP**).

---

## Índice
1. [Estructura de paquetes](#estructura-de-paquetes)
2. [Por qué usar interfaces (contratos)](#por-qué-usar-interfaces-contratos)
3. [Descripción de cada capa y clases](#descripción-de-cada-capa-y-clases)
4. [Modelo de dominio: POJOs y Value Objects](#modelo-de-dominio-pojos-y-value-objects)
5. [Excepciones personalizadas](#excepciones-personalizadas)
6. [Repository (DAO) — Interfaces y JDBC](#repository-dao--interfaces-y-jdbc)
7. [Service — Reglas de negocio](#service--reglas-de-negocio)
8. [UI — Presentación desacoplada](#ui--presentación-desacoplada)
9. [Infraestructura — Conexión y recursos técnicos](#infraestructura--conexión-y-recursos-técnicos)
10. [Pasos de migración](#pasos-de-migración)
11. [Buenas prácticas, pruebas y checklist](#buenas-prácticas-pruebas-y-checklist)
12. [Anexos: UML y snippets útiles](#anexos-uml-y-snippets-útiles)

---

## Estructura de paquetes

```text
com.mycompany.academic
├─ domain/                              # Modelo (POJOs + reglas inmutables ligeras)
│  ├─ Estudiante.java
│  ├─ Nota.java
│  └─ Email.java
├─ exceptions/                          # Excepciones personalizadas
│  ├─ ErrorCode.java
│  ├─ Severity.java
│  ├─ AcademicException.java
│  ├─ NegocioException.java
│  ├─ InfraestructuraException.java
│  ├─ DatoInvalidoException.java
│  ├─ DuplicadoException.java
│  ├─ RecursoNoEncontradoException.java
│  └─ PersistenciaException.java
├─ repository/                          # DAO/Repository (interfaces)
│  ├─ Repositorio.java
│  └─ EstudianteRepository.java
├─ repository/jdbc/                     # Implementaciones JDBC del DAO
│  └─ JdbcEstudianteRepository.java
├─ infra/config/                        # Infraestructura/técnico
│  └─ ConexionDB.java
├─ service/                             # Servicios (interfaces)
│  └─ EstudianteService.java
├─ service/impl/                        # Implementaciones de servicios
│  └─ EstudianteServiceImpl.java
└─ ui/                                  # Presentación (Swing / consola)
   ├─ LoginFrame.java
   └─ RegistroEstudianteFrame.java
```

---

## Por qué usar interfaces (contratos)

**Interfaces = contratos estables.** Definen *qué* hace un componente sin fijar *cómo* lo hace. Ventajas:

- **Bajo acoplamiento:** la UI conoce `EstudianteService`, no su implementación.
- **Sustituibilidad (DIP):** puedes cambiar `JdbcEstudianteRepository` por `InMemoryEstudianteRepository` o `JpaEstudianteRepository` sin tocar servicios o UI.
- **Testabilidad:** en pruebas, inyectas *mocks/fakes* de `EstudianteRepository` sin tocar la BD real.
- **Evolución segura:** puedes extender la implementación sin romper a los consumidores, mientras respetes el contrato.
- **Paralelización del trabajo:** distintos equipos implementan contra el mismo contrato.

> **Regla práctica:** todo lo que sea **borde** o **externo** (BD, red, archivos) debe estar detrás de una **interfaz**.

---

## Descripción de cada capa y clases

- **domain/**: *Modelo del negocio*. Clases simples (**POJOs**) y **Value Objects** (como `Email`).
- **exceptions/**: jerarquía de errores clara para **negar datos inválidos** o **traducir fallos técnicos** en mensajes de negocio.
- **repository/**: contratos para **persistencia** (CRUD + búsquedas). Sin SQL aquí.
- **repository/jdbc/**: **implementación concreta** de los contratos usando JDBC y SQL.
- **service/**: contratos del **caso de uso** (reglas, validaciones, orquestación).
- **service/impl/**: implementación de la lógica **de negocio** que usa repositorios.
- **infra/config/**: utilidades técnicas (conexión BD, configuración).
- **ui/**: presentación (Swing o consola). **No** sabe de SQL ni JDBC, solo usa `EstudianteService`.

---

## Modelo de dominio: POJOs y Value Objects

### `domain/Email.java` (Value Object con validación)

```java
package com.mycompany.academic.domain;

import java.util.Objects;
import java.util.regex.Pattern;

/**
 * Value Object inmutable que representa un email válido.
 * Inmutabilidad = seguridad + reutilización. Valida en el constructor.
 */
public final class Email {
    private static final Pattern REGEX = Pattern.compile("^[\\w._%+-]+@[\\w.-]+\\.[A-Za-z]{2,}$");
    private final String value;

    public Email(String value) {
        if (value == null || !REGEX.matcher(value).matches()) {
            throw new IllegalArgumentException("Email inválido: " + value);
        }
        this.value = value;
    }

    public String toString() { return value; }

    @Override public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Email)) return false;
        Email email = (Email) o;
        return value.equalsIgnoreCase(email.value);
    }

    @Override public int hashCode() { return Objects.hash(value.toLowerCase()); }
}
```

### `domain/Nota.java` (POJO)

```java
package com.mycompany.academic.domain;

/**
 * POJO simple para una nota numérica (0..5, por ejemplo).
 * Podrías convertirlo también en Value Object con validaciones más fuertes.
 */
public class Nota {
    private double valor;

    public Nota(double valor) {
        setValor(valor);
    }
    public double getValor() { return valor; }
    public void setValor(double valor) {
        if (valor < 0.0 || valor > 5.0) { // regla ejemplo
            throw new IllegalArgumentException("La nota debe estar entre 0.0 y 5.0");
        }
        this.valor = valor;
    }
}
```

### `domain/Estudiante.java` (POJO principal)

```java
package com.mycompany.academic.domain;

import java.util.ArrayList;
import java.util.List;

/**
 * Entidad de dominio: Estudiante.
 * Solo lógica mínima de consistencia (no meter lógica de persistencia aquí).
 */
public class Estudiante {
    private Long id;
    private String nombre;
    private Email email;
    private List<Nota> notas = new ArrayList<>();

    public Estudiante(Long id, String nombre, Email email, List<Nota> notas) {
        this.id = id;
        this.nombre = nombre;
        this.email = email;
        if (notas != null) this.notas = notas;
    }

    public Estudiante(String nombre, Email email) {
        this(null, nombre, email, new ArrayList<>());
    }

    // Getters/Setters (podrías volverlos inmutables y usar builder, si prefieres)
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }

    public Email getEmail() { return email; }
    public void setEmail(Email email) { this.email = email; }

    public List<Nota> getNotas() { return notas; }
    public void setNotas(List<Nota> notas) { this.notas = notas; }
}
```

---

## Excepciones personalizadas

### Jerarquía y códigos

```java
package com.mycompany.academic.exceptions;

public enum ErrorCode {
    INVALID_DATA, DUPLICATE_RESOURCE, RESOURCE_NOT_FOUND, DB_ERROR, UNKNOWN
}
```
```java
package com.mycompany.academic.exceptions;

public enum Severity { LOW, MEDIUM, HIGH, CRITICAL }
```

```java
package com.mycompany.academic.exceptions;

/**
 * Base de la jerarquía. Permite adjuntar código y severidad.
 * Úsala para propagar información clara hasta UI/Logs.
 */
public class AcademicException extends RuntimeException {
    private final ErrorCode code;
    private final Severity severity;

    public AcademicException(String message, ErrorCode code, Severity severity, Throwable cause) {
        super(message, cause);
        this.code = code;
        this.severity = severity;
    }
    public AcademicException(String message, ErrorCode code, Severity severity) {
        this(message, code, severity, null);
    }
    public ErrorCode getCode() { return code; }
    public Severity getSeverity() { return severity; }
}
```

Derivadas de negocio e infraestructura:

```java
package com.mycompany.academic.exceptions;

public class NegocioException extends AcademicException {
    public NegocioException(String msg, ErrorCode code) {
        super(msg, code, Severity.MEDIUM);
    }
}
```
```java
package com.mycompany.academic.exceptions;

public class InfraestructuraException extends AcademicException {
    public InfraestructuraException(String msg, Throwable cause) {
        super(msg, ErrorCode.DB_ERROR, Severity.HIGH, cause);
    }
}
```
Especializadas:

```java
package com.mycompany.academic.exceptions;

public class DatoInvalidoException extends NegocioException {
    public DatoInvalidoException(String msg) { super(msg, ErrorCode.INVALID_DATA); }
}
```
```java
package com.mycompany.academic.exceptions;

public class DuplicadoException extends NegocioException {
    public DuplicadoException(String msg) { super(msg, ErrorCode.DUPLICATE_RESOURCE); }
}
```
```java
package com.mycompany.academic.exceptions;

public class RecursoNoEncontradoException extends NegocioException {
    public RecursoNoEncontradoException(String msg) { super(msg, ErrorCode.RESOURCE_NOT_FOUND); }
}
```
```java
package com.mycompany.academic.exceptions;

public class PersistenciaException extends InfraestructuraException {
    public PersistenciaException(String msg, Throwable cause) { super(msg, cause); }
}
```

> **Tip:** En el **repository** traduce `SQLException` → `PersistenciaException` / `DuplicadoException`. En **service**, valida datos y lanza `DatoInvalidoException` / `RecursoNoEncontradoException`.

---

## Repository (DAO) — Interfaces y JDBC

### Contrato genérico y especializado

```java
// repository/Repositorio.java
package com.mycompany.academic.repository;

import java.util.List;
import java.util.Optional;

/**
 * Contrato CRUD genérico.
 * No expone detalles de persistencia (SQL, JDBC) a capas superiores.
 */
public interface Repositorio<T, ID> {
    T crear(T t);
    Optional<T> buscarPorId(ID id);
    List<T> buscarTodos();
    T actualizar(T t);
    void eliminar(ID id);
}
```
```java
// repository/EstudianteRepository.java
package com.mycompany.academic.repository;

import com.mycompany.academic.domain.Estudiante;
import java.util.Optional;

/**
 * Contrato específico del agregado Estudiante.
 * Permite queries especializadas (por email, etc.).
 */
public interface EstudianteRepository extends Repositorio<Estudiante, Long> {
    Optional<Estudiante> buscarPorEmail(String email);
}
```

### Implementación JDBC con comentarios

```java
// repository/jdbc/JdbcEstudianteRepository.java
package com.mycompany.academic.repository.jdbc;

import com.mycompany.academic.domain.Email;
import com.mycompany.academic.domain.Estudiante;
import com.mycompany.academic.exceptions.DuplicadoException;
import com.mycompany.academic.exceptions.PersistenciaException;
import com.mycompany.academic.repository.EstudianteRepository;
import com.mycompany.academic.infra.config.ConexionDB;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

/**
 * Implementación concreta del contrato EstudianteRepository usando JDBC.
 * Responsabilidad: mapear filas <-> objetos y traducir errores de BD a excepciones de dominio.
 */
public class JdbcEstudianteRepository implements EstudianteRepository {

    @Override
    public Estudiante crear(Estudiante e) {
        final String sql = "INSERT INTO estudiante(nombre, email) VALUES (?, ?)";
        try (Connection c = ConexionDB.getConexion();
             PreparedStatement ps = c.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {

            // Mapeo objeto -> parámetros SQL
            ps.setString(1, e.getNombre());
            ps.setString(2, e.getEmail().toString());
            ps.executeUpdate();

            // Recuperar PK generada (AUTO_INCREMENT/IDENTITY)
            try (ResultSet rs = ps.getGeneratedKeys()) {
                if (rs.next()) e.setId(rs.getLong(1));
            }
            return e;
        } catch (SQLException ex) {
            // 1062 (MySQL) o 23505 (Postgres) = UNIQUE VIOLATION
            if (ex.getErrorCode() == 1062 || "23505".equals(ex.getSQLState())) {
                throw new DuplicadoException("Ya existe un estudiante con ese email");
            }
            throw new PersistenciaException("Error al crear estudiante", ex);
        }
    }

    @Override
    public Optional<Estudiante> buscarPorId(Long id) {
        final String sql = "SELECT id, nombre, email FROM estudiante WHERE id = ?";
        try (Connection c = ConexionDB.getConexion();
             PreparedStatement ps = c.prepareStatement(sql)) {

            ps.setLong(1, id);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    // Mapeo fila -> objeto
                    Estudiante e = new Estudiante(
                        rs.getLong("id"),
                        rs.getString("nombre"),
                        new Email(rs.getString("email")),
                        new ArrayList<>()
                    );
                    return Optional.of(e);
                }
                return Optional.empty();
            }
        } catch (SQLException ex) {
            throw new PersistenciaException("Error al consultar estudiante por id", ex);
        }
    }

    @Override
    public List<Estudiante> buscarTodos() {
        final String sql = "SELECT id, nombre, email FROM estudiante";
        List<Estudiante> out = new ArrayList<>();
        try (Connection c = ConexionDB.getConexion();
             PreparedStatement ps = c.prepareStatement(sql);
             ResultSet rs = ps.executeQuery()) {

            while (rs.next()) {
                out.add(new Estudiante(
                    rs.getLong("id"),
                    rs.getString("nombre"),
                    new Email(rs.getString("email")),
                    new ArrayList<>()
                ));
            }
            return out;
        } catch (SQLException ex) {
            throw new PersistenciaException("Error al listar estudiantes", ex);
        }
    }

    @Override
    public Estudiante actualizar(Estudiante e) {
        final String sql = "UPDATE estudiante SET nombre = ?, email = ? WHERE id = ?";
        try (Connection c = ConexionDB.getConexion();
             PreparedStatement ps = c.prepareStatement(sql)) {

            ps.setString(1, e.getNombre());
            ps.setString(2, e.getEmail().toString());
            ps.setLong(3, e.getId());
            ps.executeUpdate();
            return e;
        } catch (SQLException ex) {
            throw new PersistenciaException("Error al actualizar estudiante", ex);
        }
    }

    @Override
    public void eliminar(Long id) {
        final String sql = "DELETE FROM estudiante WHERE id = ?";
        try (Connection c = ConexionDB.getConexion();
             PreparedStatement ps = c.prepareStatement(sql)) {

            ps.setLong(1, id);
            ps.executeUpdate();
        } catch (SQLException ex) {
            throw new PersistenciaException("Error al eliminar estudiante", ex);
        }
    }

    @Override
    public Optional<Estudiante> buscarPorEmail(String email) {
        final String sql = "SELECT id, nombre, email FROM estudiante WHERE email = ?";
        try (Connection c = ConexionDB.getConexion();
             PreparedStatement ps = c.prepareStatement(sql)) {

            ps.setString(1, email);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    return Optional.of(new Estudiante(
                        rs.getLong("id"),
                        rs.getString("nombre"),
                        new Email(rs.getString("email")),
                        new ArrayList<>()
                    ));
                }
                return Optional.empty();
            }
        } catch (SQLException ex) {
            throw new PersistenciaException("Error al buscar por email", ex);
        }
    }
}
```

---

## Service — Reglas de negocio

### Contrato e implementación

```java
// service/EstudianteService.java
package com.mycompany.academic.service;

import com.mycompany.academic.domain.Estudiante;
import java.util.List;

/**
 * Casos de uso para Estudiante.
 * Aquí NO va SQL ni detalles de persistencia.
 */
public interface EstudianteService {
    Estudiante registrar(Estudiante e);
    Estudiante actualizar(Estudiante e);
    Estudiante consultarPorId(Long id);
    List<Estudiante> listar();
    void eliminar(Long id);
}
```
```java
// service/impl/EstudianteServiceImpl.java
package com.mycompany.academic.service.impl;

import com.mycompany.academic.domain.Estudiante;
import com.mycompany.academic.exceptions.DatoInvalidoException;
import com.mycompany.academic.exceptions.RecursoNoEncontradoException;
import com.mycompany.academic.repository.EstudianteRepository;
import com.mycompany.academic.service.EstudianteService;

import java.util.List;

/**
 * Implementa reglas de negocio: validación, orquestación, consistencia.
 * Depende de la interfaz EstudianteRepository (DIP).
 */
public class EstudianteServiceImpl implements EstudianteService {

    private final EstudianteRepository repo;

    public EstudianteServiceImpl(EstudianteRepository repo) {
        this.repo = repo;
    }

    @Override
    public Estudiante registrar(Estudiante e) {
        if (e == null || e.getNombre() == null || e.getNombre().isBlank())
            throw new DatoInvalidoException("Nombre requerido");

        // Regla de unicidad por email
        repo.buscarPorEmail(e.getEmail().toString())
            .ifPresent(x -> { throw new DatoInvalidoException("Email ya registrado"); });

        return repo.crear(e);
    }

    @Override
    public Estudiante actualizar(Estudiante e) {
        consultarPorId(e.getId()); // garantiza existencia previa
        return repo.actualizar(e);
    }

    @Override
    public Estudiante consultarPorId(Long id) {
        return repo.buscarPorId(id)
                .orElseThrow(() -> new RecursoNoEncontradoException("Estudiante no encontrado"));
    }

    @Override
    public List<Estudiante> listar() { return repo.buscarTodos(); }

    @Override
    public void eliminar(Long id) {
        consultarPorId(id); // valida existencia antes de borrar
        repo.eliminar(id);
    }
}
```

---

## UI — Presentación desacoplada

```java
// ui/RegistroEstudianteFrame.java (fragmento conceptual)
package com.mycompany.academic.ui;

import com.mycompany.academic.domain.Email;
import com.mycompany.academic.domain.Estudiante;
import com.mycompany.academic.repository.jdbc.JdbcEstudianteRepository;
import com.mycompany.academic.service.EstudianteService;
import com.mycompany.academic.service.impl.EstudianteServiceImpl;

import javax.swing.*;
import java.awt.*;

public class RegistroEstudianteFrame extends JFrame {
    private final EstudianteService service;

    public RegistroEstudianteFrame() {
        // "Inyección manual": en producción podrías usar un contenedor DI
        this.service = new EstudianteServiceImpl(new JdbcEstudianteRepository());
        initUI();
    }

    private void initUI() {
        setTitle("Registro de Estudiantes");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(400, 200);

        JTextField txtNombre = new JTextField();
        JTextField txtEmail = new JTextField();
        JButton btnGuardar = new JButton("Guardar");

        btnGuardar.addActionListener(e -> {
            try {
                var est = new Estudiante(txtNombre.getText(), new Email(txtEmail.getText()));
                service.registrar(est);
                JOptionPane.showMessageDialog(this, "Estudiante registrado con id: " + est.getId());
            } catch (Exception ex) {
                // Muestra mensajes amigables (de negocio) o técnicos según el tipo
                JOptionPane.showMessageDialog(this, ex.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            }
        });

        setLayout(new GridLayout(3, 2));
        add(new JLabel("Nombre:")); add(txtNombre);
        add(new JLabel("Email:"));  add(txtEmail);
        add(new JLabel());          add(btnGuardar);
    }
}
```

> **Clave:** La UI solo conoce el **contrato** `EstudianteService`. No hay SQL ni `Connection` en esta capa.

---

## Infraestructura — Conexión y recursos técnicos

```java
// infra/config/ConexionDB.java
package com.mycompany.academic.infra.config;

import java.sql.Connection;
import java.sql.DriverManager;
import java.util.Properties;

/**
 * Proveedor de conexiones simple (para demo).
 * En proyectos reales, usa DataSource y pooling (HikariCP, etc.).
 */
public final class ConexionDB {
    private static final String URL  = "jdbc:mysql://localhost:3306/academico?useSSL=false";
    private static final String USER = "root";
    private static final String PASS = "root";

    private ConexionDB() {}

    public static Connection getConexion() {
        try {
            Properties p = new Properties();
            p.setProperty("user", USER);
            p.setProperty("password", PASS);
            return DriverManager.getConnection(URL, p);
        } catch (Exception e) {
            throw new RuntimeException("No se pudo obtener la conexión", e);
        }
    }
}
```

> **Sugerido:** mover credenciales a `db.properties` y cargarlas por `ClassLoader`. Para MySQL, usa driver `com.mysql.cj.jdbc.Driver` y dependencias vía Maven/Gradle.

---

## Pasos de migración

1. **Mover** `ConexionDB.java` a `infra/config` y ajustar imports.
2. **Crear** `repository/Repositorio.java` y `repository/EstudianteRepository.java` (contratos).
3. **Renombrar** `db/EstudianteDAO.java` → `repository/jdbc/JdbcEstudianteRepository.java` e implementar la interfaz.
4. **Crear** `service/EstudianteService.java` y **migrar** tu `RegistroEstudiantesServiceImpl` a `service/impl/EstudianteServiceImpl`.
5. **Actualizar UI** para depender de `EstudianteService` (no de impls concretas).
6. **Agregar excepciones** en `exceptions/` y **traducir** `SQLException` en el repository.
7. **Probar**: primero `repository` con una BD de prueba; luego `service` con *fakes*; finalmente `ui`.

---

## Buenas prácticas, pruebas y checklist

**Buenas prácticas**
- **DIP:** capas superiores dependen de **interfaces**, no de implementaciones.
- **Single Responsibility:** cada clase hace una cosa clara (mapea, valida, orquesta, pinta).
- **Validación en Service y VOs:** evita que datos inválidos lleguen a BD.
- **Manejo de transacciones:** si en el futuro tienes múltiples operaciones, encapsula en `service` una *transacción* (con commit/rollback).
- **Logs y auditoría:** loguea en `repository`/`infra`; mensajes amigables en `service`/`ui`.

**Pruebas**
- **Unitarias (service):** inyecta `EstudianteRepository` *fake* para probar reglas sin BD.
- **Integración (repository):** usa una BD real o contenedor (Testcontainers).
- **UI:** prueba handlers con mocks de `EstudianteService`.

**Checklist**
- [x] UI no conoce JDBC/SQL.
- [x] `ServiceImpl` usa solo interfaces de repositorio.
- [x] `Jdbc*Repository` traduce `SQLException` a excepciones personalizadas.
- [x] `ConexionDB` centralizada y configurable.
- [x] Value Objects (`Email`) validan desde el modelo.
- [x] Estructura de paquetes limpia y coherente.

---

## Anexos: UML y snippets útiles

**Diagrama (texto) de dependencias**

```
[UI] ---> (EstudianteService) <--- [ServiceImpl] ---> (EstudianteRepository) <--- [JdbcEstudianteRepository] ---> [BD]
                 ^                        ^
              Contrato                Implementa
```

**Mock simple para pruebas de Service**

```java
class InMemoryEstudianteRepository implements EstudianteRepository {
    private final Map<Long, Estudiante> db = new HashMap<>();
    private long seq = 0;

    @Override public Estudiante crear(Estudiante e) { e.setId(++seq); db.put(e.getId(), e); return e; }
    @Override public Optional<Estudiante> buscarPorId(Long id) { return Optional.ofNullable(db.get(id)); }
    @Override public List<Estudiante> buscarTodos() { return new ArrayList<>(db.values()); }
    @Override public Estudiante actualizar(Estudiante e) { db.put(e.getId(), e); return e; }
    @Override public void eliminar(Long id) { db.remove(id); }
    @Override public Optional<Estudiante> buscarPorEmail(String email) {
        return db.values().stream().filter(x -> x.getEmail().toString().equalsIgnoreCase(email)).findFirst();
    }
}
```

---

## Conclusión

Con esta estructura y los **contratos por interfaces**, tu código queda preparado para crecer: podrás cambiar de JDBC a JPA, migrar a microservicios, o añadir caché sin reescribir la UI ni las reglas de negocio. Además, **probar** tu lógica será más fácil y rápido.