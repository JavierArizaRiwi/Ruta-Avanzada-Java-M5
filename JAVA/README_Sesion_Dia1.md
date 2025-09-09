# Sesión Día 1 – Semana 1: Fundamentos y Configuración de Proyecto Java

Este documento está pensado para personas que nunca han trabajado con Java. Aquí aprenderás a instalar las herramientas necesarias, crear tu primer proyecto y entender qué hace cada paso.

---

## Objetivos del día

1. Instalar y verificar Java en Linux.
2. Instalar Maven para compilar y ejecutar proyectos Java.
3. Instalar un editor de código (IDE) para escribir tu código.
4. Crear la estructura básica de un proyecto Java usando Maven.
5. Configurar Git para controlar versiones y conectar tu proyecto con GitHub.
6. Escribir tus primeras clases en Java y ejecutarlas.
7. Documentar el proceso.

---

## Paso 1 – Instalar Java 17

**¿Qué es Java?**  
Java es el lenguaje de programación que usarás. Necesitas instalarlo para poder crear y ejecutar programas.

**Verificar si tienes Java instalado:**  
Abre la terminal y escribe:

```bash
java -version
```

Si ves algo como `openjdk version "17..."`, ya tienes Java 17.  
Si no, instala Java con:

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

Vuelve a verificar la instalación:

```bash
java -version
```

---

## Paso 2 – Instalar Maven

**¿Qué es Maven?**  
Maven es una herramienta que te ayuda a organizar, compilar y ejecutar proyectos Java.

Instala Maven con:

```bash
sudo apt install maven -y
```

Verifica la instalación:

```bash
mvn -v
```

---

## Paso 3 – Instalar un editor de código (IDE)

**¿Qué es un IDE?**  
Un IDE es un programa que te ayuda a escribir y organizar tu código.

Puedes instalar uno de estos dos:

- **IntelliJ IDEA Community**  
  ```bash
  sudo snap install intellij-idea-community --classic
  ```

- **NetBeans**  
  ```bash
  sudo snap install netbeans --classic
  ```

---

## Paso 4 – Crear la estructura básica del proyecto Java

**¿Qué es un proyecto Maven?**  
Es una forma organizada de crear proyectos Java.

1. Abre tu IDE y crea un nuevo proyecto Java usando Maven.
2. Cuando te pida datos, escribe:
   - **GroupId:** com.codeup
   - **ArtifactId:** academico

3. Organiza las carpetas así:

```
src/main/java/com/codeup/academico
 ├─ domain
 ├─ ui/console
 └─ App.java
```

- **domain:** Aquí pondrás las clases que representan objetos importantes (por ejemplo, Estudiante).
- **ui/console:** Aquí pondrás el código que interactúa con el usuario por consola.
- **App.java:** Es el punto de entrada de tu programa.

---

## Paso 5 – Configurar Git y GitHub

**¿Qué es Git?**  
Git es una herramienta para guardar versiones de tu código y trabajar en equipo.

**¿Qué es GitHub?**  
GitHub es una página web donde puedes guardar tu código y compartirlo.

1. Verifica si tienes Git:

   ```bash
   git --version
   ```

   Si no lo tienes, instálalo:

   ```bash
   sudo apt install git -y
   ```

2. Configura tu nombre y correo (esto identifica tus cambios):

   ```bash
   git config --global user.name "Tu Nombre"
   git config --global user.email "tuemail@ejemplo.com"
   ```

3. Inicializa el repositorio (esto crea una carpeta especial para guardar versiones):

   ```bash
   git init
   git checkout -b develop
   ```

4. Conecta tu proyecto con GitHub (debes crear el repositorio en GitHub antes):

   ```bash
   git remote add origin https://github.com/TU_USUARIO/codeup-academico.git
   git push -u origin develop
   ```

5. Crea una rama para trabajar en una nueva funcionalidad:

   ```bash
   git checkout -b feature/setup
   ```

---

## Paso 6 – Escribir tu primer código en Java

**¿Qué es una clase en Java?**  
Una clase es como un molde para crear objetos. Por ejemplo, la clase `Estudiante` te permite crear estudiantes.

**Archivo principal: App.java**

```java
package com.codeup.academico;

// Esta clase es el punto de inicio del programa
public class App {
    public static void main(String[] args) {
        System.out.println("Sistema Académico CodeUp iniciado correctamente");
    }
}
```

**Clase Estudiante: domain/Estudiante.java**

```java
package com.codeup.academico.domain;

// Esta clase representa un estudiante
public class Estudiante {
    private final String id;     // Identificador único del estudiante
    private String nombre;       // Nombre del estudiante

    // Constructor: crea un estudiante con id y nombre
    public Estudiante(String id, String nombre) {
        if (id == null || id.isBlank()) throw new IllegalArgumentException("id requerido");
        if (nombre == null || nombre.isBlank()) throw new IllegalArgumentException("nombre requerido");
        this.id = id;
        this.nombre = nombre;
    }

    // Métodos para obtener los datos del estudiante
    public String getId() { return id; }
    public String getNombre() { return nombre; }
}
```

**Ejemplo de cómo crear e imprimir un estudiante en App.java:**

```java
// ...dentro del método main en App.java...
Estudiante estudiante = new Estudiante("1", "Ana Pérez");
System.out.println("Estudiante creado: " + estudiante.getNombre() + " (ID: " + estudiante.getId() + ")");
```

---

## Paso 7 – Compilar y ejecutar el proyecto

**¿Qué significa compilar?**  
Traducir tu código a un formato que la computadora pueda entender.

**¿Cómo se hace?**  
En la terminal, dentro de la carpeta del proyecto, escribe:

```bash
mvn compile
mvn exec:java -Dexec.mainClass="com.codeup.academico.App"
```

---

## Paso 8 – Documentar el proyecto

Crea un archivo llamado `README.md` en la raíz del proyecto con esta información:

```markdown
# Sistema Académico CodeUp

## Requisitos
- Java 17
- Maven
- Git

## Instalación y ejecución

```bash
git clone https://github.com/TU_USUARIO/codeup-academico.git
cd codeup-academico
mvn compile
mvn exec:java -Dexec.mainClass="com.codeup.academico.App"
```

## Estructura inicial

```
src/main/java/com/codeup/academico
 ├─ domain
 ├─ ui/console
 └─ App.java
```
```

---

## Ejercicio práctico

1. Crea una clase `Curso` en el paquete `domain` similar a la clase `Estudiante`:

```java
package com.codeup.academico.domain;

public class Curso {
    private final String codigo;
    private String nombre;

    public Curso(String codigo, String nombre) {
        if (codigo == null || codigo.isBlank()) throw new IllegalArgumentException("código requerido");
        if (nombre == null || nombre.isBlank()) throw new IllegalArgumentException("nombre requerido");
        this.codigo = codigo;
        this.nombre = nombre;
    }

    public String getCodigo() { return codigo; }
    public String getNombre() { return nombre; }
}
```

2. En `App.java`, crea un estudiante y un curso, e imprímelos:

```java
Estudiante estudiante = new Estudiante("1", "Ana Pérez");
Curso curso = new Curso("101", "Programación Java");

System.out.println("Estudiante: " + estudiante.getNombre() + " (ID: " + estudiante.getId() + ")");
System.out.println("Curso: " + curso.getNombre() + " (Código: " + curso.getCodigo() + ")");
```

3. Haz commit de tus cambios en la rama `feature/setup`:

```bash
git add .
git commit -m "Primer estudiante y curso creados"
git push origin feature/setup
```

4. Crea un Pull Request en GitHub para unir tus cambios a la rama `develop`.

---

## Resultado esperado

- Java, Maven y un IDE instalados y funcionando en Linux.
- Proyecto Java creado y ejecutándose correctamente.
- Repositorio GitHub conectado y con ramas (`develop` y `feature/setup`).
- Primer commit con las clases `App.java`, `Estudiante.java` y `Curso.java`.
- Documentación inicial en `README.md`.

---

**¡Listo! Ahora tienes tu primer proyecto Java funcionando y documentado. Si tienes dudas, pregunta a tu instructor o busca en la documentación oficial de Java, Maven, Git y GitHub.**
