# Puente a Spring Boot: lo mínimo que debes dominar antes de empezar

## Objetivo
Este complemento conecta el bloque de Java SE con Spring Boot. La idea es que no llegues a Spring Boot viendo solo anotaciones sueltas, sino entendiendo primero qué problema resuelve, cómo encaja con la arquitectura por capas y qué piezas de Java debes reconocer desde el día uno.

## 1. Qué debe quedar claro antes de Spring Boot
Antes de empezar con Spring Boot conviene dominar estos fundamentos:
- Clases, objetos, interfaces y abstracción.
- Excepciones y validaciones.
- Colecciones y `Optional`.
- JDBC básico y arquitectura por capas.
- Separación entre dominio, servicio, repositorio y presentación.
- JSON y conceptos de API REST.
- Maven y estructura de proyecto.

Si una persona no entiende esto, Spring Boot se aprende de memoria pero no se comprende.

## 2. Cómo cambia el estilo con Spring Boot
Spring Boot no cambia Java, cambia la forma de montar aplicaciones.

### Java SE
- Ejecutas una clase `main`.
- Construyes todo a mano.
- Configuras dependencias, wiring y ciclos de vida de forma manual.
- Usas `Scanner`, `JOptionPane`, Swing o consola.

### Spring Boot
- Arranca una aplicación con un contenedor embebido.
- Inyecta dependencias automáticamente.
- Simplifica configuración con `application.properties` o `application.yml`.
- Expone endpoints REST con `@RestController`.
- Gestiona transacciones, validaciones y persistencia con menos código repetitivo.

## 3. Mapeo conceptual: Java SE → Spring Boot
| Java SE | Spring Boot |
|---|---|
| `main()` manual | `@SpringBootApplication` |
| `new Servicio()` | Inyección con `@Autowired` o constructor |
| `DAO` JDBC manual | `JpaRepository` o `JdbcTemplate` |
| `Controller` de consola / Swing | `@RestController` |
| Validaciones manuales | Bean Validation (`@Valid`, `@NotNull`, `@Size`) |
| Manejo de errores con `try/catch` en UI | `@ControllerAdvice` y manejadores globales |
| Configuración en código | `application.properties` |

## 4. Qué debes saber de Spring Boot para no perderte
### 4.1. Dependencias iniciales
Lo primero que verás es el `pom.xml` con starters:
- `spring-boot-starter-web`
- `spring-boot-starter-validation`
- `spring-boot-starter-data-jpa`
- driver de base de datos
- `spring-boot-starter-test`

### 4.2. Estructura típica
```text
com.example.app
├─ controller
├─ service
├─ repository
├─ model
├─ dto
├─ exception
└─ AppApplication.java
```

### 4.3. Anotaciones que debes entender bien
- `@SpringBootApplication`
- `@RestController`
- `@RequestMapping`
- `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`
- `@Service`
- `@Repository`
- `@Entity`
- `@Id`
- `@GeneratedValue`
- `@Transactional`
- `@Valid`
- `@ControllerAdvice`

## 5. Qué temas de este curso sirven como base directa
Estos bloques ya te dejan listo para Spring Boot si los entiendes bien:
- Interfaces y contratos.
- Arquitectura por capas.
- DTO, Mapper y Service.
- Manejo de excepciones personalizadas.
- JDBC y acceso a datos.
- Maven y empaquetado.
- Validación de datos.

## 6. Qué suele costar al pasar a Spring Boot
### 6.1. Inyección de dependencias
En Java SE tú construyes objetos manualmente. En Spring, el contenedor decide cómo crear e inyectar objetos.

### 6.2. Manejo de peticiones HTTP
Un `@RestController` no se usa como una clase normal. Cada método representa una ruta y un verbo HTTP.

### 6.3. Persistencia con JPA
Ya no escribes todo el CRUD con SQL manual si no hace falta. Spring Data abstrae gran parte del acceso a datos.

### 6.4. Excepciones centralizadas
En vez de capturar todo en cada clase, normalmente se centraliza el manejo con controladores globales.

## 7. Ruta recomendada para entrar a Spring Boot
1. Crear un proyecto con Spring Initializr.
2. Añadir web, validation, JPA y base de datos.
3. Hacer un CRUD sencillo de una sola entidad.
4. Separar controller, service, repository y model.
5. Añadir DTOs y mappers.
6. Centralizar excepciones.
7. Agregar pruebas.
8. Pasar a relaciones entre entidades y validaciones más reales.

## 8. Conclusión
Spring Boot será mucho más fácil si antes entiendes el patrón de capas y el flujo completo de una aplicación Java. El objetivo del puente no es enseñarte Spring Boot completo aquí, sino dejarte listo para que el salto sea natural y no una ruptura brusca.