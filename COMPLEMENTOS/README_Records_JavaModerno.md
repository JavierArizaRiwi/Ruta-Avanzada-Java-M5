# Records, enums, sealed classes y modelado moderno

## 1. No todo objeto necesita una clase tradicional

Elige el tipo según su significado:

| Construcción | Úsala cuando |
|---|---|
| Clase | Hay identidad, estado mutable controlado, ciclo de vida o API desacoplada de la representación |
| `record` | El valor se define transparentemente por todos sus componentes |
| `enum` | Existe un conjunto cerrado de instancias conocidas |
| `sealed class/interface` | Existe una familia cerrada de subtipos, cada uno con datos diferentes |
| Interfaz | Necesitas expresar un rol o contrato que múltiples tipos pueden cumplir |

Estas herramientas no compiten entre sí. Un record puede implementar interfaces; una interfaz sellada puede ser implementada por records; un enum puede tener estado y comportamiento.

## 2. Qué genera un record

Los records quedaron como característica final en Java 16. Esta declaración:

```java
public record Coordenada(double latitud, double longitud) {}
```

declara componentes privados finales y genera:

- constructor canónico `new Coordenada(latitud, longitud)`;
- accesores `latitud()` y `longitud()` —no `getLatitud()`—;
- `equals` y `hashCode` basados en todos los componentes;
- `toString` con nombre y valores.

Un record es final implícitamente y extiende `java.lang.Record`; no puede extender otra clase. Sí puede implementar interfaces.

```java
public record Email(String valor) implements Comparable<Email> {
    @Override
    public int compareTo(Email otro) {
        return valor.compareToIgnoreCase(otro.valor);
    }
}
```

## 3. Constructor compacto e invariantes

El constructor compacto valida o normaliza antes de asignar automáticamente los componentes:

```java
public record Email(String valor) {
    public Email {
        Objects.requireNonNull(valor, "El email es obligatorio");
        valor = valor.trim().toLowerCase(Locale.ROOT);
        if (!valor.contains("@")) {
            throw new IllegalArgumentException("Email inválido: " + valor);
        }
    }
}
```

La asignación `valor = ...` modifica el parámetro del constructor; al finalizar, Java lo asigna al campo. En el constructor compacto no escribas `this.valor = valor`.

También puedes declarar un constructor alternativo, pero debe delegar al canónico:

```java
public record Dinero(BigDecimal monto, Currency moneda) {
    public Dinero {
        Objects.requireNonNull(monto);
        Objects.requireNonNull(moneda);
        monto = monto.setScale(2, RoundingMode.HALF_EVEN);
    }

    public Dinero(String monto, String codigoMoneda) {
        this(new BigDecimal(monto), Currency.getInstance(codigoMoneda));
    }
}
```

## 4. Inmutabilidad superficial

Los componentes no se pueden reasignar, pero un objeto mutable referenciado todavía puede cambiar:

```java
public record Curso(List<String> estudiantes) {}

List<String> nombres = new ArrayList<>();
Curso curso = new Curso(nombres);
nombres.add("Ana"); // curso.estudiantes() ahora también observa "Ana"
```

Haz copia defensiva:

```java
public record Curso(List<String> estudiantes) {
    public Curso {
        estudiantes = List.copyOf(estudiantes);
    }
}
```

`List.copyOf` crea una vista no modificable independiente de cambios estructurales posteriores. La inmutabilidad profunda requiere que los propios elementos también sean inmutables o se copien.

Con arrays necesitas copiar al entrar **y** al salir porque el accesor generado devolvería el array interno:

```java
public record Firma(byte[] bytes) {
    public Firma {
        bytes = bytes.clone();
    }

    @Override
    public byte[] bytes() {
        return bytes.clone();
    }
}
```

## 5. Métodos y miembros permitidos

Un record puede tener métodos de instancia, métodos estáticos, campos estáticos e implementar interfaces.

```java
public record Rectangulo(double ancho, double alto) implements Figura {
    public Rectangulo {
        if (ancho <= 0 || alto <= 0) {
            throw new IllegalArgumentException("Dimensiones positivas requeridas");
        }
    }

    @Override
    public double area() {
        return ancho * alto;
    }
}
```

No puede declarar campos de instancia adicionales: el estado completo debe estar en el encabezado. Si necesitas estado oculto, caché mutable o una API que no revele representación, una clase normal suele ser mejor.

## 6. Casos buenos y malos

### Buenos candidatos

- DTOs de entrada y salida.
- Objetos de valor: email, coordenada, rango de fechas, dinero.
- Claves compuestas de mapas.
- Resultados de consultas o cálculos.
- Eventos inmutables.

### Cuidado

- Entidades JPA: los proveedores y proxies históricamente esperan clases no finales y constructor accesible; verifica soporte antes de usar records como entidades.
- Secretos: `toString` generado expone componentes.
- Modelos cuya representación cambiará sin querer cambiar su API pública.
- Objetos con identidad y transición de estado, como un pedido con ciclo de vida complejo.
- Componentes mutables sin copia defensiva.

Un record no es “Lombok incorporado”. Es una declaración semántica: el estado completo y la API de acceso son transparentes.

## 7. Enums con comportamiento

Un enum representa instancias finitas y seguras por tipo:

```java
public enum EstadoPedido {
    CREADO {
        @Override public boolean puedeCancelar() { return true; }
    },
    PAGADO {
        @Override public boolean puedeCancelar() { return true; }
    },
    ENVIADO,
    ENTREGADO;

    public boolean puedeCancelar() { return false; }
}
```

Evita persistir `ordinal()`: cambia si reordenas constantes. Persiste un código estable explícito.

```java
public enum Rol {
    ADMIN("ADM"), ESTUDIANTE("EST");

    private final String codigo;

    Rol(String codigo) { this.codigo = codigo; }
    public String codigo() { return codigo; }
}
```

## 8. Tipos sellados

Las clases e interfaces selladas quedaron finales en Java 17. Restringen quién puede extender o implementar un tipo:

```java
public sealed interface ResultadoPago
        permits PagoAprobado, PagoRechazado, PagoPendiente {}

public record PagoAprobado(String autorizacion) implements ResultadoPago {}
public record PagoRechazado(String motivo) implements ResultadoPago {}
public record PagoPendiente(Instant proximoIntento) implements ResultadoPago {}
```

Cada subtipo permitido debe ser `final`, `sealed` o `non-sealed`. Si están en el mismo archivo o módulo, el compilador conoce la familia cerrada.

No selles una jerarquía solo para impedir extensión: sella cuando el dominio realmente tiene un conjunto controlado de alternativas.

## 9. Pattern matching y switch exhaustivo

Java 21 finalizó pattern matching para `switch` y record patterns. Una jerarquía sellada permite procesar todos los casos sin `default`:

```java
static String describir(ResultadoPago resultado) {
    return switch (resultado) {
        case PagoAprobado(var autorizacion) -> "Aprobado: " + autorizacion;
        case PagoRechazado(var motivo) -> "Rechazado: " + motivo;
        case PagoPendiente(var instante) -> "Reintento: " + instante;
    };
}
```

El compilador comprueba exhaustividad. Si agregas otro subtipo permitido, los switches incompletos dejan de compilar: el cambio se vuelve visible.

Pattern matching para `instanceof` evita cast manual:

```java
if (figura instanceof Rectangulo r && r.area() > 100) {
    System.out.println(r.ancho());
}
```

## 10. Modelar éxito y error sin `null`

Una suma de tipos sellada puede representar resultados esperados:

```java
sealed interface Resultado<T> permits Exito, Fallo {}

record Exito<T>(T valor) implements Resultado<T> {}
record Fallo<T>(String codigo, String mensaje) implements Resultado<T> {}
```

No reemplaces automáticamente todas las excepciones con este patrón. Úsalo cuando éxito y rechazo son resultados normales que el llamador debe considerar; conserva excepciones para fallos excepcionales o violaciones del contrato.

## 11. Migrar de POJO a record

Antes:

```java
public final class EstudianteDto {
    private final Long id;
    private final String nombre;

    // constructor, getters, equals, hashCode, toString...
}
```

Después:

```java
public record EstudianteDto(Long id, String nombre) {
    public EstudianteDto {
        Objects.requireNonNull(nombre);
        nombre = nombre.trim();
    }
}
```

La migración cambia `getNombre()` por `nombre()` y puede afectar frameworks, serialización y compatibilidad binaria. Revisa consumidores antes de cambiar una API pública.

## 12. Ejercicios

1. Modela `Dinero` evitando escala inconsistente y moneda nula.
2. Crea una jerarquía sellada para `Notificacion`: email, SMS y push.
3. Procesa cada notificación con un `switch` exhaustivo.
4. Demuestra con una prueba por qué un record con `List` no es profundamente inmutable; corrígelo.
5. Compara un `Pedido` mutable con un `PedidoDto` record y justifica cada elección.

## Checklist

- [ ] Sé qué miembros genera un record.
- [ ] Valido invariantes en un constructor compacto.
- [ ] Hago copias defensivas de componentes mutables.
- [ ] No confundo record con entidad ni con inmutabilidad profunda.
- [ ] Uso enum para instancias finitas y sealed para familias finitas de tipos.
- [ ] Aprovecho switch exhaustivo cuando el dominio es cerrado.

