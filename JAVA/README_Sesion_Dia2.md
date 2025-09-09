# Sesión Día 2 – Semana 1: Tu primera aplicación gráfica en Java Swing

Este documento está diseñado para personas que están aprendiendo Java y quieren crear su primera aplicación gráfica sencilla. Aquí aprenderás a usar NetBeans y Java Swing para registrar estudiantes y calcular el promedio de sus notas, aplicando los conceptos básicos de la Programación Orientada a Objetos (POO).

---

## Objetivo

Construir una aplicación gráfica en Java Swing que permita:

- Registrar los datos de un estudiante (nombre y edad).
- Ingresar tres notas y calcular el promedio.
- Mostrar si el estudiante está aprobado o reprobado (promedio ≥ 3.0).
- Mostrar la nota más alta.
- Limpiar los campos del formulario.

---

## Descripción general

Vas a crear un formulario visual donde podrás escribir los datos de un estudiante y sus notas. Al presionar un botón, el programa calculará el promedio, mostrará si el estudiante aprobó o no, y cuál fue la nota más alta. También tendrás botones para limpiar los campos y salir de la aplicación.

---

## Pasos de desarrollo

### Paso 1: Crear el proyecto en NetBeans

1. Abre NetBeans.
2. Ve a **File > New Project > Java with Maven > Java Application**.
3. Nombra el proyecto como `RegistroAcademico`.

---

### Paso 2: Diseñar la interfaz gráfica

1. Haz clic derecho en el paquete principal y selecciona **New > JFrame Form**.
2. Nombra el formulario como `RegistroEstudianteFrame`.
3. Usa el diseñador visual para agregar los siguientes componentes:
   - **Campos de texto (JTextField):**
     - Nombre
     - Edad
     - Nota 1
     - Nota 2
     - Nota 3
   - **Botones (JButton):**
     - Calcular
     - Limpiar
     - Salir
   - **Etiquetas (JLabel):**
     - Promedio
     - Nota máxima
     - Resultado (Aprobado/Reprobado)

---

### Paso 3: Crear la clase Estudiante

1. Haz clic derecho en el paquete `domain` y selecciona **New > Java Class**.
2. Nombra la clase como `Estudiante`.
3. Declara los atributos como privados:

   ```java
   private String nombre;
   private int edad;
   private double nota1, nota2, nota3;
   ```

4. Crea un constructor que reciba todos los atributos:

   ```java
   public Estudiante(String nombre, int edad, double nota1, double nota2, double nota3) {
       this.nombre = nombre;
       this.edad = edad;
       this.nota1 = nota1;
       this.nota2 = nota2;
       this.nota3 = nota3;
   }
   ```

5. Implementa los métodos:

   ```java
   // Calcula el promedio de las tres notas
   public double calcularPromedio() {
       return (nota1 + nota2 + nota3) / 3.0;
   }

   // Devuelve la nota más alta
   public double notaMaxima() {
       return Math.max(nota1, Math.max(nota2, nota3));
   }

   // Indica si el estudiante está aprobado (promedio >= 3.0)
   public boolean estaAprobado() {
       return calcularPromedio() >= 3.0;
   }
   ```

---

### Paso 4: Programar los botones en el formulario

- **Botón Calcular:**
  - Toma los valores de los campos de texto.
  - Crea un objeto `Estudiante`.
  - Llama a los métodos y muestra los resultados en las etiquetas.

  ```java
  // Ejemplo dentro del evento del botón Calcular
  String nombre = txtNombre.getText();
  int edad = Integer.parseInt(txtEdad.getText());
  double nota1 = Double.parseDouble(txtNota1.getText());
  double nota2 = Double.parseDouble(txtNota2.getText());
  double nota3 = Double.parseDouble(txtNota3.getText());

  Estudiante estudiante = new Estudiante(nombre, edad, nota1, nota2, nota3);

  lblPromedio.setText("Promedio: " + estudiante.calcularPromedio());
  lblNotaMaxima.setText("Nota máxima: " + estudiante.notaMaxima());
  lblResultado.setText(estudiante.estaAprobado() ? "Aprobado" : "Reprobado");
  ```

- **Botón Limpiar:**
  - Vacía todos los campos y etiquetas.

  ```java
  txtNombre.setText("");
  txtEdad.setText("");
  txtNota1.setText("");
  txtNota2.setText("");
  txtNota3.setText("");
  lblPromedio.setText("");
  lblNotaMaxima.setText("");
  lblResultado.setText("");
  ```

- **Botón Salir:**
  - Cierra la aplicación.

  ```java
  dispose(); // o System.exit(0);
  ```

---

## Criterios de aceptación

- El proyecto debe compilar y ejecutarse sin errores en NetBeans.
- Los atributos de la clase `Estudiante` deben ser privados.
- Deben existir métodos para calcular promedio, nota máxima y aprobado/reprobado.
- La interfaz debe ser clara y funcional, mostrando los resultados correctos.

---

## Tiempo estimado

- **Primera hora:** Diseño del formulario en Swing.
- **Segunda hora:** Creación de la clase `Estudiante` y sus métodos.
- **Tercera hora:** Conexión entre la interfaz y la clase, pruebas y corrección de errores.

---

## Recomendaciones

- Usa nombres de variables y métodos claros y descriptivos.
- Prueba con diferentes combinaciones de notas para confirmar que los cálculos funcionan.
- Comenta el código para explicar brevemente qué hace cada sección.

---

**¡Listo! Al finalizar tendrás tu primera aplicación gráfica en Java que registra estudiantes y calcula sus notas. Si tienes dudas, consulta con tu instructor o busca ejemplos en línea sobre Java Swing y Programación Orientada a Objetos. ¡Buena suerte!**