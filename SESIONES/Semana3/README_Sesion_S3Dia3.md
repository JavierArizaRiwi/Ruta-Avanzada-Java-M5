# Mini-Tienda con Interfaces + JDBC básico (Introducción)

Este documento es una guía de aprendizaje para entender los conceptos que se aplicarán en la **Historia de Usuario (HU) – Semana 3**.  
El objetivo es que comprendas **qué son las Interfaces, cómo funciona JDBC y cómo usar JOptionPane** en Java, antes de implementar la Mini-Tienda.

---

## 1. ¿Qué es JDBC?

**JDBC (Java Database Connectivity)** es una API de Java que permite conectar programas con bases de datos relacionales.

**Flujo básico de JDBC:**
1. **Cargar el driver:** Es la librería que permite que Java se comunique con la base de datos. Normalmente, solo necesitas tener el driver en tus dependencias (Maven/Gradle).
2. **Crear una conexión:** Usas la clase `DriverManager` para obtener un objeto `Connection` que representa la conexión activa con la base de datos.
3. **Enviar consultas SQL:** Usas objetos como `PreparedStatement` para ejecutar sentencias SQL (como SELECT, INSERT, UPDATE, DELETE).
4. **Procesar los resultados:** Si ejecutas una consulta (SELECT), obtienes un `ResultSet` para recorrer los datos devueltos.
5. **Cerrar la conexión:** Es importante liberar los recursos cerrando la conexión cuando ya no se necesita.

**Ejemplo de conexión:**
```java
// Se obtiene una conexión a la base de datos usando la URL, usuario y contraseña.
// Es importante manejar excepciones y cerrar la conexión después de usarla.
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/tienda", "root", "password"
);
```
**Ejemplo de consulta SELECT:**
```java
String sql = "SELECT id, nombre, precio, stock FROM productos";
try (PreparedStatement ps = conn.prepareStatement(sql);
     ResultSet rs = ps.executeQuery()) {
    while (rs.next()) {
        int id = rs.getInt("id");
        String nombre = rs.getString("nombre");
        double precio = rs.getDouble("precio");
        int stock = rs.getInt("stock");
        System.out.println(id + " - " + nombre + " - $" + precio + " - Stock: " + stock);
    }
}
```
**Ejemplo de inserción (INSERT):**
```java
String sql = "INSERT INTO productos (nombre, precio, stock) VALUES (?, ?, ?)";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setString(1, "Teclado");
    ps.setDouble(2, 49.99);
    ps.setInt(3, 10);
    ps.executeUpdate();
}
```
**Comentario:**  
Estos ejemplos muestran cómo ejecutar operaciones básicas con JDBC, siempre usando `PreparedStatement` para evitar inyecciones SQL.

---

## 2. ¿Qué son las Interfaces en Java?

Una **interface** es un contrato que define qué métodos debe tener una clase, pero no cómo se implementan.

**Ejemplo de interface:**
```java
public interface Repositorio<T> {
    void crear(T t);
    T buscarPorId(int id);
    List<T> listar();
    void actualizar(T t);
    void eliminar(int id);
}
```
**¿Por qué se declara con `<T>`?**  
El `<T>` indica que la interface es **genérica** y puede trabajar con cualquier tipo de objeto, no solo con uno específico.  
Esto permite que el repositorio sea reutilizable para diferentes entidades (por ejemplo, `Producto`, `Cliente`, `Usuario`).  
Así, puedes crear `Repositorio<Producto>`, `Repositorio<Cliente>`, etc., usando la misma interface.  
El tipo `T` se reemplaza por el tipo concreto cuando implementas o usas la interface.

**Implementación de la interface:**
```java
public class ProductoRepositorio implements Repositorio<Producto> {
    @Override
    public void crear(Producto p) {
        // Lógica para insertar en BD
    }

    @Override
    public Producto buscarPorId(int id) {
        // Lógica para consultar en BD
        return null;
    }

    @Override
    public List<Producto> listar() {
        // Lógica para listar productos
        return new ArrayList<>();
    }

    @Override
    public void actualizar(Producto p) {
        // Lógica para actualizar producto
    }

    @Override
    public void eliminar(int id) {
        // Lógica para eliminar producto
    }
}
```
**Ejemplo de uso de la interface:**
```java
ProductoRepositorio repo = new ProductoRepositorio();
Producto nuevo = new Producto("Mouse", 25.50, 20);
repo.crear(nuevo);

Producto encontrado = repo.buscarPorId(1);
System.out.println("Producto encontrado: " + encontrado.getNombre());
```
**Comentario:**  
La clase `ProductoRepositorio` implementa los métodos definidos en la interface. Aquí es donde se escribe la lógica real, por ejemplo, cómo insertar un producto en la base de datos.

**Beneficio:**  
Separar el **qué hace** (contrato) del **cómo lo hace** (implementación) permite cambiar la lógica interna sin afectar el resto del sistema. Facilita pruebas y mantenimiento.

---

## 3. ¿Qué es JOptionPane?

`JOptionPane` es una clase de Java que permite crear **ventanas emergentes** para interactuar con el usuario.

**Ejemplo de uso:**
```java
String nombre = JOptionPane.showInputDialog("Ingrese el nombre del producto:");
JOptionPane.showMessageDialog(null, "Producto: " + nombre);
```
**Ejemplo de menú con JOptionPane:**
```java
String[] opciones = {"Crear", "Consultar", "Actualizar", "Eliminar", "Salir"};
int seleccion = JOptionPane.showOptionDialog(
    null,
    "Seleccione una opción:",
    "Menú Mini-Tienda",
    JOptionPane.DEFAULT_OPTION,
    JOptionPane.INFORMATION_MESSAGE,
    null,
    opciones,
    opciones[0]
);
```
**Ejemplo de validación de entrada:**
```java
String precioStr = JOptionPane.showInputDialog("Ingrese el precio:");
try {
    double precio = Double.parseDouble(precioStr);
    if (precio <= 0) throw new NumberFormatException();
    // Continuar con el flujo
} catch (NumberFormatException ex) {
    JOptionPane.showMessageDialog(null, "Precio inválido. Debe ser un número positivo.");
}
```
**Comentario:**  
- `showInputDialog` muestra una ventana para que el usuario ingrese datos.
- `showMessageDialog` muestra una ventana con un mensaje.
- `showOptionDialog` permite crear menús interactivos.
- Validar la entrada del usuario es clave para evitar errores y mejorar la experiencia.

**Ventajas:**  
- Es fácil de usar y no requiere configuración adicional.
- Mejora la experiencia del usuario al mostrar mensajes claros y formularios simples.
- Permite construir menús y flujos interactivos en aplicaciones de escritorio.

---

## 4. Esquema de la Base de Datos

La HU trabajará con una tabla llamada **productos**.

**Ejemplo de creación de tabla:**
```sql
CREATE TABLE productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50) UNIQUE,
    precio DECIMAL(10,2),
    stock INT
);
```
**Ejemplo de consulta para verificar productos:**
```sql
SELECT * FROM productos;
```
**Comentario:**  
- La columna `id` es la clave primaria y se incrementa automáticamente.
- `nombre` debe ser único para evitar productos duplicados.
- `precio` y `stock` almacenan el valor y la cantidad disponible.

---

## 5. Flujo de la Mini-Tienda

1. **Menú principal:** Se muestra usando JOptionPane, permitiendo al usuario elegir qué acción realizar (crear, consultar, actualizar, eliminar).
2. **Operaciones CRUD con JDBC:**
   - **Crear (INSERT):** Agrega un nuevo producto a la base de datos.
   - **Leer (SELECT):** Consulta productos existentes.
   - **Actualizar (UPDATE):** Modifica datos de un producto.
   - **Eliminar (DELETE):** Borra un producto de la base de datos.
3. **Validaciones:**
   - Verifica que los campos no estén vacíos antes de guardar.
   - Comprueba que los valores numéricos sean válidos (por ejemplo, precio positivo).
   - Evita duplicados en el nombre usando la restricción UNIQUE.

**Diagrama de flujo simplificado:**
```
[JOptionPane Menú] → [Selecciona opción] → [CRUD con JDBC] → [Mensaje al usuario]
```
**Ejemplo de flujo de creación de producto:**
```java
String nombre = JOptionPane.showInputDialog("Nombre:");
String precioStr = JOptionPane.showInputDialog("Precio:");
String stockStr = JOptionPane.showInputDialog("Stock:");
try {
    double precio = Double.parseDouble(precioStr);
    int stock = Integer.parseInt(stockStr);
    Producto nuevo = new Producto(nombre, precio, stock);
    repo.crear(nuevo);
    JOptionPane.showMessageDialog(null, "Producto creado exitosamente.");
} catch (Exception ex) {
    JOptionPane.showMessageDialog(null, "Datos inválidos. Intente de nuevo.");
}
```
**Comentario:**  
Este flujo asegura que el usuario interactúe de forma sencilla y que todas las operaciones sean validadas y comunicadas correctamente.

---

## 6. Buenas Prácticas

- Usar **PreparedStatement** en lugar de concatenar SQL para evitar inyecciones SQL y mejorar el rendimiento.
- Manejar excepciones (`try-catch`) mostrando mensajes claros al usuario en caso de error.
- Usar `try-with-resources` para cerrar conexiones, sentencias y resultados automáticamente, evitando fugas de recursos.
- Validar los datos antes de guardarlos en la base para evitar errores y mantener la integridad.

**Ejemplo de try-with-resources:**
```java
try (Connection conn = DriverManager.getConnection(url, user, pass);
     PreparedStatement ps = conn.prepareStatement(sql)) {
    // Operaciones con la base de datos
}
```
**Comentario:**  
Aplicar estas prácticas hace que tu aplicación sea más segura, robusta y fácil de mantener.

---

## 7. Mejoras y recomendaciones adicionales

- **Explica por qué usar interfaces genéricas:** Permiten reutilizar el mismo contrato para diferentes entidades, haciendo el código más flexible y evitando duplicación.
- **Agrega comentarios detallados en el código:** Cada bloque debe explicar qué hace y por qué es importante.
- **Incluye ejemplos claros de cada concepto:** Así el aprendizaje es más visual y práctico.
- **Aplica validaciones y manejo de errores:** No solo por seguridad, sino para mejorar la experiencia del usuario.
- **Utiliza JOptionPane para todos los flujos de interacción:** Esto facilita la construcción de menús y formularios amigables.
- **Documenta el flujo de la aplicación:** Un diagrama o esquema ayuda a entender cómo se conectan los componentes.

---

## 8. Conclusión

Con este material deberías comprender:
- Cómo se conecta Java con una base de datos usando JDBC y por qué es importante cerrar conexiones y manejar errores.
- Cómo las Interfaces ayudan a organizar el código, separando el contrato de la implementación.
- Cómo JOptionPane mejora la interacción con el usuario en aplicaciones de escritorio.
- Cómo se combinan estos elementos en una Mini-Tienda para realizar operaciones CRUD de manera segura y eficiente.

**Comentario final:**  
Dominar estos conceptos te permitirá construir aplicaciones Java más profesionales, seguras y fáciles de mantener. Practica cada parte por separado y luego intégralas en tu Mini-Tienda para crear una aplicación completa y funcional. ¡Éxito en tu aprendizaje y desarrollo!