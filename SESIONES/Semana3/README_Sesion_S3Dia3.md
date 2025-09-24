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
**Comentario:**  
Este paso es fundamental para cualquier operación con la base de datos. Si la conexión falla, no podrás realizar ninguna consulta.

---

## 2. ¿Qué son las Interfaces en Java?

Una **interface** es un contrato que define qué métodos debe tener una clase, pero no cómo se implementan.

**Ejemplo de interface:**
```java
public interface Repositorio<T> {
    void crear(T t);
    T buscarPorId(int id);
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
}
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
**Comentario:**  
- `showInputDialog` muestra una ventana para que el usuario ingrese datos.
- `showMessageDialog` muestra una ventana con un mensaje.
- Estas ventanas son útiles para pedir información o mostrar resultados sin usar la consola.

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
**Comentario:**  
Este flujo asegura que el usuario interactúe de forma sencilla y que todas las operaciones sean validadas y comunicadas correctamente.

---

## 6. Buenas Prácticas

- Usar **PreparedStatement** en lugar de concatenar SQL para evitar inyecciones SQL y mejorar el rendimiento.
- Manejar excepciones (`try-catch`) mostrando mensajes claros al usuario en caso de error.
- Usar `try-with-resources` para cerrar conexiones, sentencias y resultados automáticamente, evitando fugas de recursos.
- Validar los datos antes de guardarlos en la base para evitar errores y mantener la integridad.

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