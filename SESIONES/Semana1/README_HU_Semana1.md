# Historias de Usuario – Semana 1

En esta semana, el objetivo es construir las bases de un sistema académico profesional en Java Swing, siguiendo buenas prácticas de programación y diseño.  
A continuación se describen las historias de usuario (HU) que guían el desarrollo y los criterios de aceptación para cada una, junto con detalles prácticos para implementar las funcionalidades clave.

---

## HU01 – Registro de estudiantes

**Como usuario**  
Quiero poder ingresar el nombre, edad y tres notas de un estudiante  
Para que el sistema almacene y muestre la información en una tabla.

**Detalles de implementación:**
- Utiliza un formulario con campos `JTextField` para nombre y notas, y `JSpinner` para edad.
- Al pulsar "Guardar", valida que todos los campos estén completos y correctos.
- Si hay errores (campos vacíos, datos no numéricos, notas fuera de rango), muestra un mensaje claro con `JOptionPane`.
- Al guardar, el estudiante aparece en la tabla (`JTable`) sin sobrescribir los anteriores.

---

## HU02 – Cálculo de estadísticas individuales

**Como usuario**  
Quiero calcular el promedio, la nota máxima y saber si el estudiante está aprobado o reprobado  
Para tener información académica relevante de cada estudiante.

**Detalles de implementación:**
- Usa métodos en la clase `CalculoService` para calcular el promedio y la nota máxima.
- El botón "Calcular" toma los datos del formulario y muestra los resultados en etiquetas (`JLabel`).
- Si los datos son inválidos, muestra un mensaje de error y no realiza el cálculo.

---

## HU03 – Visualización y gestión de la lista de estudiantes

**Como usuario**  
Quiero ver todos los estudiantes registrados en una tabla  
Para consultar y comparar sus datos fácilmente.

**Detalles de implementación:**
- La tabla (`JTable`) se actualiza automáticamente al agregar o cargar estudiantes.
- Usa un modelo de tabla (`DefaultTableModel`) y un método `actualizarListaEstudiantes()` para refrescar los datos.
- Si la lista está vacía, la tabla se muestra vacía y sin errores.

---

## HU04 – Persistencia en archivos CSV

**Como usuario**  
Quiero guardar y cargar la lista de estudiantes en archivos CSV  
Para conservar los datos entre sesiones y poder compartirlos.

**Detalles de implementación:**
- Agrega dos botones: "Guardar CSV" y "Cargar CSV".
- Al guardar, usa `JFileChooser` para seleccionar la ubicación y llama a `ArchivoService.guardarCSV()`.
- Al cargar, usa `JFileChooser` para seleccionar el archivo y llama a `ArchivoService.cargarCSV()`.
- Valida que el archivo tenga el formato correcto (6 columnas: id, nombre, edad, nota1, nota2, nota3).
- Si el archivo está corrupto o tiene datos inválidos, muestra un mensaje de error y no modifica la lista.
- Al cargar, puedes elegir si reemplazar la lista actual o fusionar por ID para evitar duplicados.
- Después de cargar, llama a `actualizarListaEstudiantes()` para refrescar la tabla.

**Ejemplo de validación al cargar:**
```java
try {
    var estudiantes = archivoService.cargarCSV(archivo);
    registroService.reemplazar(estudiantes);
    actualizarListaEstudiantes();
    JOptionPane.showMessageDialog(this, "Datos cargados correctamente.");
} catch (Exception ex) {
    JOptionPane.showMessageDialog(this, "Error al cargar archivo: " + ex.getMessage(),
        "Error", JOptionPane.ERROR_MESSAGE);
}
```

---

## HU05 – Login y control de acceso

**Como usuario**  
Quiero iniciar sesión antes de acceder al sistema  
Para asegurar que solo usuarios autorizados puedan gestionar los datos.

**Detalles de implementación:**
- Crea un formulario de login (`LoginFrame`) con campos `JTextField` para usuario y `JPasswordField` para contraseña.
- Al pulsar "Ingresar", valida que ambos campos estén completos.
- Llama a `UsuarioService.autenticar(usuario, password)` para verificar las credenciales.
- Si son correctas, abre el frame principal (`RegistroEstudianteFrame`); si no, muestra un mensaje de error en un `JLabel`.
- Limpia el campo de contraseña después de cada intento para mayor seguridad.
- Agrega un botón "Cerrar sesión" en el frame principal que regresa al login tras confirmación.

**Ejemplo de validación de login:**
```java
private void btnIngresarActionPerformed(java.awt.event.ActionEvent evt) {
    String user = txtUsuario.getText().trim();
    char[] pass = txtPassword.getPassword();

    if (user.isBlank() || pass.length == 0) {
        lblMensaje.setText("Usuario y contraseña son obligatorios.");
        return;
    }

    boolean ok = usuarioService.autenticar(user, pass);
    Arrays.fill(pass, '\0'); // Limpia la contraseña en memoria

    if (ok) {
        var frame = new RegistroEstudianteFrame();
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
        this.dispose();
    } else {
        lblMensaje.setText("Credenciales inválidas.");
    }
}
```

---

## Buenas prácticas y manejo de errores

- Valida todas las entradas antes de procesar datos o guardar archivos.
- Usa bloques `try/catch` para capturar errores y mostrar mensajes descriptivos con `JOptionPane`.
- Mantén la lógica de negocio en servicios (`ArchivoService`, `UsuarioService`, etc.) y la presentación en la interfaz.
- Documenta el código y comenta las partes clave para facilitar el mantenimiento.
- Limpia los campos sensibles (como contraseñas) después de usarlos.

---

**Estas historias de usuario y detalles de implementación te guiarán para construir un sistema académico robusto, seguro y profesional en Java Swing, con persistencia y control