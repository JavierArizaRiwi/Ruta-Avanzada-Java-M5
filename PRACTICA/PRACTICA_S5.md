# Proyecto integrador — Registro y exportación de pedidos

- **Nivel:** cierre de Java SE y preparación para Spring Boot
- **Duración sugerida:** 12–18 horas
- **Versión base:** Java 21 LTS

## 1. Propósito

Construir una aplicación de pedidos que integre modelado orientado a objetos, colecciones y Streams, arquitectura por capas, JDBC, transacciones, archivos CSV, excepciones y pruebas. La interfaz puede ser consola o Swing; ninguna regla de negocio debe depender de ella.

El objetivo no es producir muchas clases, sino demostrar que cada responsabilidad tiene una frontera clara y que las decisiones se pueden justificar.

## 2. Escenario

Una tienda necesita registrar pedidos con varias líneas, confirmar el pedido descontando existencias, consultar pedidos y exportar un reporte CSV. Un pedido puede pasar por los estados `BORRADOR`, `CONFIRMADO` o `CANCELADO`.

Reglas mínimas:

1. Un producto tiene ID, nombre, precio monetario y existencias no negativas.
2. Una línea tiene producto, cantidad positiva y precio capturado al momento de agregarla.
3. Un pedido nuevo comienza en `BORRADOR`.
4. Solo un pedido en borrador admite líneas.
5. Confirmar requiere al menos una línea y existencias suficientes para todas.
6. La confirmación y el descuento de stock son una única transacción.
7. Un pedido confirmado no se edita.
8. Cancelar un pedido confirmado repone stock dentro de otra transacción; cancelar dos veces no cambia datos.
9. El dinero se representa con `BigDecimal`, nunca con `double`.
10. Fechas e instantes usan `java.time`.

## 3. Casos de uso

### CU1 — Crear pedido

- Entrada: identificador de cliente.
- Resultado: pedido en borrador con ID.
- Errores: cliente vacío o inexistente según tu modelo.

### CU2 — Agregar línea

- Entrada: ID de pedido, ID de producto y cantidad.
- Resultado: línea agregada con precio actual del producto.
- Errores: pedido/producto inexistente, estado incorrecto o cantidad inválida.

### CU3 — Confirmar pedido

- Comprueba todas las reglas antes de modificar stock.
- Persiste estado y movimientos de inventario atómicamente.
- Ante cualquier error realiza rollback completo.

### CU4 — Consultar

- Buscar pedido por ID con sus líneas.
- Listar por estado y rango de fechas.
- Obtener total y cantidad de unidades.

### CU5 — Cancelar

- Cambia estado y repone existencias.
- Define y documenta si un borrador se elimina o se marca cancelado.

### CU6 — Exportar CSV

- Exporta pedidos confirmados de un rango de fechas.
- El encabezado y orden de columnas son estables.
- Escapa comas, comillas y saltos de línea correctamente.
- Escribe en UTF-8 mediante `Files.newBufferedWriter` y `try-with-resources`.

## 4. Arquitectura requerida

```text
com.tienda.pedidos
├── domain
│   ├── Pedido.java
│   ├── LineaPedido.java
│   ├── Producto.java
│   ├── EstadoPedido.java
│   └── Dinero.java                 # opcional: value object
├── application
│   ├── PedidoService.java
│   ├── PedidoServiceImpl.java
│   └── dto
├── repository
│   ├── PedidoRepository.java
│   └── ProductoRepository.java
├── infrastructure
│   ├── jdbc
│   ├── csv
│   └── connection
├── ui
│   └── Main.java
└── error
```

Dirección de dependencias:

```text
UI -> application -> domain
        |
        v
 repository (interfaces) <- infrastructure (implementaciones)
```

- `domain` no importa JDBC, Swing ni detalles de CSV.
- `application` coordina casos de uso y transacciones.
- `repository` define contratos según el dominio, no según tablas.
- `infrastructure` traduce SQL, filas, archivos y errores técnicos.
- `ui` convierte entrada/salida y maneja errores esperados sin contener reglas.

## 5. Modelo sugerido

```java
public enum EstadoPedido {
    BORRADOR, CONFIRMADO, CANCELADO
}

public record LineaPedido(
        long productoId,
        String nombreProducto,
        BigDecimal precioUnitario,
        int cantidad) {

    public LineaPedido {
        Objects.requireNonNull(nombreProducto);
        Objects.requireNonNull(precioUnitario);
        if (precioUnitario.signum() < 0 || cantidad <= 0) {
            throw new IllegalArgumentException("Línea inválida");
        }
    }

    public BigDecimal subtotal() {
        return precioUnitario.multiply(BigDecimal.valueOf(cantidad));
    }
}
```

Decide si `Pedido` será mutable con operaciones encapsuladas o inmutable devolviendo nuevas versiones. Documenta el costo de tu elección. No expongas directamente una lista mutable de líneas.

## 6. Persistencia JDBC

Tablas mínimas:

```sql
CREATE TABLE producto (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(120) NOT NULL,
    precio DECIMAL(19, 2) NOT NULL CHECK (precio >= 0),
    stock INT NOT NULL CHECK (stock >= 0)
);

CREATE TABLE pedido (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    cliente VARCHAR(120) NOT NULL,
    estado VARCHAR(20) NOT NULL,
    creado_en TIMESTAMP NOT NULL
);

CREATE TABLE linea_pedido (
    pedido_id BIGINT NOT NULL,
    producto_id BIGINT NOT NULL,
    nombre_producto VARCHAR(120) NOT NULL,
    precio_unitario DECIMAL(19, 2) NOT NULL,
    cantidad INT NOT NULL CHECK (cantidad > 0),
    PRIMARY KEY (pedido_id, producto_id),
    FOREIGN KEY (pedido_id) REFERENCES pedido(id),
    FOREIGN KEY (producto_id) REFERENCES producto(id)
);
```

Requisitos técnicos:

- `PreparedStatement` para todos los valores.
- `try-with-resources` para conexión, statement y result set cuando corresponda.
- Credenciales mediante variables de entorno; no se versionan secretos.
- Traducción de `SQLException` a `DataAccessException` conservando `cause`.
- `BigDecimal` con `getBigDecimal`/`setBigDecimal`.
- Transacción explícita para confirmar y cancelar.
- No abrir una conexión nueva por cada línea de un mismo caso de uso.

## 7. Transacción de confirmación

```text
BEGIN
  cargar pedido y líneas
  verificar que sigue en BORRADOR
  comprobar/descontar stock de cada producto
  cambiar pedido a CONFIRMADO
COMMIT

si cualquier paso falla -> ROLLBACK
```

Considera concurrencia: dos pedidos podrían intentar consumir el mismo stock. Documenta una estrategia, por ejemplo actualización condicional:

```sql
UPDATE producto
SET stock = stock - ?
WHERE id = ? AND stock >= ?;
```

Si el número de filas afectadas es cero, el producto no existe o no hay existencias suficientes. Para un proyecto real deberías distinguir ambos casos y definir aislamiento/reintentos.

## 8. Streams requeridos

Usa Streams donde expresen una transformación clara:

- calcular total del pedido;
- agrupar unidades vendidas por producto;
- crear un resumen por estado;
- ordenar resultados para exportación.

No uses `parallelStream()` ni ejecutes una consulta JDBC dentro de cada `map`. Compara al menos una solución con bucle y explica cuál es más legible.

## 9. Manejo de errores

Jerarquía mínima sugerida:

```text
PedidoException
├── PedidoNoEncontradoException
├── EstadoPedidoInvalidoException
└── StockInsuficienteException

DataAccessException
ExportacionException
```

- La UI presenta mensajes de dominio sin imprimir stack traces al usuario.
- La infraestructura conserva la causa técnica para logging y diagnóstico.
- No captures `Exception` de forma general salvo en el límite superior de la aplicación.
- No uses excepciones para decisiones normales que pueden expresarse con un resultado claro.

## 10. Pruebas obligatorias

### Unitarias sin base de datos

- no permite línea con cantidad cero o negativa;
- calcula subtotal y total con escala correcta;
- no confirma un pedido vacío;
- no modifica un pedido confirmado;
- falla si no hay stock;
- servicio funciona con repositorios fake.

### Integración JDBC

- guarda y reconstruye un pedido con líneas;
- hace rollback si falla una actualización intermedia;
- evita stock negativo bajo la estrategia elegida;
- traduce correctamente un error SQL.

### CSV

- contiene encabezado;
- conserva UTF-8;
- escapa nombre con coma y comillas;
- exportar cero resultados produce un archivo válido.

## 11. Entregables

- Código fuente con estructura Maven estándar.
- `README.md` con instalación, variables, esquema y comandos.
- Script SQL reproducible.
- Pruebas automatizadas ejecutables con `mvn test`.
- Ejemplo de CSV sin datos sensibles.
- Diagrama simple de capas y explicación de dos decisiones de diseño.
- Registro breve de mejoras pendientes.

No versionar `target/`, credenciales, archivos del IDE ni datos reales.

## 12. Rúbrica (100 puntos)

| Criterio | Puntos | Evidencia |
|---|---:|---|
| Modelo e invariantes | 15 | estados válidos, dinero correcto, colecciones protegidas |
| Casos de uso | 15 | crear, agregar, confirmar, cancelar, consultar |
| Arquitectura | 15 | dependencias correctas y responsabilidades claras |
| JDBC y transacciones | 20 | SQL parametrizado, recursos, commit/rollback, concurrencia documentada |
| Errores y validación | 10 | jerarquía útil, causa preservada, mensajes correctos |
| Streams y Java moderno | 10 | uso legible y justificado, records/enums adecuados |
| Pruebas | 10 | unitarias, integración y límites |
| Documentación y entrega | 5 | proyecto reproducible y sin secretos |

Una entrega no puede superar 60 puntos si permite stock negativo, no hace rollback, contiene credenciales o mezcla JDBC con la UI.

## 13. Evolución hacia Spring Boot

Después de completar Java SE, migra sin cambiar reglas del dominio:

| Java SE | Spring Boot |
|---|---|
| `Main` ensambla objetos | contenedor inyecta dependencias |
| UI consola/Swing | `@RestController` |
| implementación JDBC | `JdbcTemplate` o Spring Data JPA |
| transacción manual | `@Transactional` en servicio |
| validación manual de DTO | Jakarta Validation |
| captura superior | `@RestControllerAdvice` |

La migración está bien diseñada si el dominio y la mayoría de las pruebas unitarias sobreviven sin cambios.

## Checklist final

- [ ] El proyecto compila desde una clonación limpia.
- [ ] `mvn test` pasa sin servicios manuales inesperados.
- [ ] La UI no importa `java.sql`.
- [ ] El dominio no importa infraestructura.
- [ ] Confirmación y cancelación son atómicas.
- [ ] Nunca se usa `double` para dinero.
- [ ] El CSV es válido con caracteres especiales.
- [ ] No hay secretos ni rutas absolutas versionadas.
- [ ] Cada excepción capturada tiene una acción o traducción justificada.
- [ ] Puedo explicar por qué cada interfaz existe.
