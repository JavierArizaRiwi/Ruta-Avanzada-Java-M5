# Guía de Javadoc para Proyectos Java SE

Este instructivo explica **cómo documentar y generar Javadoc** en proyectos Java SE: desde escribir comentarios correctos, hasta producir la documentación con **línea de comandos**, **Maven** y **NetBeans**. Incluye ejemplos prácticos y un checklist de buenas prácticas.

---

## 1) ¿Qué es Javadoc y por qué usarlo?
Javadoc es la herramienta oficial del JDK para **generar documentación HTML** a partir de comentarios especiales en el código fuente. Beneficios:
- Estandariza la documentación de APIs.
- Facilita el onboarding de nuevos desarrolladores.
- Ayuda a mantener contratos claros (métodos, parámetros, excepciones).
- Se integra con IDEs, Maven/Gradle y pipelines CI.

---

## 2) Escribir comentarios Javadoc correctamente

### 2.1. Estructura básica
Los comentarios Javadoc se abren con `/**` y se ubican **inmediatamente antes** de la clase, método, campo o paquete que documentan.

```java
/**
 * Calculadora básica para operaciones aritméticas.
 * <p>Ejemplo de uso:</p>
 * <pre>{@code
 * Calculadora calc = new Calculadora();
 * int suma = calc.sumar(2, 3);
 * }</pre>
 * @author Tu Nombre
 * @since 1.0
 */
public class Calculadora {

    /**
     * Suma dos enteros.
     *
     * @param a primer sumando
     * @param b segundo sumando
     * @return la suma de {@code a} y {@code b}
     * @throws ArithmeticException si ocurre un overflow (ejemplo conceptual)
     */
    public int sumar(int a, int b) {
        return Math.addExact(a, b);
    }
}
```

### 2.2. Etiquetas Javadoc más comunes
- `@param <nombre>`: describe cada parámetro.
- `@return`: describe el valor de retorno.
- `@throws` o `@exception`: describe condiciones de error y excepciones lanzadas.
- `@since`: versión de la API en la que se añadió el elemento.
- `@deprecated`: marca APIs obsoletas; explicar alternativa.
- `@see` y `{@link ...}`: referencias cruzadas a otros tipos/miembros.
- `{@code ...}` y `<pre>...</pre>`: bloques de código y literales.
- `@author`, `@version`: metadatos del autor/versión (opcionales, a menudo se gestionan vía control de versiones).

### 2.3. Documentar paquetes y módulos
- **`package-info.java`**: crea un archivo por paquete con la descripción general del paquete.
```java
/**
 * Contiene las clases de dominio del módulo de Pagos.
 * Provee entidades y utilidades para la capa de negocio.
 */
package com.mycompany.pagos.domain;
```
- **`module-info.java`** (si usas módulos Java 9+): agrega descripciones en el encabezado del módulo (comentario Javadoc antes del `module`), y usa `exports` para indicar qué paquetes son públicos.

---

## 3) Generar Javadoc por línea de comandos (JDK)

### 3.1. Estructura típica de proyecto
Supongamos un proyecto con fuentes en `src/main/java` y paquetes bajo `com.mycompany.pagos`.

### 3.2. Comando básico
```bash
javadoc \
  -d docs/api \
  -sourcepath src/main/java \
  -subpackages com.mycompany.pagos
```
- `-d docs/api`: carpeta de salida HTML.
- `-sourcepath`: raíz del código fuente.
- `-subpackages`: genera doc para todos los subpaquetes.

### 3.3. Opciones útiles
- `-encoding UTF-8 -docencoding UTF-8 -charset UTF-8`: asegura correcto manejo de acentos.
- `-private` | `-protected` | `-public`: nivel de visibilidad a documentar (por defecto `protected` y superiores).
- `-link`: enlaza con documentación externa (por ejemplo, el JDK).
```bash
javadoc \
  -d docs/api \
  -sourcepath src/main/java \
  -subpackages com.mycompany.pagos \
  -encoding UTF-8 -docencoding UTF-8 -charset UTF-8 \
  -link https://docs.oracle.com/en/java/javase/17/docs/api/
```

---

## 4) Generar Javadoc con Maven

### 4.1. Plugin básico
Agrega el plugin al `pom.xml`:
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-javadoc-plugin</artifactId>
      <version>3.6.3</version>
      <configuration>
        <encoding>UTF-8</encoding>
        <docencoding>UTF-8</docencoding>
        <charset>UTF-8</charset>
        <source>17</source>
        <!-- Para incluir privados (opcional): -->
        <!-- <show>private</show> -->
        <!-- Enlace a Javadoc del JDK -->
        <links>
          <link>https://docs.oracle.com/en/java/javase/17/docs/api/</link>
        </links>
      </configuration>
    </plugin>
  </plugins>
</build>
```

### 4.2. Generar la documentación
```bash
mvn clean javadoc:javadoc
```
Salida por defecto: `target/site/apidocs`.

### 4.3. Empaquetar la Javadoc como JAR (útil para publicar artefactos)
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-javadoc-plugin</artifactId>
  <version>3.6.3</version>
  <executions>
    <execution>
      <id>attach-javadocs</id>
      <goals>
        <goal>jar</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```
Luego:
```bash
mvn clean package javadoc:jar
```
Artefacto resultante: `target/tu-artifactId-version-javadoc.jar`.

---

## 5) Generar Javadoc desde NetBeans

### 5.1. Proyectos Maven
- Click derecho en el proyecto → **Generate Javadoc** (o **Run > Generate Javadoc** según versión).  
- Salida habitual: `target/site/apidocs` (Maven).

### 5.2. Proyectos Java SE no-Maven
- **Run > Generate Javadoc** y configurar la carpeta de salida.  
- Verifica el **JDK** del proyecto (Project Properties → Build → Compile).

---

## 6) Buenas prácticas y checklist

**Contenido y claridad**
- Describe **qué hace** el método, **qué espera** y **qué garantiza**.
- Documenta **precondiciones** (valores válidos) y **postcondiciones** (resultados).
- Cada parámetro debe tener su `@param` correspondiente; usa `@return` y `@throws` siempre que aplique.
- Usa `{@link Tipo#miembro}` para enlaces internos y `@see` para referencias externas/relevantes.

**Estilo y formato**
- Usa **oraciones completas** y lenguaje consistente.
- Evita copiar/pegar implementaciones: documenta el **contrato**, no el “cómo”.
- Prefiere `{@code ...}` para literales y `pre` para bloques de código formateado.
- Mantén la documentación actualizada cuando cambie la API.

**Cobertura**
- Documenta **clases públicas** y **métodos de la API**.  
- Considera documentar `package-info.java` con visión general y dependencias claves.

**Validación**
- Genera Javadoc en CI (GitHub Actions/Jenkins) para fallar si hay errores de doc.  
- Enlaza a Javadoc del JDK con `-link` o `<links>` para mejorar navegación.

**Revisiones**
- Revisa advertencias de Javadoc (métodos sin `@param`/`@return`, enlaces rotos, etc.).
- Asegura encoding UTF‑8 en código y plugin para evitar caracteres mal renderizados.

---

## 7) Ejemplo mínimo completo

**Archivo:** `src/main/java/com/mycompany/pagos/Calculadora.java`
```java
package com.mycompany.pagos;

/**
 * Utilidad aritmética para operaciones básicas del módulo de Pagos.
 * <p>Provee métodos seguros para suma y resta con control de overflow.</p>
 * @since 1.0
 */
public class  Calculadora {

    /**
     * Suma dos enteros usando {@link Math#addExact(int, int)}.
     *
     * @param a primer sumando
     * @param b segundo sumando
     * @return la suma de a y b
     * @throws ArithmeticException si se produce un overflow aritmético
     */
    public int sumar(int a, int b) {
        return Math.addExact(a, b);
    }

    /**
     * Resta dos enteros usando {@link Math#subtractExact(int, int)}.
     *
     * @param a minuendo
     * @param b sustraendo
     * @return la resta de a menos b
     * @throws ArithmeticException si se produce un overflow aritmético
     */
    public int restar(int a, int b) {
        return Math.subtractExact(a, b);
    }
}
```

**Comando JDK:**
```bash
javadoc -d docs/api -sourcepath src/main/java -subpackages com.mycompany.pagos \
  -encoding UTF-8 -docencoding UTF-8 -charset UTF-8 \
  -link https://docs.oracle.com/en/java/javase/17/docs/api/
```

**Maven:**
```bash
mvn clean javadoc:javadoc
```

---

## 8) Integración en CI (opcional rápido)

**GitHub Actions** (workflow mínimo):
```yaml
name: build-and-docs
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - name: Build & Test
        run: mvn -B clean verify
      - name: Generate Javadoc
        run: mvn -B javadoc:javadoc
      - name: Archive Javadoc
        uses: actions/upload-artifact@v4
        with:
          name: apidocs
          path: target/site/apidocs
```

---

### Conclusión
- **Documenta mientras desarrollas**: escribir Javadoc temprano reduce deuda técnica.
- **Automatiza la generación** con Maven/CI.
- **Expón contratos claros**: parámetros, retorno, excepciones y enlaces útiles.
- **Publica la Javadoc** junto con tus artefactos para una distribución profesional.
