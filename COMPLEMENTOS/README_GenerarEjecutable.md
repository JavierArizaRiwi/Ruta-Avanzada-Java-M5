# Guía para Generar un JAR Ejecutable en NetBeans con Maven

Este documento explica cómo construir un archivo **JAR ejecutable** a partir de un proyecto Maven en NetBeans. Se incluye la configuración del archivo `pom.xml`, los pasos de compilación y ejecución, así como una opción adicional para generar un JAR con todas las dependencias incluidas (*fat JAR*).

---

## 1. Configuración del `pom.xml`

En el archivo `pom.xml` se definen las propiedades y plugins necesarios para compilar y empaquetar el proyecto. Asegúrate de incluir:

- **Versión de Java**: especificada en `<maven.compiler.release>`.
- **Clase principal**: definida en la propiedad `<exec.mainClass>`.
- **Plugins de compilación y empaquetado**: `maven-compiler-plugin` y `maven-jar-plugin`.

### Ejemplo de `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.mycompany</groupId>
    <artifactId>Pagos</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.release>17</maven.compiler.release>
        <exec.mainClass>com.mycompany.pagos.DemoIntegrador</exec.mainClass>
    </properties>

    <build>
        <plugins>
            <!-- Compilador -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>

            <!-- Generación del JAR -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>${exec.mainClass}</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

---

## 2. Pasos en NetBeans

1. **Abrir el proyecto**  
   - File > Open Project… > Seleccionar proyecto Maven.

2. **Verificar clase principal**  
   - Click derecho en el proyecto > Properties > Run > Main Class.  
   - Debe coincidir con:  
     ```
     com.mycompany.pagos.DemoIntegrador
     ```

3. **Compilar y empaquetar**  
   - Click derecho sobre el proyecto > Clean and Build.  
   - Maven ejecutará automáticamente `mvn clean package`.  
   - El archivo generado se ubicará en:  
     ```
     target/Pagos-1.0-SNAPSHOT.jar
     ```

4. **Ejecutar el JAR**  
   - Desde NetBeans: presionar **Run (F6)**.  
   - Desde terminal:  
     ```bash
     cd ruta/del/proyecto/target
     java -jar Pagos-1.0-SNAPSHOT.jar
     ```

---

## 3. Opción Avanzada: JAR con Dependencias (Fat JAR)

Si tu proyecto utiliza librerías externas, puedes generar un JAR único que las incluya. Para esto se utiliza el plugin `maven-shade-plugin`.

### Configuración en el `pom.xml`

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.5.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <createDependencyReducedPom>true</createDependencyReducedPom>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.mycompany.pagos.DemoIntegrador</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Con esta configuración, al ejecutar **Clean and Build**, se generará un archivo:  

```
target/Pagos-1.0-SNAPSHOT-shaded.jar
```

Este archivo incluye todas las dependencias necesarias y puede ejecutarse directamente en cualquier máquina con:

```bash
java -jar Pagos-1.0-SNAPSHOT-shaded.jar
```

---

## 4. Conclusión

- El `pom.xml` es la pieza clave para definir la compilación y empaquetado del proyecto.  
- NetBeans simplifica el proceso con la opción **Clean and Build**.  
- Si se requiere distribuir un único archivo con todas las dependencias, se recomienda usar el `maven-shade-plugin`.