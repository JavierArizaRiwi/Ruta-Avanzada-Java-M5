# Estructuración de Proyecto Académico con Arquitectura por Capas y Excepciones Personalizadas

## Introducción

Esta guía explica cómo reestructurar el proyecto **academic** aplicando una **arquitectura por capas profesional** y un manejo robusto de **excepciones personalizadas** utilizando `enum` para la gestión de errores.  
El objetivo es lograr un sistema modular, mantenible y escalable.

---

## 1. Estructura final del proyecto

```text
com.mycompany.academic
├─ domain/                      # Clases del dominio (modelo y reglas)
│  ├─ Estudiante.java
│  ├─ Nota.java
│  └─ Email.java
├─ dto/                         # Transferencia de datos entre UI y Service
│  ├─ EstudianteRequest.java
│  └─ EstudianteResponse.java
├─ exceptions/                  # Jerarquía de excepciones
│  ├─ ErrorCode.java
│  ├─ Severity.java
│  ├─ AcademicException.java
│  ├─ NegocioException.java
│  ├─ InfraestructuraException.java
│  ├─ DatoInvalidoException.java
│  ├─ DuplicadoException.java
│  ├─ RecursoNoEncontradoException.java
│  └─ PersistenciaException.java
├─ mapper/                      # Conversión entre entidad y DTO
│  └─ EstudianteMapper.java
├─ repository/                  # Contratos (DAO / Repository)
│  └─ EstudianteRepository.java
├─ infra/                       # Implementaciones técnicas
│  ├─ config/
│  │  └─ ConexionDB.java
│  └─ persistence/jdbc/
│     └─ JdbcEstudianteRepository.java
├─ service/
│  ├─ EstudianteService.java
│  └─ impl/
│     ├─ EstudianteServiceImpl.java
│     ├─ ArchivoServiceImpl.java
│     └─ UsuarioServiceImpl.java
└─ ui/                          # Interfaz de usuario (Swing o consola)
   ├─ LoginFrame.java
   └─ RegistroEstudianteFrame.java
```

---

## 2. Responsabilidades de las capas

| Capa | Responsabilidad |
|------|------------------|
| **Presentación (UI)** | Interactúa con el usuario, captura excepciones y muestra mensajes. |
| **Servicio (Service)** | Aplica las reglas de negocio y lanza excepciones personalizadas. |
| **Repositorio (DAO)** | Accede a la base de datos, traduce errores SQL. |
| **Dominio (Model)** | Define las entidades y validaciones internas. |

---

## 3. Manejo de excepciones con Enums

### 3.1 Enum `ErrorCode` y `Severity`

```java
public enum ErrorCode {
    INVALID_DATA("E001", "Datos inválidos o incompletos", Severity.WARN),
    DUPLICATE_RESOURCE("E002", "El recurso ya existe", Severity.WARN),
    RESOURCE_NOT_FOUND("E003", "Recurso no encontrado", Severity.INFO),
    DB_ERROR("E100", "Error general de base de datos", Severity.ERROR),
    DB_CONNECTION_FAILED("E101", "Error de conexión con la base de datos", Severity.FATAL),
    TRANSACTION_ERROR("E102", "Error durante la transacción", Severity.ERROR),
    UNKNOWN_ERROR("E999", "Error desconocido", Severity.FATAL);

    private final String code;
    private final String message;
    private final Severity severity;

    ErrorCode(String code, String message, Severity severity) {
        this.code = code;
        this.message = message;
        this.severity = severity;
    }

    public String getCode() { return code; }
    public String getMessage() { return message; }
    public Severity getSeverity() { return severity; }
}
```

```java
public enum Severity {
    INFO, WARN, ERROR, FATAL
}
```

---

### 3.2 Excepción base y jerarquía

```java
public abstract class AcademicException extends RuntimeException {
    private final ErrorCode errorCode;

    protected AcademicException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }

    protected AcademicException(ErrorCode errorCode, Throwable cause) {
        super(errorCode.getMessage(), cause);
        this.errorCode = errorCode;
    }

    public ErrorCode getErrorCode() { return errorCode; }
    public String getCode() { return errorCode.getCode(); }
    public Severity getSeverity() { return errorCode.getSeverity(); }
}
```

Subclases:

```java
public abstract class NegocioException extends AcademicException {
    protected NegocioException(ErrorCode code) { super(code); }
}

public abstract class InfraestructuraException extends AcademicException {
    protected InfraestructuraException(ErrorCode code, Throwable cause) { super(code, cause); }
}

public class DatoInvalidoException extends NegocioException {
    public DatoInvalidoException() { super(ErrorCode.INVALID_DATA); }
}

public class DuplicadoException extends NegocioException {
    public DuplicadoException() { super(ErrorCode.DUPLICATE_RESOURCE); }
}

public class RecursoNoEncontradoException extends NegocioException {
    public RecursoNoEncontradoException() { super(ErrorCode.RESOURCE_NOT_FOUND); }
}

public class PersistenciaException extends InfraestructuraException {
    public PersistenciaException(Throwable cause) { super(ErrorCode.DB_ERROR, cause); }
}
```

---

## 4. Validaciones de dominio

```java
public class Estudiante {
    private Long id;
    private String nombre;
    private Email email;

    public Estudiante(Long id, String nombre, Email email) {
        if (nombre == null || nombre.isBlank())
            throw new DatoInvalidoException();
        if (email == null)
            throw new DatoInvalidoException();
        this.id = id;
        this.nombre = nombre.trim();
        this.email = email;
    }
}
```

---

## 5. Repositorio y persistencia JDBC

**Interfaz:**

```java
public interface EstudianteRepository {
    Estudiante save(Estudiante e);
    Optional<Estudiante> findById(Long id);
    List<Estudiante> findAll();
    void deleteById(Long id);
}
```

**Implementación JDBC:**

```java
public class JdbcEstudianteRepository implements EstudianteRepository {
    private final DataSource ds;

    public JdbcEstudianteRepository(DataSource ds) {
        this.ds = ds;
    }

    @Override
    public Estudiante save(Estudiante e) {
        final String sql = "INSERT INTO estudiante(nombre,email) VALUES(?,?)";
        try (Connection c = ds.getConnection();
             PreparedStatement ps = c.prepareStatement(sql)) {
            ps.setString(1, e.getNombre());
            ps.setString(2, e.getEmail().toString());
            ps.executeUpdate();
            return e;
        } catch (SQLException ex) {
            if (ex.getErrorCode() == 1062)
                throw new DuplicadoException();
            throw new PersistenciaException(ex);
        }
    }
}
```

---

## 6. Capa de servicio

```java
public interface EstudianteService {
    EstudianteResponse registrar(EstudianteRequest req);
    EstudianteResponse buscarPorId(Long id);
    List<EstudianteResponse> listar();
}
```

```java
public class EstudianteServiceImpl implements EstudianteService {
    private final EstudianteRepository repo;

    public EstudianteServiceImpl(EstudianteRepository repo) {
        this.repo = repo;
    }

    @Override
    public EstudianteResponse registrar(EstudianteRequest req) {
        if (req.getNombre() == null || req.getNombre().isBlank())
            throw new DatoInvalidoException();

        repo.findById(req.getId()).ifPresent(e -> { throw new DuplicadoException(); });

        Estudiante nuevo = EstudianteMapper.toDomain(req);
        Estudiante guardado = repo.save(nuevo);
        return EstudianteMapper.toResponse(guardado);
    }
}
```

---

## 7. Capa de presentación (Swing o consola)

```java
try {
    EstudianteRequest req = new EstudianteRequest();
    req.setNombre(txtNombre.getText());
    req.setEmail(txtEmail.getText());

    EstudianteResponse res = estudianteService.registrar(req);
    JOptionPane.showMessageDialog(this, "Estudiante creado: " + res.getNombre());
} catch (NegocioException e) {
    JOptionPane.showMessageDialog(this, e.getErrorCode().getMessage(),
        e.getSeverity().name(), JOptionPane.WARNING_MESSAGE);
} catch (InfraestructuraException e) {
    JOptionPane.showMessageDialog(this, "Error técnico: " + e.getMessage(),
        "ERROR", JOptionPane.ERROR_MESSAGE);
}
```

---

## 8. Beneficios de esta estructura

| Beneficio | Descripción |
|------------|-------------|
| **Modularidad** | Cada capa tiene su responsabilidad definida. |
| **Reutilización** | Los componentes pueden reutilizarse fácilmente. |
| **Escalabilidad** | Facilita futuras migraciones a frameworks modernos. |
| **Mantenibilidad** | Código limpio, organizado y con excepciones centralizadas. |
| **Trazabilidad** | Los códigos de error (`E001`, `E002`, etc.) permiten rastrear los problemas rápidamente. |

---

**Autor:** Javier Ariza  
**Guía Técnica:** Arquitectura por Capas con Excepciones Personalizadas en Java