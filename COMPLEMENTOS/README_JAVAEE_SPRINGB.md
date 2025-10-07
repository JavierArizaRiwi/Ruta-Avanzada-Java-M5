# Comparativa y Guía Práctica: Java SE, Java EE (JBoss/WildFly) y Spring Boot

---

## 1. Diferencia General entre Java SE y Java EE

| Característica | **Java SE (Standard Edition)** | **Java EE (Enterprise Edition / Jakarta EE)** |
|----------------|--------------------------------|---------------------------------------------|
| Enfoque | Aplicaciones de escritorio o consola | Aplicaciones empresariales (web, APIs, servicios) |
| Librerías | Core Java (collections, threads, JDBC, I/O) | Extensiones: Servlets, JSP, EJB, JPA, JAX-RS, CDI |
| Ejecución | JVM local | Servidor de aplicaciones (Tomcat, WildFly, Payara, GlassFish, etc.) |
| Dependencias | Manuales | Inyección de dependencias automática |
| Arquitectura | Monolítica o simple | Multicapa (web, negocio, persistencia) |

> **Java SE = código local**, **Java EE = código empresarial distribuido.**

---

##  2. Capas en una Aplicación Java EE

```
PRESENTACIÓN  →  CONTROL  →  LÓGICA  →  PERSISTENCIA  →  BD
(JSP/JSF)        (Servlet)   (EJB/CDI)     (JPA/Hibernate)  (MySQL)
```

Cada capa tiene su rol y utiliza diferentes componentes o anotaciones.

---

##  3. Servidor de Aplicaciones: JBoss / WildFly

### • ¿Qué hace un servidor de aplicaciones?

| Funcionalidad | Rol del servidor (JBoss/WildFly) |
|----------------|----------------------------------|
| Ciclo de vida de componentes | Gestiona creación y destrucción de beans |
| Inyección de dependencias | Reconoce `@EJB`, `@Inject`, `@PersistenceContext` |
| Seguridad | Autenticación, roles, JAAS |
| Transacciones | Manejo automático con `@Transactional` |
| Configuración | Pools de conexiones, despliegue WAR, JNDI |

**JBoss (ahora WildFly)** implementa todas las especificaciones Java EE.

### • Flujo típico de trabajo en Java EE
1. Crear proyecto *Dynamic Web Project* en Eclipse.
2. Definir `web.xml` y `persistence.xml`.
3. Implementar Servlets, EJB, JPA.
4. Empaquetar en `.war`.
5. Desplegar en JBoss (carpeta `standalone/deployments`).

---

##  4. Ejemplo de Proyecto Java EE con JBoss

### Estructura:
```
miapp/
 ├── src/main/java/
 │    ├── model/Usuario.java        (@Entity)
 │    ├── service/UsuarioService.java (@Stateless)
 │    └── web/UsuarioServlet.java   (@WebServlet)
 ├── src/main/webapp/
 │    ├── index.jsp
 │    └── WEB-INF/web.xml
 ├── META-INF/persistence.xml
 ├── beans.xml
 └── pom.xml
```

### Ejemplo de código:
```java
@Entity
public class Usuario {
    @Id
    @GeneratedValue
    private Long id;
    private String nombre;
}

@Stateless
public class UsuarioService {
    @PersistenceContext
    private EntityManager em;
    public void guardar(Usuario u) { em.persist(u); }
    public List<Usuario> listar() { return em.createQuery("FROM Usuario").getResultList(); }
}

@WebServlet("/usuarios")
public class UsuarioServlet extends HttpServlet {
    @EJB
    private UsuarioService servicio;
    protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.getWriter().println(servicio.listar());
    }
}
```

---

##  5. Anotaciones Importantes en Java EE

| Anotación | Uso |
|-------------|------|
| `@Stateless` | Declara un EJB sin estado |
| `@Stateful` | Declara un EJB con estado |
| `@EJB` | Inyecta un bean empresarial |
| `@Inject` | Inyección de dependencias con CDI |
| `@Named` | Expone una clase CDI a la capa de presentación |
| `@PersistenceContext` | Inyecta el `EntityManager` |
| `@Transactional` | Control de transacciones automáticas |

### Ejemplo de CDI (Context and Dependency Injection)
```java
@Named
public class ProductoService {
    public List<String> listar() {
        return List.of("Monitor", "Teclado");
    }
}

@WebServlet("/productos")
public class ProductoServlet extends HttpServlet {
    @Inject
    private ProductoService servicio;

    protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.getWriter().println(servicio.listar());
    }
}
```

---

## 6. Comparación con Spring Boot

| Característica | **Java EE (JBoss/WildFly)** | **Spring Boot** |
|------------------|------------------------------|-----------------|
| Ejecución | Despliegue en servidor externo (WAR) | Servidor embebido (JAR ejecutable) |
| Configuración | XML o `beans.xml` | `@SpringBootApplication` + `application.properties` |
| Inyección | `@EJB`, `@Inject` | `@Autowired` |
| Transacciones | `@Transactional` (JTA) | `@Transactional` (Spring Data) |
| Seguridad | JAAS | Spring Security |
| Despliegue | Manual | Automático (`mvn spring-boot:run`) |

### Equivalencias de anotaciones

| Capa | Java EE | Spring Boot |
|------|----------|-------------|
| Controlador | `@WebServlet` | `@RestController` |
| Servicio | `@Stateless`, `@Inject` | `@Service`, `@Autowired` |
| Repositorio | `EntityManager` | `@Repository`, `JpaRepository` |
| Configuración | `beans.xml` | `@Configuration`, `@Bean` |

### Ejemplo en Spring Boot
```java
@Service
public class ProductoService {
    public List<String> listar() {
        return List.of("Monitor", "Teclado");
    }
}

@RestController
@RequestMapping("/productos")
public class ProductoController {
    @Autowired
    private ProductoService servicio;

    @GetMapping
    public List<String> listar() {
        return servicio.listar();
    }
}
```

---

##  7. Migrar de Java EE (JBoss) a Spring Boot

### Reemplazos principales:
| De | A |
|----|---|
| `@EJB` | `@Service` + `@Autowired` |
| `@PersistenceContext` | `@Autowired EntityManager` |
| `web.xml` | Configuración automática |
| `.war` | `.jar` ejecutable |

### Ventajas de Spring Boot
- No necesita servidor externo.
- Configuración más sencilla.
- Mejor integración con Docker y CI/CD.
- Ecosistema moderno (Spring Security, Spring Data, Spring Cloud).

---

##  8. Recomendación para tu Team

| Etapa | Tema | Herramientas |
|--------|------|---------------|
| 1 | Java EE básico (Servlet + JSP) | Eclipse + Tomcat/WildFly |
| 2 | EJB y CDI (@EJB, @Inject, @Named) | Payara / WildFly |
| 3 | Persistencia (JPA con Hibernate) | MySQL + persistence.xml |
| 4 | Spring Boot modernización | Spring Initializr + IntelliJ |
| 5 | REST + Docker | Postman + Docker Compose |

---

##  9. Ejemplo Final: Comparativo Completo

### Java EE (con JBoss/WildFly)
```java
@Stateless
public class UsuarioService {
    @PersistenceContext
    private EntityManager em;

    public List<Usuario> listar() {
        return em.createQuery("SELECT u FROM Usuario u", Usuario.class).getResultList();
    }
}

@WebServlet("/usuarios")
public class UsuarioServlet extends HttpServlet {
    @EJB
    private UsuarioService servicio;

    protected void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        res.getWriter().println(servicio.listar());
    }
}
```

### Spring Boot (equivalente)
```java
@Service
public class UsuarioService {
    @Autowired
    private UsuarioRepository repo;

    public List<Usuario> listar() {
        return repo.findAll();
    }
}

@RestController
@RequestMapping("/usuarios")
public class UsuarioController {
    @Autowired
    private UsuarioService servicio;

    @GetMapping
    public List<Usuario> listar() {
        return servicio.listar();
    }
}
```

---

## Conclusión
- **Java EE + JBoss** es ideal para entornos corporativos legados.
- **Spring Boot** ofrece simplicidad, flexibilidad y portabilidad.
- Entender ambos te permite trabajar en sistemas antiguos y modernos sin problema.

> Si dominas Servlets, EJB, JPA y CDI en Java EE, y entiendes cómo se traducen a `@RestController`, `@Service`, `@Repository` en Spring Boot, ya estás listo para cubrir cualquier vacante enterprise moderna.