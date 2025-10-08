# DTO, Mapper y Otras Piezas en Arquitectura por Capas (Java)

## Objetivo
Este documento explica, con enfoque práctico, las diferencias entre **DTO**, **Mapper** y otras piezas comunes en aplicaciones Java con **arquitectura por capas**. Incluye tablas comparativas, ejemplos de código y buenas prácticas para aplicar en proyectos como `academic` o una mini-tienda.

---

## 1. Mapa general de una app por capas
```
UI (Swing/REST) 
  → DTO Request
  → Service (casos de uso)
      → Mapper: DTO → Entidad
      → Repository (persistencia)
      ← Entidad (desde la BD)
      ← Mapper: Entidad → DTO Response
  ← DTO Response
```
- **UI** solo habla con **Services** usando **DTOs**.
- **Service** orquesta reglas y persistencia, apoyándose en **Mappers**.
- **Repository/DAO** lee/escribe **entidades de dominio**.

---

## 2. Definiciones breves

| Concepto | ¿Qué es? | ¿Para qué se usa? | ¿Dónde vive? |
|---|---|---|---|
| **Entidad / Modelo de Dominio** | Objeto que representa el **negocio** con reglas e invariantes. | Encapsular comportamiento y consistencia del dominio. | `domain` |
| **DTO (Data Transfer Object)** | Objeto **solo de datos** para **entrar/salir** de casos de uso o APIs. | Transportar información sin exponer el dominio. | `dto` (borde: UI/API ↔ Service) |
| **ViewModel** | Variante de DTO pensada para **presentación**. | Adaptar datos a lo que la UI necesita mostrar. | `ui` |
| **VO (Value Object)** | Tipo **inmutable** con semántica (por ej. `Email`, `Money`). | Validación y significado fuerte de valores. | `domain` |
| **Repository/DAO** | **Contrato** e implementación de acceso a datos. | Aislar la tecnología (JDBC/JPA) del negocio. | `repository` (interfaces) + `infra` (impl) |
| **Service** | **Contrato** de casos de uso + orquestación. | Coordinar validaciones, repos y transacciones. | `service` (interfaces) + `service.impl` |
| **Mapper / Converter / Assembler** | Utilidades para **transformar** objetos. | DTO ↔ Entidad, uniones de fuentes, conversiones de tipo. | `mapper` (o capa de aplicación) |
| **Adapter** | Patrón para compatibilizar interfaces. | Integrarse con SDKs/servicios externos. | `infra/adapters` |

---

## 3. DTO vs Entidad vs VO — diferencias clave

| Característica | DTO | Entidad / Modelo | VO (Value Object) |
|---|---|---|---|
| Propósito | Transporte de datos | Reglas de negocio, invariantes | Semántica de un valor, inmutable |
| Lógica de negocio | No | Sí | Validación del propio valor |
| Persistencia (anotaciones) | No | Puede tener (JPA) | No |
| Mutabilidad | Normalmente mutable | Depende del diseño | Inmutable |
| Alcance | Bordes (UI/API/Service) | Núcleo del dominio | Núcleo del dominio |

Regla de oro: **no expongas entidades del dominio** hacia fuera; usa **DTOs**.

---

## 4. Mapper vs Converter vs Assembler vs Adapter

| Tipo | Qué hace | Cuándo usar |
|---|---|---|
| **Mapper** | Copia campos entre objetos (DTO ↔ Entidad). | Conversión 1–1; puede automatizarse con MapStruct. |
| **Converter** | Conversión de **tipo** (String→LocalDate, int→BigDecimal). | Cambios de representación de datos. |
| **Assembler** | Construye un DTO **a partir de varias fuentes**. | Cuando un DTO reúne datos de varios agregados. |
| **Adapter** | Hace compatible una interfaz con otra. | Integraciones con SDKs/APIs externas. |

---

## 5. Ejemplo práctico

### 5.1 Entidad (dominio)
```java
// domain/Producto.java
public class Producto {
    private Long id;
    private String nombre;
    private double precio;
    private int stock;

    public Producto(Long id, String nombre, double precio, int stock) {
        if (nombre == null || nombre.isBlank()) throw new IllegalArgumentException("Nombre vacío");
        if (precio <= 0) throw new IllegalArgumentException("Precio inválido");
        if (stock < 0) throw new IllegalArgumentException("Stock inválido");
        this.id = id; this.nombre = nombre.trim(); this.precio = precio; this.stock = stock;
    }
    // getters/setters
}
```

### 5.2 DTOs (borde)
```java
// dto/ProductoRequest.java
public class ProductoRequest {
    public String nombre;
    public double precio;
    public int stock;
}

// dto/ProductoResponse.java
public class ProductoResponse {
    public Long id;
    public String nombre;
    public double precio;
    public int stock;
}
```

### 5.3 Mapper
```java
// mapper/ProductoMapper.java
public final class ProductoMapper {
    private ProductoMapper() {}

    public static Producto toDomain(ProductoRequest r) {
        return new Producto(null, r.nombre, r.precio, r.stock);
    }
    public static ProductoResponse toResponse(Producto p) {
        ProductoResponse dto = new ProductoResponse();
        dto.id = p.getId();
        dto.nombre = p.getNombre();
        dto.precio = p.getPrecio();
        dto.stock = p.getStock();
        return dto;
    }
}
```

### 5.4 Repository y Service
```java
// repository/ProductoRepository.java
public interface ProductoRepository {
    Producto save(Producto p);
    Optional<Producto> findById(Long id);
    List<Producto> findAll();
    void deleteById(Long id);
}
```

```java
// service/InventarioService.java
public interface InventarioService {
    ProductoResponse agregar(ProductoRequest req);
}
```

```java
// service/impl/InventarioServiceImpl.java
public class InventarioServiceImpl implements InventarioService {
    private final ProductoRepository repo;
    public InventarioServiceImpl(ProductoRepository repo){ this.repo = repo; }

    @Override public ProductoResponse agregar(ProductoRequest req) {
        // Validaciones de caso de uso adicionales (si aplica)
        Producto p = ProductoMapper.toDomain(req);      // validaciones del constructor
        Producto guardado = repo.save(p);
        return ProductoMapper.toResponse(guardado);
    }
}
```

---

## 6. Buenas prácticas

1. **DTOs por caso de uso:** `CreateProductoRequest` ≠ `UpdateProductoRequest`.
2. **No mezclar capas:** Repository devuelve **entidades**, no DTOs.
3. **Validación en el lugar correcto:**
   - Formato/campos requeridos → borde (DTO + Bean Validation si usas Jakarta Validation).
   - Invariantes de negocio → **dominio** (constructor/métodos).
   - Reglas de caso de uso → **service**.
4. **Mapper centralizado:** evita duplicar mapeos dentro de controladores o servicios.
5. **Automatiza mapeos repetitivos:** considera **MapStruct** para proyectos grandes.
6. **Versiona DTOs** en APIs públicas (v1, v2) para no romper clientes.
7. **VOs para conceptos con reglas propias:** `Email`, `Telefono`, `Money`.
8. **Tests específicos:** prueba mappers (con datos límite) y reglas de dominio.

---

## 7. Antipatrones a evitar

- Exponer **entidades** en endpoints o UI.
- Cargar **lógica de negocio en DTOs** (deben ser simples contenedores de datos).
- Mapeos “rápidos” duplicados en controladores/servicios.
- Repositorios devolviendo DTOs o la UI hablando directo con JDBC.
- DTOs “omnívoros” con todos los campos de todo (rompen el principio de mínimo necesario).

---

## 8. Checklist rápido para tu proyecto

- [ ] UI solo usa **DTOs** y **Services (interfaces)**.
- [ ] Services **no** conocen JDBC ni SQL.
- [ ] Repositories trabajan con **entidades de dominio**.
- [ ] **Mappers** centralizados para DTO ↔ Dominio.
- [ ] **VOs** definidos para valores con reglas (Email, Money).
- [ ] **Tests** para mappers y reglas de dominio.
- [ ] **DTOs versionados** si hay APIs públicas.

---

## 9. Glosario mínimo

- **DTO**: Objeto de transferencia de datos, sin lógica de negocio.
- **Entidad/Modelo**: Representa el negocio; contiene reglas e invariantes.
- **VO**: Objeto de valor inmutable con semántica específica.
- **Mapper**: Conversión entre modelos (ej. DTO ↔ Entidad).
- **Repository/DAO**: Contrato de persistencia independiente de la tecnología.
- **Service**: Orquestación de casos de uso y reglas de aplicación.

---

## 10. Conclusión
Separar **DTOs**, **Entidades**, **Mappers** y el resto de componentes te permite construir sistemas **mantenibles, testeables y escalables**. Esta organización favorece el reemplazo tecnológico (JDBC → JPA), la evolución de contratos (DTOs) y una clara asignación de responsabilidades por capa.