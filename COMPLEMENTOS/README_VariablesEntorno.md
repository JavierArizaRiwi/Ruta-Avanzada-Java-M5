# Uso de Variables de Entorno en Java (Linux)

En aplicaciones Java, es una buena práctica no exponer credenciales directamente en el código fuente.  
En su lugar, se utilizan variables de entorno que se configuran en el sistema operativo y luego se leen desde la aplicación.

---

## 1. Definir Variables de Entorno en Linux

En una terminal, exporta las variables:

```bash
export DB_URL="jdbc:mysql://localhost:3306/academia"
export DB_USER="root"
export DB_PASS="JavyIngeniero2025."
```

Estas variables solo estarán disponibles en la sesión actual de la terminal.  
Si cierras la sesión o la terminal, desaparecerán.

### Persistir en el sistema
Si deseas que estén siempre disponibles al iniciar sesión, agrégalas al archivo `~/.bashrc` o `~/.zshrc`:

```bash
# Variables para la conexión a MySQL
export DB_URL="jdbc:mysql://localhost:3306/academia"
export DB_USER="root"
export DB_PASS="JavyIngeniero2025."
```

Después, recarga el archivo:

```bash
source ~/.bashrc
```

---

## 2. Consultar Variables de Entorno en Linux

Para verificar si están definidas:

```bash
echo $DB_URL
echo $DB_USER
echo $DB_PASS
```

Para listar todas las variables activas:

```bash
printenv
```

---

## 3. Acceder a Variables de Entorno en Java

En Java, se usan los métodos de la clase `System`:

```java
package com.mycompany.academic.db;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConexionDB {
    private static final String URL  = System.getenv("DB_URL");
    private static final String USER = System.getenv("DB_USER");
    private static final String PASS = System.getenv("DB_PASS");

    public static Connection getConexion() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASS);
    }
}
```

### Explicación
- `System.getenv("DB_URL")`: obtiene el valor de la variable `DB_URL` definida en el sistema.
- `System.getenv("DB_USER")`: obtiene el usuario.
- `System.getenv("DB_PASS")`: obtiene la contraseña.

---

## 4. Ejecución en Linux

Al compilar y ejecutar, asegúrate de que las variables estén exportadas:

```bash
# Compilar
javac -d out src/com/mycompany/academic/db/ConexionDB.java

# Ejecutar (dentro del classpath de out)
java -cp out com.mycompany.academic.db.ConexionDB
```

---

## 5. Buenas Prácticas

- No subir credenciales a GitHub ni a repositorios.
- Usar variables de entorno o un archivo `.env` (con librerías externas como dotenv-java).
- Separar las configuraciones por ambientes:
  - `DB_URL_DEV` (desarrollo)
  - `DB_URL_TEST` (pruebas)
  - `DB_URL_PROD` (producción)

Ejemplo de conexión por ambiente:

```bash
export DB_URL_DEV="jdbc:mysql://localhost:3306/academia_dev"
export DB_URL_PROD="jdbc:mysql://prod-server:3306/academia"
```

En Java:

```java
String env = System.getenv("APP_ENV"); // "DEV" o "PROD"
String url = System.getenv("DB_URL_" + env);
```

---

Con esto, tu aplicación queda más segura, portable y fácil de mantener.