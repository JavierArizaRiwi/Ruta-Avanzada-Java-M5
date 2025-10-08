# Comparativa y Guía Práctica: **Java SE**, **Java EE (WildFly/JBoss) con EJB + JAX‑RS**, y **Spring Boot**

> **Nota clave:** Según tu indicación, **NO** usaremos *Servlets* en Java EE; la capa “controller” será **JAX‑RS** (`@Path`) y la lógica de negocio será **EJB** (`@Stateless`).

---

## 1) Diferencia general entre **Java SE** y **Java EE/Jakarta EE**

| Característica | **Java SE (Standard Edition)** | **Java EE/Jakarta EE (Enterprise)** |
|---|---|---|
| Enfoque | Apps de consola/escritorio | Apps empresariales (web/API/mensajería) |
| Librerías | Core Java (Collections, Concurrency, JDBC, I/O) | Servidor de apps: **JAX‑RS**, **EJB**, **JPA**, **CDI**, **JMS**, **Bean Validation**, etc. |
| Ejecución | JVM local | Servidor de aplicaciones (**WildFly/JBoss**, Payara, GlassFish) |
| Inyección | Manual o frameworks externos | Nativa con **CDI** y **EJB** |
| Arquitectura | Sencilla/monolítica | Multicapa y transaccional |

> **Java SE = ejecución local y directa.**  
> **Java EE = ejecución gestionada en un servidor, con inyección, transacciones y seguridad.**

---

## 2) Capas en una app Java EE/Jakarta EE (con JAX‑RS)

```
PRESENTACIÓN (JSF/Front SPA) → API (JAX‑RS @Path) → LÓGICA (EJB) → PERSISTENCIA (JPA) → BD
```

- **API (Controlador):** recursos **JAX‑RS** (`@Path`, `@GET`, `@POST`…), devuelven JSON/HTTP.
- **Lógica:** **EJB** (`@Stateless`) encapsulan reglas, transacciones y orquestación.
- **Persistencia:** **JPA** con `EntityManager` y entidades `@Entity`.
- **CDI:** pegamento para inyección (`@Inject`) y estereotipos (`@Named`).

---

## 3) **WildFly/JBoss** como servidor de aplicaciones

**¿Qué aporta?**

- **Ciclo de vida** de beans (EJB/CDI), **inyección**, **JNDI**, **transacciones (JTA)**, **seguridad**,
  **pools** (hilos y conexiones), despliegue **WAR**/**EAR**, **observabilidad** (logging/subsistemas).

**Arranque básico**

```bash
# Linux/macOS
./bin/standalone.sh
# Windows
bin\standalone.bat
# Consola de administración
http://localhost:9990  (crear usuario con add-user.sh si lo necesitas)
# Apps (HTTP)
http://localhost:8080
```

---

## 4) Proyecto de ejemplo **Java EE con EJB + JAX‑RS** (sin Servlets)

### 4.1 Estructura mínima (Maven, packaging WAR)

```
miapp/
 ├─ pom.xml
 └─ src/main/
    ├─ java/com/ejemplo/
    │  ├─ api/AppConfig.java           # Activa JAX‑RS en /api/*
    │  ├─ api/UsuarioResource.java     # Controlador JAX‑RS
    │  ├─ domain/Usuario.java          # @Entity JPA
    │  └─ service/UsuarioService.java  # @Stateless EJB
    ├─ resources/META-INF/persistence.xml
    └─ webapp/WEB-INF/beans.xml        # Activa CDI (vacío está bien)
```

### 4.2 `pom.xml` (Jakarta EE 10; APIs con `provided` porque las da WildFly)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.ejemplo</groupId>
  <artifactId>miapp-jaxrs-ejb</artifactId>
  <version>1.0.0</version>
  <packaging>war</packaging>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <jakartaee.version>10.0.0</jakartaee.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>jakarta.platform</groupId>
      <artifactId>jakarta.jakartaee-api</artifactId>
      <version>${jakartaee.version}</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>3.4.0</version>
      </plugin>
      <!-- Opcional: despliegue directo a WildFly -->
      <plugin>
        <groupId>org.wildfly.plugins</groupId>
        <artifactId>wildfly-maven-plugin</artifactId>
        <version>5.0.0.Final</version>
        <configuration>
          <hostname>localhost</hostname>
          <port>9990</port>
          <username>admin</username>
          <password>admin</password>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

### 4.3 **Activar JAX‑RS** (`/api/*`)

```java
package com.ejemplo.api;

import jakarta.ws.rs.ApplicationPath;
import jakarta.ws.rs.core.Application;

@ApplicationPath("/api")
public class AppConfig extends Application {
  // Vacío: solo habilita JAX‑RS bajo /api/*
}
```

### 4.4 **Entidad JPA**

```java
package com.ejemplo.domain;

import jakarta.persistence.*;

@Entity
@Table(name = "usuarios")
public class Usuario {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(nullable = false, length = 100)
  private String nombre;

  // getters/setters
  public Long getId() { return id; }
  public void setId(Long id) { this.id = id; }
  public String getNombre() { return nombre; }
  public void setNombre(String nombre) { this.nombre = nombre; }
}
```

### 4.5 **EJB de negocio** (`@Stateless`) con JPA

```java
package com.ejemplo.service;

import com.ejemplo.domain.Usuario;
import jakarta.ejb.Stateless;
import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import java.util.List;

@Stateless
public class UsuarioService {

  @PersistenceContext
  EntityManager em;

  public Usuario crear(String nombre) {
    Usuario u = new Usuario();
    u.setNombre(nombre);
    em.persist(u);
    return u;
  }

  public List<Usuario> listar() {
    return em.createQuery("SELECT u FROM Usuario u", Usuario.class).getResultList();
  }
}
```

### 4.6 **Recurso JAX‑RS** (controlador HTTP **sin** servlets)

```java
package com.ejemplo.api;

import com.ejemplo.domain.Usuario;
import com.ejemplo.service.UsuarioService;
import jakarta.ejb.EJB;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;
import java.util.List;

@Path("/usuarios")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class UsuarioResource {

  @EJB
  UsuarioService servicio;

  public static class CrearUsuarioDTO {
    public String nombre;
  }

  @GET
  public List<Usuario> listar() {
    return servicio.listar();
  }

  @POST
  public Response crear(CrearUsuarioDTO dto) {
    if (dto == null || dto.nombre == null || dto.nombre.isBlank()) {
      return Response.status(Response.Status.BAD_REQUEST)
                     .entity("{\"error\":\"nombre requerido\"}")
                     .build();
    }
    Usuario u = servicio.crear(dto.nombre.trim());
    return Response.status(Response.Status.CREATED).entity(u).build();
  }
}
```

### 4.7 **`persistence.xml`** (JPA) y **beans.xml** (CDI)

`src/main/resources/META-INF/persistence.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence" version="3.0">
  <persistence-unit name="miPU" transaction-type="JTA">
    <jta-data-source>java:/jdbc/MiDS</jta-data-source>
    <class>com.ejemplo.domain.Usuario</class>
    <properties>
      <property name="jakarta.persistence.schema-generation.database.action" value="create"/>
      <!-- Si usas Hibernate como provider del servidor, puedes ajustar dialecto, etc. -->
    </properties>
  </persistence-unit>
</persistence>
```

`src/main/webapp/WEB-INF/beans.xml` (vacío mínimamente válido)

```xml
<beans xmlns="https://jakarta.ee/xml/ns/jakartaee" version="4.0"/>
```

### 4.8 **Configurar DataSource en WildFly (CLI)**

```bash
# Con WildFly corriendo:
$ WILDFLY_HOME/bin/jboss-cli.sh --connect

# Ejemplo: driver H2 embebido (ajusta a tu BD real)
module add --name=com.h2database.h2 --resources=/ruta/h2-2.x.x.jar --dependencies=jakarta.transaction.api,javax.api
/subsystem=datasources/jdbc-driver=h2:add(driver-name=h2, driver-module-name=com.h2database.h2)

# DataSource JTA
/subsystem=datasources/data-source=MiDS:add(jndi-name=java:/jdbc/MiDS, driver-name=h2, connection-url=jdbc:h2:mem:test;DB_CLOSE_DELAY=-1, user-name=sa, password=sa)
:reload
```

### 4.9 **Compilar y desplegar**

```bash
mvn clean package
# Opción A: copiar target/miapp-jaxrs-ejb.war a WILDFLY_HOME/standalone/deployments
# Opción B: usar el plugin
mvn wildfly:deploy
```

**Probar API:**

- `GET  http://localhost:8080/miapp-jaxrs-ejb/api/usuarios`
- `POST http://localhost:8080/miapp-jaxrs-ejb/api/usuarios` con JSON `{"nombre":"Javier"}`

---

## 5) Anotaciones clave (sin servlets)

| Capa | Anotación | Descripción |
|---|---|---|
| API | `@Path`, `@GET`, `@POST`, `@PUT`, `@DELETE`, `@Produces`, `@Consumes` | **JAX‑RS** para exponer endpoints REST |
| Lógica | `@Stateless`, `@Stateful`, `@Singleton` | **EJB** para reglas, transacciones y concurrencia |
| Persistencia | `@Entity`, `@Id`, `@GeneratedValue`, `@Column`, `@Table` | **JPA** para mapear entidades |
| Inyección | `@Inject` (CDI), `@EJB`, `@PersistenceContext` | Inyección de beans y `EntityManager` |
| Configuración | `@ApplicationPath` | Activa JAX‑RS y define el prefijo base |

---

## 6) Comparación con **Spring Boot** (equivalencias conceptuales)

| Capa | **Java EE (WildFly)** | **Spring Boot** |
|---|---|---|
| API | **JAX‑RS** (`@Path`) | **Spring MVC** `@RestController` |
| Lógica | **EJB** (`@Stateless`) | `@Service` + `@Transactional` |
| Persistencia | `EntityManager` (JPA) | `Spring Data JPA` (`JpaRepository`) |
| Inyección | CDI `@Inject` / `@EJB` | `@Autowired` |
| Empaquetado | WAR (servidor externo) | JAR ejecutable (Tomcat embebido) |

**Ejemplo Spring Boot equivalente:**

```java
@Service
public class UsuarioService {
  @Autowired UsuarioRepository repo;
  public Usuario crear(String nombre){ return repo.save(new Usuario(null, nombre)); }
  public List<Usuario> listar(){ return repo.findAll(); }
}

@RestController
@RequestMapping("/api/usuarios")
public class UsuarioController {
  @Autowired UsuarioService servicio;
  @GetMapping public List<Usuario> listar(){ return servicio.listar(); }
  @PostMapping public ResponseEntity<Usuario> crear(@RequestBody Map<String,String> body){
    var u = servicio.crear(body.get("nombre"));
    return ResponseEntity.status(201).body(u);
  }
}
```

---

## 7) Migración conceptual **EE → Spring Boot**

| De (EE) | A (Spring) |
|---|---|
| `@Stateless` | `@Service` + `@Transactional` |
| `@Inject`/`@EJB` | `@Autowired` |
| `EntityManager` directo | `JpaRepository` o `EntityManager` con `@PersistenceContext` |
| WAR + WildFly | JAR con Tomcat embebido |
| `@ApplicationPath("/api")` + JAX‑RS | `@RestController` + `@RequestMapping("/api")` |

---

## 8) Recomendación de ruta formativa para tu equipo

1. **Base EE con JAX‑RS:** CRUD sencillo EJB + JPA + endpoints JAX‑RS.
2. **Transacciones y validación:** `@Transactional`/CMT y Bean Validation (`@NotNull`, `@Size`).  
3. **Seguridad:** realm nativo de WildFly o Keycloak (JWT/OIDC) + filtros JAX‑RS.  
4. **Pruebas:** `arquillian` / pruebas de integración con contenedor embebido.  
5. **Modernización:** traducir EJB a `@Service` y JAX‑RS a `@RestController` si migran a Spring.

---

## 9) Checklist de despliegue en **WildFly**

- [ ] WildFly corriendo en **Java 17**.  
- [ ] **DataSource JTA** con JNDI `java:/jdbc/MiDS`.  
- [ ] `persistence.xml` con `<jta-data-source>`.  
- [ ] `beans.xml` presente para **CDI**.  
- [ ] Empaquetado **WAR** y desplegado en `standalone/deployments/`.  
- [ ] Endpoints accesibles en `/api/*` (sin servlets).

---

## 10) Troubleshooting rápido

- **`ClassNotFoundException` de APIs Jakarta:** añade `jakarta.jakartaee-api` con `scope=provided` y usa una versión de WildFly compatible con Jakarta EE 10.
- **`NameNotFoundException` JNDI:** revisa nombre del DataSource (`java:/jdbc/…`) y la activación del módulo del driver.
- **`NoSuchMethodError`/choque de libs:** no empaquetes las APIs Jakarta dentro del WAR; las provee el servidor.
- **CORS en JAX‑RS:** agrega un `ContainerResponseFilter` para añadir headers CORS si un front SPA consume tu API.

```java
@Provider
public class CorsFilter implements ContainerResponseFilter {
  @Override
  public void filter(ContainerRequestContext req, ContainerResponseContext res) {
    res.getHeaders().add("Access-Control-Allow-Origin", "*");
    res.getHeaders().add("Access-Control-Allow-Headers", "origin, content-type, accept, authorization");
    res.getHeaders().add("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS, HEAD");
  }
}
```

---

## 11) Mini‑roadmap de **producción**

- **Logs estructurados** (JSON) y **MDC** para trazabilidad.  
- **Métricas** vía MicroProfile Metrics / Prometheus exporter.  
- **Health checks** con MicroProfile Health.  
- **Seguridad** con Keycloak (OIDC), roles y *role-based constraints*.  
- **Capa DTO/Mapper** (MapStruct) para no exponer entidades JPA.

---

### Fin — Plantilla lista para usar
Compila, configura el DataSource y tendrás un **API REST JAX‑RS** corriendo en WildFly, con **EJB** como capa de negocio y **JPA** para la persistencia — **sin servlets**.