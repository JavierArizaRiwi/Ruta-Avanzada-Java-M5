
# 📘 Interfaces en Java — Guía Completa con Proyecto de Ejemplo

> **Código módulo**:
> **Objetivo**: Entender **qué son las interfaces**, **por qué** usarlas y **cómo** aplicarlas en un proyecto real (con JDBC o en memoria), favoreciendo **desacoplamiento**, **polimorfismo** y **testabilidad**.

---

## 1) Conceptos clave

### ¿Qué es una interfaz?
Una **interfaz** es un contrato que especifica **qué** debe hacer una clase, **no cómo**. Contiene **firmas de métodos** (y opcionalmente constantes) que las clases **implementan** con `implements`.

```java
public interface Operacion {
    double ejecutar(double a, double b);
}
```

### Ventajas
- **Polimorfismo**: múltiples implementaciones detrás del mismo tipo.
- **Desacoplamiento**: se depende de **abstracciones** y no de concreciones.
- **Diseño modular**: facilita cambios y substitución de componentes.
- **Testabilidad**: puedes **simular** dependencias (mocks/fakes) fácilmente.

### Características modernas (Java 8+)
- **Métodos `default`**: permiten agregar comportamiento por defecto sin romper implementaciones existentes.
- **Métodos `static`**: utilidades relacionadas con la interfaz.
- **Interfaces funcionales**: una sola abstracción (`@FunctionalInterface`) → compatibles con **lambdas**.

```java
@FunctionalInterface
public interface Transform<T> {
    T apply(T in);
    // default y static también son válidos aquí
}
```

### Herencia múltiple de tipos
Las clases **no** soportan herencia múltiple, pero **sí** pueden **implementar varias interfaces**:
```java
public class MiServicio implements AutoCloseable, Runnable { /* ... */ }
```

---

## 2) Interfaces vs Clases Abstractas (rápido)

| Aspecto | Interfaz | Clase abstracta |
|---|---|---|
| Estado (campos) | Solo constantes `public static final` | Puede tener estado (campos) |
| Métodos con implementación | `default`/`static` (limitado) | Métodos concretos normales |
| Herencia | Múltiples interfaces | Una sola súper-clase |
| Cuándo usar | Contratos puros / roles | Plantillas parciales con estado compartido |

---

## 3) Buenas prácticas
- **Programar contra interfaces**, no contra implementaciones.
- Preferir **métodos pequeños y enfocados** en la interfaz (ISP: Interface Segregation Principle).
- Mantener la interfaz **estable**; evolucionar con **métodos `default`** si es necesario.
- Nombrar **con sustantivos** (p.ej., `RepositorioUsuario`) o adjetivar el rol (`Notificador`).
- Evitar “**Dios interfaces**” (demasiados métodos en una sola interfaz).

---

## 4) Proyecto de ejemplo — Mini Inventario (Consola + Repositorio conmutables)

### Contexto
Construimos un **módulo de inventario** con:
- Una **interfaz de repositorio** (`ProductoRepository`).
- **Dos implementaciones**: en memoria y JDBC.
- Un **servicio de dominio** que **depende de la interfaz** (no conoce la tecnología).
- Un **controlador** (consola) que usa el servicio.
- Un **fake** para pruebas rápidas sin BD.

### Estructura de carpetas
```
interfaces-inventario/
 ├─ src/
 │   ├─ domain/
 │   │   ├─ Producto.java
 │   │   ├─ ProductoRepository.java      (INTERFAZ)
 │   │   └─ InventarioService.java
 │   ├─ infra/
 │   │   ├─ InMemoryProductoRepository.java
 │   │   └─ JdbcProductoRepository.java
 │   └─ app/
 │       └─ Main.java
 └─ README.md
```

### 4.1 Dominio

**Producto.java**
```java
package domain;

public class Producto {
    private Integer id;
    private String nombre;
    private int stock;
    private double precio;

    public Producto(Integer id, String nombre, int stock, double precio) {
        this.id = id;
        this.nombre = nombre;
        this.stock = stock;
        this.precio = precio;
    }

    public Integer getId() { return id; }
    public String getNombre() { return nombre; }
    public int getStock() { return stock; }
    public double getPrecio() { return precio; }

    public void agregarStock(int unidades) { this.stock += unidades; }
    public void quitarStock(int unidades) { this.stock -= unidades; }
}
```

**ProductoRepository.java (INTERFAZ)**
```java
package domain;

import java.util.List;
import java.util.Optional;

public interface ProductoRepository {
    Producto guardar(Producto p);
    Optional<Producto> porId(int id);
    List<Producto> listar();
    boolean eliminar(int id);

    default boolean existe(int id) {
        return porId(id).isPresent();
    }
}
```

**InventarioService.java**
```java
package domain;

import java.util.List;

public class InventarioService {
    private final ProductoRepository repo;

    // Inversión de Dependencia: inyectamos la INTERFAZ
    public InventarioService(ProductoRepository repo) {
        this.repo = repo;
    }

    public Producto crearProducto(String nombre, int stock, double precio) {
        // Validaciones de negocio
        if (stock < 0 || precio < 0) {
            throw new IllegalArgumentException("Stock y precio deben ser no negativos");
        }
        // id se delega al repo (p.ej. autoincremental)
        Producto p = new Producto(null, nombre, stock, precio);
        return repo.guardar(p);
    }

    public List<Producto> listar() { return repo.listar(); }

    public void reponer(int id, int unidades) {
        Producto p = repo.porId(id).orElseThrow(() -> new IllegalArgumentException("No existe producto " + id));
        p.agregarStock(unidades);
        repo.guardar(p);
    }

    public void vender(int id, int unidades) {
        Producto p = repo.porId(id).orElseThrow(() -> new IllegalArgumentException("No existe producto " + id));
        if (p.getStock() < unidades) throw new IllegalStateException("Stock insuficiente");
        p.quitarStock(unidades);
        repo.guardar(p);
    }

    public boolean eliminar(int id) { return repo.eliminar(id); }
}
```

### 4.2 Infraestructura (Implementaciones de la interfaz)

**InMemoryProductoRepository.java**
```java
package infra;

import domain.Producto;
import domain.ProductoRepository;

import java.util.*;
import java.util.concurrent.atomic.AtomicInteger;

public class InMemoryProductoRepository implements ProductoRepository {
    private final Map<Integer, Producto> data = new HashMap<>();
    private final AtomicInteger seq = new AtomicInteger(0);

    @Override
    public Producto guardar(Producto p) {
        Integer id = p.getId();
        if (id == null) {
            id = seq.incrementAndGet();
            p = new Producto(id, p.getNombre(), p.getStock(), p.getPrecio());
        }
        data.put(id, p);
        return p;
    }

    @Override
    public Optional<Producto> porId(int id) {
        return Optional.ofNullable(data.get(id));
    }

    @Override
    public List<Producto> listar() {
        return new ArrayList<>(data.values());
    }

    @Override
    public boolean eliminar(int id) {
        return data.remove(id) != null;
    }
}
```

**JdbcProductoRepository.java**
```java
package infra;

import domain.Producto;
import domain.ProductoRepository;

import java.sql.*;
import java.util.*;

public class JdbcProductoRepository implements ProductoRepository {
    private final Connection conn;

    public JdbcProductoRepository(Connection conn) {
        this.conn = conn;
    }

    @Override
    public Producto guardar(Producto p) {
        try {
            if (p.getId() == null) {
                String sql = "INSERT INTO producto(nombre, stock, precio) VALUES (?, ?, ?)";
                try (PreparedStatement ps = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
                    ps.setString(1, p.getNombre());
                    ps.setInt(2, p.getStock());
                    ps.setDouble(3, p.getPrecio());
                    ps.executeUpdate();
                    try (ResultSet rs = ps.getGeneratedKeys()) {
                        if (rs.next()) {
                            int id = rs.getInt(1);
                            return new Producto(id, p.getNombre(), p.getStock(), p.getPrecio());
                        }
                    }
                }
            } else {
                String sql = "UPDATE producto SET nombre=?, stock=?, precio=? WHERE id=?";
                try (PreparedStatement ps = conn.prepareStatement(sql)) {
                    ps.setString(1, p.getNombre());
                    ps.setInt(2, p.getStock());
                    ps.setDouble(3, p.getPrecio());
                    ps.setInt(4, p.getId());
                    ps.executeUpdate();
                    return p;
                }
            }
        } catch (SQLException e) {
            throw new RuntimeException("Error guardando producto", e);
        }
        throw new RuntimeException("No se pudo guardar el producto");
    }

    @Override
    public Optional<Producto> porId(int id) {
        try {
            String sql = "SELECT id, nombre, stock, precio FROM producto WHERE id=?";
            try (PreparedStatement ps = conn.prepareStatement(sql)) {
                ps.setInt(1, id);
                try (ResultSet rs = ps.executeQuery()) {
                    if (rs.next()) {
                        return Optional.of(new Producto(
                            rs.getInt("id"),
                            rs.getString("nombre"),
                            rs.getInt("stock"),
                            rs.getDouble("precio")
                        ));
                    }
                }
            }
            return Optional.empty();
        } catch (SQLException e) {
            throw new RuntimeException("Error consultando producto", e);
        }
    }

    @Override
    public List<Producto> listar() {
        List<Producto> out = new ArrayList<>();
        try {
            String sql = "SELECT id, nombre, stock, precio FROM producto ORDER BY id";
            try (PreparedStatement ps = conn.prepareStatement(sql);
                 ResultSet rs = ps.executeQuery()) {
                while (rs.next()) {
                    out.add(new Producto(
                        rs.getInt("id"),
                        rs.getString("nombre"),
                        rs.getInt("stock"),
                        rs.getDouble("precio")
                    ));
                }
            }
            return out;
        } catch (SQLException e) {
            throw new RuntimeException("Error listando productos", e);
        }
    }

    @Override
    public boolean eliminar(int id) {
        try {
            String sql = "DELETE FROM producto WHERE id=?";
            try (PreparedStatement ps = conn.prepareStatement(sql)) {
                ps.setInt(1, id);
                return ps.executeUpdate() > 0;
            }
        } catch (SQLException e) {
            throw new RuntimeException("Error eliminando producto", e);
        }
    }
}
```

### 4.3 Aplicación (Consola)

**Main.java**
```java
package app;

import domain.InventarioService;
import infra.InMemoryProductoRepository;
// import infra.JdbcProductoRepository;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) throws Exception {
        // Cambia la implementación sin tocar el servicio (¡polimorfismo!)
        var repo = new InMemoryProductoRepository();
        // var conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/tienda", "root", "pass");
        // var repo = new JdbcProductoRepository(conn);

        var service = new InventarioService(repo);
        var sc = new Scanner(System.in);

        System.out.println("=== Inventario ===");
        while (true) {
            System.out.println("1) Crear  2) Listar  3) Reponer  4) Vender  5) Eliminar  0) Salir");
            int op = Integer.parseInt(sc.nextLine());
            if (op == 0) break;
            try {
                switch (op) {
                    case 1 -> {
                        System.out.print("Nombre: "); var n = sc.nextLine();
                        System.out.print("Stock: ");  var s = Integer.parseInt(sc.nextLine());
                        System.out.print("Precio: "); var p = Double.parseDouble(sc.nextLine());
                        var creado = service.crearProducto(n, s, p);
                        System.out.println("Creado ID=" + creado.getId());
                    }
                    case 2 -> service.listar().forEach(x ->
                        System.out.println(x.getId() + " | " + x.getNombre() + " | " + x.getStock() + " | " + x.getPrecio()));
                    case 3 -> {
                        System.out.print("ID: "); var id = Integer.parseInt(sc.nextLine());
                        System.out.print("Unidades: "); var u = Integer.parseInt(sc.nextLine());
                        service.reponer(id, u);
                        System.out.println("Stock actualizado.");
                    }
                    case 4 -> {
                        System.out.print("ID: "); var id = Integer.parseInt(sc.nextLine());
                        System.out.print("Unidades: "); var u = Integer.parseInt(sc.nextLine());
                        service.vender(id, u);
                        System.out.println("Venta registrada.");
                    }
                    case 5 -> {
                        System.out.print("ID: "); var id = Integer.parseInt(sc.nextLine());
                        System.out.println(service.eliminar(id) ? "Eliminado." : "No existe.");
                    }
                    default -> System.out.println("Opción inválida.");
                }
            } catch (Exception e) {
                System.out.println("Error: " + e.getMessage());
            }
        }
        sc.close();
    }
}
```

---

## 5) Interfaces + Patrones (Uso práctico)

- **Strategy**: estrategias de precio o descuento intercambiables.
```java
public interface PoliticaPrecio { double calcular(double base); }

public class PrecioNormal implements PoliticaPrecio {
    public double calcular(double base) { return base; }
}

public class PrecioDescuento implements PoliticaPrecio {
    private final double porcentaje;
    public PrecioDescuento(double porcentaje) { this.porcentaje = porcentaje; }
    public double calcular(double base) { return base * (1 - porcentaje); }
}
```

- **Repository**: interfaces para persistencia → conmutar BD/memoria.
- **Observer**: notificar a subsistemas (p.ej., alertas de bajo stock) mediante una interfaz `SuscriptorStock`.

---

## 6) Pruebas con Fakes/Mocks (sin BD)
```java
class FakeRepo implements domain.ProductoRepository {
    // Implementación mínima de pruebas...
}
/*
Ventaja: al inyectar la INTERFAZ, puedes pasar FakeRepo al servicio
para probar reglas de negocio sin levantar una base de datos real.
*/
```

---

## 7) Ejercicios propuestos
1. Agrega `buscarPorNombre(String q)` a la **interfaz** y a ambas implementaciones.
2. Implementa **Observer**: cuando el stock baje de 5, notificar por consola.
3. Cambia la política de precios usando **Strategy** (normal vs descuento).
4. Escribe una prueba unitaria del `InventarioService` usando un **FakeRepo**.
5. Cambia la app para usar JDBC sin cambiar **nada** en `InventarioService`.

---

## 8) SQL de referencia (JDBC)
```sql
CREATE TABLE producto (
  id INT PRIMARY KEY AUTO_INCREMENT,
  nombre VARCHAR(80) NOT NULL,
  stock INT NOT NULL,
  precio DECIMAL(10,2) NOT NULL
);
```

---

## 9) Resumen
- Las **interfaces** permiten **contratos claros**, **polimorfismo** y **desacoplamiento**.
- Combinadas con patrones como **Repository**, **Strategy** u **Observer**, facilitan diseño limpio.
- Inyectar **interfaces** hace tu código **probable**, **sustituible** y **listo** para crecer (memoria → JDBC → REST).
