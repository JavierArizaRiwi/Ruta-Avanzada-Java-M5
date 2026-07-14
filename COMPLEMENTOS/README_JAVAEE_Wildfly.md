# Guía completa: Configurar WildFly/JBoss para Java EE (Jakarta EE) con EJB y “controller” (JAX‑RS o Spring MVC)

> **Resumen**  
> Esta guía te muestra **dos rutas** de trabajo en WildFly (antes JBoss):  
> **A)** *Jakarta EE puro*: **EJB** para la lógica y **JAX‑RS** (`@Path`) como “controller” REST.  
> **B)** *Spring MVC*: usar `@Controller` de **Spring** desplegado en WildFly e **invocar EJB** por JNDI.  
> Incluye **estructura de proyecto**, **pom.xml**, **clases con comentarios clave**, **comandos**, **despliegue** y **troubleshooting**.

---

## 0) Prerrequisitos

- **Java 17 o 21** (o compatible con tu versión de WildFly).  
- **Maven 3.8+**.  
- **WildFly 30.x** (o versión estable reciente).  
- Conexión local a `localhost:8080` para apps y `localhost:9990` para la consola de administración (Management).

---

## 1) Instalar y arrancar WildFly

1. **Descarga** WildFly desde la página oficial.
2. **Descomprime** en una carpeta, por ejemplo `WILDFLY_HOME=/opt/wildfly-30.0.0.Final`.
3. **Arranca** en modo standalone:
   - Linux/macOS: `./bin/standalone.sh`
   - Windows: `bin\standalone.bat`
4. (Opcional) **Usuario de administración** para la consola:  
   Ejecuta `bin/add-user.sh` y crea un usuario **Management** (por ejemplo, `admin/admin`).

**Puertos por defecto**
- Apps: `http://localhost:8080/`
- Console: `http://localhost:9990/` (Management)

> **Tip**: si te aparece “Address already in use”, verifica otros servidores en uso o cambia el puerto (`-Djboss.http.port=8081`).

---

# RUTA A) Jakarta EE puro (EJB + JAX‑RS como “controller”)

### ¿Cuándo conviene?
- Quieres **estándar Jakarta EE** sin dependencias adicionales.  
- Deseas **EJB** para transacciones, pooling y seguridad, y **JAX‑RS** como capa “controller” REST.

---

## 2A) Estructura mínima del proyecto

```
demo-ejb-jaxrs/
├─ pom.xml
└─ src/
   └─ main/
      ├─ java/
      │  └─ com/ejemplo/
      │     ├─ AppConfig.java                 # Activa JAX-RS en /api/*
      │     ├─ boundary/HelloResource.java    # “Controller” REST (JAX-RS)
      │     └─ control/SaludoService.java     # Lógica de negocio (EJB)
      └─ webapp/
         └─ WEB-INF/                          # (vacío; sin web.xml en Jakarta EE 10)
```

---

## 3A) `pom.xml` (APIs provistas por el servidor → `provided`)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.ejemplo</groupId>
  <artifactId>demo-ejb-jaxrs</artifactId>
  <version>1.0.0</version>
  <packaging>war</packaging>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <jakartaee.version>10.0.0</jakartaee.version>
    <wildfly.version>30.0.0.Final</wildfly.version>
  </properties>

  <dependencies>
    <!-- APIs Jakarta EE (EJB, CDI, JAX-RS), provistas por WildFly -->
    <dependency>
      <groupId>jakarta.platform</groupId>
      <artifactId>jakarta.jakartaee-api</artifactId>
      <version>${jakartaee.version}</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- Despliegue directo vía Management API (opcional) -->
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

> **Claves**  
> - `scope=provided`: el servidor (WildFly) provee las APIs Jakarta.  
> - `packaging=war`: empaquetamos una aplicación web con endpoints REST.

---

## 4A) Código con **comentarios clave**

### `control/SaludoService.java` (EJB)

```java
package com.ejemplo.control;

import jakarta.ejb.Stateless;

/**
 * EJB sin estado:
 * - Ideal para servicios CRUD/REST.
 * - Container-managed transactions (CMT) por defecto.
 * - Pooling, seguridad y concurrencia administrados por el contenedor.
 */
@Stateless
public class SaludoService {

    /**
     * Regla simple de negocio. En un caso real:
     * - Validaciones
     * - Acceso a repositorios (JPA/JDBC)
     * - Transacciones
     */
    public String saludar(String nombre) {
        String n = (nombre == null || nombre.isBlank()) ? "mundo" : nombre.trim();
        return "Hola, " + n + " 👋";
    }
}
```

### `boundary/HelloResource.java` (JAX‑RS “controller” REST)

```java
package com.ejemplo.boundary;

import com.ejemplo.control.SaludoService;
import jakarta.ejb.EJB;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;

/**
 * Recurso JAX-RS = “controller” en el mundo Jakarta EE:
 * - Define rutas REST con @Path
 * - Produce/Consume JSON o XML
 * - Orquesta servicios (EJB) y mapea HTTP ↔ negocio
 */
@Path("/hello")
@Produces(MediaType.APPLICATION_JSON)
public class HelloResource {

    // Inyección directa de EJB
    @EJB
    SaludoService saludoService;

    @GET
    public Response hello(@QueryParam("nombre") String nombre) {
        String msg = saludoService.saludar(nombre);
        // Respuesta JSON “manual” (para simplicidad). Puedes usar JSON-B/Jackson.
        String json = "{\"mensaje\":\"" + msg.replace("\"","\\\"") + "\"}";
        return Response.ok(json).build();
    }
}
```

### `AppConfig.java` (activa JAX‑RS)

```java
package com.ejemplo;

import jakarta.ws.rs.ApplicationPath;
import jakarta.ws.rs.core.Application;

/**
 * Habilita JAX-RS sin web.xml:
 * - Todos los recursos @Path quedarán bajo /api/*
 */
@ApplicationPath("/api")
public class AppConfig extends Application {
    // Intencionalmente vacío
}
```

---

## 5A) Compilar, desplegar y probar

**Compilar**
```bash
mvn clean package
```

**Desplegar**
- Opción 1: Copia `target/demo-ejb-jaxrs.war` a `WILDFLY_HOME/standalone/deployments/`
- Opción 2 (plugin):  
  ```bash
  mvn wildfly:deploy
  ```

**Probar en navegador o curl**
```
http://localhost:8080/demo-ejb-jaxrs/api/hello?nombre=Javier
```

```bash
curl "http://localhost:8080/demo-ejb-jaxrs/api/hello?nombre=Javier"
# {"mensaje":"Hola, Javier 👋"}
```

---

## 6A) (Opcional) DataSource y JPA/JDBC

### Añadir un DataSource por CLI (ejemplo con H2)

```bash
# Con WildFly encendido:
$ WILDFLY_HOME/bin/jboss-cli.sh --connect

# Agregar driver H2 como módulo (ajusta ruta a tu jar)
module add --name=com.h2database.h2 --resources=/ruta/h2-2.2.224.jar --dependencies=javax.api,javax.transaction.api

# Registrar driver JDBC
/subsystem=datasources/jdbc-driver=h2:add(driver-name=h2, driver-module-name=com.h2database.h2)

# Crear DataSource (memoria)
data-source add --name=MyDS --jndi-name=java:/jdbc/MyDS \
  --driver-name=h2 --connection-url=jdbc:h2:mem:test;DB_CLOSE_DELAY=-1 \
  --user-name=sa --password=sa --use-ccm=true
:reload
```

**Inyección en EJB (ejemplo)**

```java
import jakarta.annotation.Resource;
import javax.sql.DataSource;

@Stateless
public class RepoService {
    @Resource(lookup="java:/jdbc/MyDS")
    private DataSource ds;

    // Usa ds.getConnection() para JDBC, o configura JPA con persistence.xml
}
```

> **Tip**: Para JPA, agrega `persistence.xml` en `META-INF` y usa `@PersistenceContext` para inyectar `EntityManager`.

---

# RUTA B) Spring MVC `@Controller` desplegado en WildFly llamando a EJB

### ¿Cuándo conviene?
- Tu equipo ya usa **Spring MVC** y necesitas **desplegar en WildFly**.  
- Quieres mantener **EJB** para ciertas capacidades (transaccionalidad, timers, seguridad JAAS/ELYTRON, etc.) y llamarlos desde Spring.

> **Nota**: Spring MVC **no reemplaza** a EJB; aquí conviven. El acceso se hace por **JNDI** o expone el EJB vía REST/JAX‑RS y el `@Controller` lo consume.

---

## 2B) `pom.xml` con Spring Web MVC

```xml
<dependencies>
  <!-- APIs Jakarta provistas por WildFly -->
  <dependency>
    <groupId>jakarta.platform</groupId>
    <artifactId>jakarta.jakartaee-api</artifactId>
    <version>${jakartaee.version}</version>
    <scope>provided</scope>
  </dependency>

  <!-- Spring MVC -->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>6.1.11</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.1.11</version>
  </dependency>
</dependencies>
```

> **Claves**  
> - Jakarta EE en `provided`.  
> - Spring en *compile/runtime* porque lo empaquetas dentro del `.war`.

---

## 3B) Inicialización sin `web.xml` (Java config)

### `WebAppInitializer.java`

```java
package com.ejemplo.spring;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

/**
 * Reemplaza web.xml. Registra el DispatcherServlet (Spring MVC) y su contexto.
 */
public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

  @Override
  protected Class<?>[] getRootConfigClasses() {
    // Beans raíz (servicios, seguridad, etc.)
    return new Class<?>[]{ SpringConfig.class };
  }

  @Override
  protected Class<?>[] getServletConfigClasses() {
    // Config web específica (si separas). Con null, usamos solo RootConfig.
    return null;
  }

  @Override
  protected String[] getServletMappings() {
    // Mapea el DispatcherServlet a "/"
    return new String[]{ "/" };
  }
}
```

### `SpringConfig.java`

```java
package com.ejemplo.spring;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

/**
 * Habilita Spring MVC y escaneo de componentes.
 */
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = {"com.ejemplo.spring"})
public class SpringConfig { }
```

---

## 4B) EJB (igual a la ruta A) y **Controller** Spring que invoca EJB por JNDI

> **Importante**: Para lookups “tipados” crea una **interfaz local** del EJB (p.ej. `SaludoServiceLocal`) y usa ese FQN en el JNDI.

### `SaludoServiceLocal.java` (interfaz del EJB)

```java
package com.ejemplo.control;

import jakarta.ejb.Local;

@Local
public interface SaludoServiceLocal {
    String saludar(String nombre);
}
```

### `SaludoService.java` (implementación EJB)

```java
package com.ejemplo.control;

import jakarta.ejb.Stateless;

@Stateless
public class SaludoService implements SaludoServiceLocal {
    @Override
    public String saludar(String nombre) {
        String n = (nombre == null || nombre.isBlank()) ? "mundo" : nombre.trim();
        return "Hola, " + n + " 👋";
    }
}
```

### `HelloController.java` (Spring MVC `@Controller`)

```java
package com.ejemplo.spring;

import com.ejemplo.control.SaludoServiceLocal;
import jakarta.naming.InitialContext;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

/**
 * Controller Spring. Realiza lookup JNDI al EJB.
 * Ajusta el nombre JNDI a tu paquete/artefacto.
 */
@Controller
@RequestMapping("/hello")
public class HelloController {

  private SaludoServiceLocal lookup() throws Exception {
    // Patrón de nombre típico:
    // java:global/<EAR|WAR>/<module>/<BeanSimpleName>!<FQN-Interfaz>
    return (SaludoServiceLocal) new InitialContext().lookup(
      "java:global/demo-ejb-jaxrs/demo-ejb-jaxrs/SaludoService!com.ejemplo.control.SaludoServiceLocal"
    );
  }

  @GetMapping
  @ResponseBody
  public ResponseEntity<String> hello(@RequestParam(required=false) String nombre) throws Exception {
    String msg = lookup().saludar(nombre);
    String json = "{\"mensaje\":\"" + msg.replace("\"","\\\"") + "\"}";
    return ResponseEntity.ok(json);
  }
}
```

> **Dónde ver el JNDI real**: al desplegar, WildFly registra los nombres JNDI del EJB en el **server log**. Búscalos para copiar el exacto.

---

## 5B) Despliegue y prueba

**Compila y despliega** igual que en la ruta A.

**Prueba**
```
http://localhost:8080/<tu-war>/hello?nombre=Javier
```

---

# Comparativa rápida (A vs B)

| Aspecto | Ruta A (Jakarta EE puro) | Ruta B (Spring MVC + EJB) |
| --- | --- | --- |
| “Controller” | **JAX‑RS** (`@Path`) | **Spring MVC** (`@Controller`) |
| Complejidad | Más simple (menos deps) | Mayor (Spring + JNDI) |
| Capacidades EJB | Nativas | Nativas (se invocan por JNDI) |
| Curva de aprendizaje | Jakarta EE estándar | Spring + JNDI + Jakarta EE |
| Cuándo usar | Proyectos EE puros o ligeros | Equipos ya Spring MVC pero con EJB existentes |

---

# 7) Seguridad básica (pista rápida)

- **Jakarta EE/JAX‑RS**: agrega filtros o `@RolesAllowed` en EJB y configura realms/roles en WildFly.  
- **Spring MVC**: añade Spring Security y configura un `SecurityFilterChain`.  
- **WildFly Elytron**: define realms, realms de aplicación y `http-authentication-factory`.

---

# 8) Troubleshooting

**1. `ClassNotFoundException` de APIs Jakarta o Spring**
- Verifica **versiones compatibles** (`jakarta.jakartaee-api` en `provided`).  
- No empaquetes el **server API** (evita duplicados).

**2. `NameNotFoundException` en lookup JNDI**
- Revisa el **log** de WildFly y copia el **nombre JNDI exacto** del EJB.  
- Confirma el **nombre del módulo/artefacto** y el **FQN** de la interfaz local/remota.

**3. `HTTP 404` en endpoints**
- ¿El contexto correcto? `http://localhost:8080/<artifactId>/...`  
- ¿`@ApplicationPath` activo para JAX‑RS?  
- En Spring MVC, ¿`DispatcherServlet` mapeado a `/`?

**4. Conflictos de JSON**
- Si usas JSON‑B (Jakarta) o Jackson (Spring), evita **dos providers** en el mismo WAR.  
- Mantén una sola estrategia por ruta o excluye uno si te duplica providers.

**5. Errores con DataSource**
- Driver JDBC como **módulo** y **jdbc-driver** agregado.  
- DataSource creado con **jndi-name** correcto (`java:/jdbc/…`).  
- Verifica credenciales y cadenas de conexión.

---

# 9) Variantes y buenas prácticas

- **Capas**: `boundary` (exposición), `control` (servicios/EJB), `entity` (JPA), `infrastructure` (config, datasources, clients).  
- **DTO/Mapper**: separa entidades JPA de objetos expuestos en REST. Usa MapStruct si prefieres.  
- **Validación**: `jakarta.validation` con `@NotNull`, `@Size`, etc.  
- **Transacciones**: por defecto CMT en EJB; usa `@TransactionAttribute` cuando necesites granularidad.  
- **Pruebas**: considera **Arquillian** para pruebas de contenedor, o **RestAssured** para endpoints.

---

# 10) Cheatsheet de comandos

```bash
# Arrancar WildFly
$ WILDFLY_HOME/bin/standalone.sh

# Empaquetar el WAR
$ mvn clean package

# Desplegar por plugin
$ mvn wildfly:deploy

# Re-desplegar
$ mvn wildfly:redeploy

# Desinstalar
$ mvn wildfly:undeploy

# CLI (conectar)
$ WILDFLY_HOME/bin/jboss-cli.sh --connect
```

---

## 11) Mini‑diagrama (texto)

```
[Cliente HTTP] → [JAX-RS @Path] → [EJB @Stateless] → [JPA/JDBC/DataSource]

[Cliente HTTP] → [Spring @Controller] → (JNDI) → [EJB @Stateless] → [JPA/JDBC/DataSource]
```

---

## 12) Checklist final

- [ ] Java 17 o 21 y Maven instalados  
- [ ] WildFly ejecutando en `localhost:8080` y `9990`  
- [ ] `pom.xml` con `jakarta.jakartaee-api` (scope `provided`)  
- [ ] RUTA A: `@ApplicationPath("/api")`, recurso `@Path`, EJB `@Stateless`  
- [ ] RUTA B: `WebAppInitializer`, `@EnableWebMvc`, `@Controller`, lookup JNDI al EJB  
- [ ] Desplegar WAR → probar endpoint con navegador/curl  
- [ ] (Opcional) DataSource configurado y probado

---

### Licencia
Este contenido es de uso libre para fines educativos y proyectos personales dentro de tu equipo.

> ¿Quieres que te entregue este proyecto **base** como esqueleto Maven (WAR) listo para compilar y desplegar? Pídemelo y te genero el ZIP con la estructura completa.
