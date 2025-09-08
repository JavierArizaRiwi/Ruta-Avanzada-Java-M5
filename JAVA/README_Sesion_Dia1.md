# 📘 Sesión Día 1 – Semana 1: Fundamentos + Setup del proyecto

## 🎯 Objetivos del día
1. Configurar el entorno de desarrollo en Linux.  
2. Crear un proyecto base en Java con Maven.  
3. Inicializar repositorio Git y configurar GitFlow.  
4. Implementar las primeras clases del dominio aplicando principios de POO.  
5. Documentar el proceso en un README inicial.  

---

## 🛠️ Paso 1 – Verificar e instalar Java 17
Verifica si tienes Java instalado:
```bash
java -version
```

Si no tienes la versión 17, instala con:
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

Confirma instalación:
```bash
java -version
```

---

## 🛠️ Paso 2 – Instalar Maven
Maven nos servirá para compilar y ejecutar:
```bash
sudo apt install maven -y
mvn -v
```

---

## 🛠️ Paso 3 – Instalar un IDE
Puedes usar **IntelliJ IDEA Community** o **NetBeans**:

### Opción A – IntelliJ
```bash
sudo snap install intellij-idea-community --classic
```

### Opción B – NetBeans
```bash
sudo snap install netbeans --classic
```

---

## 🛠️ Paso 4 – Configurar proyecto Java
1. Crea un nuevo proyecto **Java con Maven**.  
2. Define el `GroupId` como `com.codeup` y el `ArtifactId` como `academico`.  
3. Organiza paquetes:
   ```
   src/main/java/com/codeup/academico
   ├─ domain
   ├─ ui/console
   └─ App.java
   ```

---

## 🛠️ Paso 5 – Configurar Git y GitHub
1. Verifica Git:
   ```bash
   git --version
   ```
   Si no está instalado:
   ```bash
   sudo apt install git -y
   ```

2. Configura tu usuario:
   ```bash
   git config --global user.name "Tu Nombre"
   git config --global user.email "tuemail@ejemplo.com"
   ```

3. Inicializa el repositorio:
   ```bash
   git init
   git checkout -b develop
   ```

4. Conecta con GitHub:
   ```bash
   git remote add origin https://github.com/TU_USUARIO/codeup-academico.git
   git push -u origin develop
   ```

5. Crea tu primera rama de feature:
   ```bash
   git checkout -b feature/setup
   ```

---

## 💻 Paso 6 – Primer código en Java

Archivo `App.java`:
```java
package com.codeup.academico;

public class App {
    public static void main(String[] args) {
        System.out.println("Sistema Académico CodeUp iniciado correctamente 🚀");
    }
}
```

Archivo `domain/Estudiante.java`:
```java
package com.codeup.academico.domain;

public class Estudiante {
    private final String id;
    private String nombre;

    public Estudiante(String id, String nombre) {
        if (id == null || id.isBlank()) throw new IllegalArgumentException("id requerido");
        if (nombre == null || nombre.isBlank()) throw new IllegalArgumentException("nombre requerido");
        this.id = id;
        this.nombre = nombre;
    }

    public String getId() { return id; }
    public String getNombre() { return nombre; }
}
```

---

## 🛠️ Paso 7 – Ejecutar el proyecto
Compila y ejecuta:
```bash
mvn compile
mvn exec:java -Dexec.mainClass="com.codeup.academico.App"
```

---

## 📝 Paso 8 – Documentación inicial
Crea un **README.md** en el proyecto:

```markdown
# Sistema Académico CodeUp

### Requisitos
- Java 17
- Maven
- Git

### Instalación y ejecución
```bash
git clone https://github.com/TU_USUARIO/codeup-academico.git
cd codeup-academico
mvn compile
mvn exec:java -Dexec.mainClass="com.codeup.academico.App"
```

### Estructura inicial
```
src/main/java/com/codeup/academico
 ├─ domain
 ├─ ui/console
 └─ App.java
```
```

---

## ✅ Ejercicio práctico del día
1. Crear un **estudiante** y un **curso** en código.  
2. Imprimirlos en consola con `System.out.println()`.  
3. Hacer commit en la rama `feature/setup`.  
4. Crear un Pull Request a la rama `develop` en GitHub.  

---

📌 **Resultado esperado hoy**  
- Entorno Java + Maven + IDE configurado en Linux.  
- Proyecto Java funcionando.  
- Repositorio GitHub conectado con ramas (`develop` y `feature/setup`).  
- Primer commit con `App.java` y `Estudiante.java`.  
- README inicial creado.  
