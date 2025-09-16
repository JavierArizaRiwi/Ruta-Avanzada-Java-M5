# Los 4 Principios de la Programación Orientada a Objetos (POO)

La **Programación Orientada a Objetos (POO)** es un paradigma que organiza el software en torno a **objetos** y sus **interacciones**.  
Sus **4 principios fundamentales** son: **Abstracción, Encapsulación, Herencia y Polimorfismo**.

---

## 1. Abstracción

### Explicación
La abstracción consiste en **mostrar lo esencial** y ocultar los **detalles internos de implementación**.  
Se logra con **clases abstractas e interfaces**, donde definimos **qué debe hacer un objeto** sin necesidad de especificar **cómo lo hace**.

La abstracción ayuda a que el código sea **más legible, mantenible y flexible**, porque se centra en el **qué** en lugar del **cómo**.

### Casos de uso
- Frameworks y librerías: en JDBC trabajamos con interfaces (`Connection`, `Statement`) sin importar si la base es MySQL, PostgreSQL o Oracle.  
- Diseño de APIs: se define una interfaz y cada implementación concreta sus detalles.  
- Simulaciones: una interfaz `Figura` define `calcularArea()`, y cada figura (círculo, rectángulo, triángulo) lo implementa distinto.  

### Ejemplo en Java
```java
// Interfaz
interface ConexionBD {
    void conectar();
    void ejecutar(String sql);
    void desconectar();
}

// Implementación MySQL
class ConexionMySQL implements ConexionBD {
    @Override public void conectar() { System.out.println("Conectando a MySQL..."); }
    @Override public void ejecutar(String sql) { System.out.println("MySQL ejecuta: " + sql); }
    @Override public void desconectar() { System.out.println("Desconectando MySQL"); }
}

// Uso
public class DemoAbstraccion {
    public static void main(String[] args) {
        ConexionBD conexion = new ConexionMySQL();
        conexion.conectar();
        conexion.ejecutar("SELECT * FROM usuarios");
        conexion.desconectar();
    }
}
```

### Ejercicio propuesto
Crea una interfaz `Figura` con el método `double calcularArea()`. Implementa dos clases: `Circulo` y `Rectangulo`, cada una con su propia lógica para calcular el área.

---

## 2. Encapsulación

### Explicación
La encapsulación significa **proteger los datos internos** de un objeto y exponerlos solo mediante **métodos controlados** (`getters`, `setters`, operaciones específicas).  
Los atributos se marcan como `private` y se accede a ellos mediante métodos `public` o `protected`.

La encapsulación garantiza **seguridad, validaciones y consistencia** en los datos.

### Casos de uso
- Cuentas bancarias: no se modifica el saldo directamente, se usan métodos `depositar()` o `retirar()`.  
- Aplicaciones médicas: proteger datos sensibles de pacientes.  
- Configuración controlada: algunos parámetros solo se cambian bajo ciertas condiciones.  

### Ejemplo en Java
```java
class CuentaBancaria {
    private double saldo; // atributo privado

    public CuentaBancaria(double saldoInicial) {
        if (saldoInicial < 0) throw new IllegalArgumentException("Saldo inválido");
        this.saldo = saldoInicial;
    }

    public double getSaldo() { return saldo; }

    public void depositar(double monto) {
        if (monto > 0) saldo += monto;
    }

    public boolean retirar(double monto) {
        if (monto > 0 && monto <= saldo) {
            saldo -= monto;
            return true;
        }
        return false;
    }
}

// Uso
public class DemoEncapsulacion {
    public static void main(String[] args) {
        CuentaBancaria cuenta = new CuentaBancaria(1000);
        cuenta.depositar(500);
        cuenta.retirar(300);
        System.out.println("Saldo actual: " + cuenta.getSaldo());
    }
}
```

### Ejercicio propuesto
Crea una clase `Persona` con atributos privados `nombre` y `edad`. Implementa los métodos `getNombre()`, `setNombre()`, `getEdad()` y `setEdad(int edad)` validando que la edad no sea negativa.

---

## 3. Herencia

### Explicación
La herencia permite que una **subclase** reutilice atributos y métodos de una **superclase**.  
Fomenta la **reutilización de código**, la creación de **jerarquías** y la **especialización** de clases.

Representa la relación **“es un”** (ejemplo: un `Carro` **es un** `Vehiculo`).

En Java, la herencia es **simple** (una clase solo puede heredar de otra), pero se pueden implementar múltiples interfaces.

### Tipos de herencia

- **Herencia simple:** Una clase hereda de una sola clase base.  
  Ejemplo: `class Carro extends Vehiculo { ... }`
- **Herencia múltiple (no soportada directamente en Java):** Una clase hereda de varias clases base. Java lo evita para reducir ambigüedades, pero permite implementar varias interfaces.
- **Herencia multinivel:** Una clase hereda de otra que a su vez hereda de otra.  
  Ejemplo: `class Animal { } class Mamifero extends Animal { } class Perro extends Mamifero { }`
- **Herencia jerárquica:** Varias clases heredan de una misma clase base.  
  Ejemplo: `class Animal { } class Perro extends Animal { } class Gato extends Animal { }`

### Diferencia entre `extends` e `implements`

- `extends` se usa para heredar de una clase (herencia de implementación).
- `implements` se usa para implementar una o varias interfaces (herencia de comportamiento).

```java
class Vehiculo { ... }
class Carro extends Vehiculo { ... } // Herencia de clase

interface Volador { void volar(); }
class Pajaro implements Volador { public void volar() { ... } } // Implementación de interfaz
```

### Casos de uso
- Jerarquía de vehículos: `Vehiculo → Carro, Moto, Camión`.  
- Usuarios en sistemas: `Usuario → Cliente, Administrador`.  
- Sistemas gráficos: `ComponenteUI → Botón, Etiqueta, CheckBox`.  

### Ejemplo en Java
```java
class Vehiculo {
    protected String marca;

    public Vehiculo(String marca) { this.marca = marca; }

    public void encender() {
        System.out.println(marca + " encendido");
    }
}

class Carro extends Vehiculo {
    private int puertas;

    public Carro(String marca, int puertas) {
        super(marca);
        this.puertas = puertas;
    }

    public void mostrarInfo() {
        System.out.println("Marca: " + marca + ", Puertas: " + puertas);
    }
}

// Uso
public class DemoHerencia {
    public static void main(String[] args) {
        Carro c = new Carro("Toyota", 4);
        c.encender();      // método heredado
        c.mostrarInfo();   // método propio
    }
}
```

### Ejercicio propuesto
Crea una clase base `Empleado` con atributos `nombre` y `salario`. Luego crea una subclase `Gerente` que agregue el atributo `departamento` y un método para mostrar toda la información.

---

## 4. Polimorfismo

### Explicación
El polimorfismo permite que un mismo **método** tenga **diferentes comportamientos** según el objeto que lo invoque.

### Tipos de polimorfismo

- **Estático (Overloading):** Mismo nombre de método, diferentes parámetros en la misma clase.
- **Dinámico (Overriding):** Una subclase redefine un método de la superclase.
- **Paramétrico:** Uso de genéricos para trabajar con diferentes tipos de datos.
- **De inclusión (subtipos):** Un objeto de una subclase puede ser tratado como objeto de la superclase.

El polimorfismo hace que los programas sean **flexibles, extensibles y fáciles de mantener**.

### Casos de uso
- Animales: `hacerSonido()` se comporta distinto en `Perro`, `Gato` o `Vaca`.  
- Procesamiento de pagos: distintas clases implementan `Pago.procesar()`.  
- Colecciones genéricas: `List<String>` o `List<Integer>` funcionan igual.  

### Ejemplo en Java
```java
class Animal {
    public void hacerSonido() {
        System.out.println("Sonido genérico");
    }
}

class Perro extends Animal {
    @Override
    public void hacerSonido() {
        System.out.println("Guau");
    }
}

class Gato extends Animal {
    @Override
    public void hacerSonido() {
        System.out.println("Miau");
    }
}

// Uso
public class DemoPolimorfismo {
    public static void main(String[] args) {
        Animal a1 = new Perro();
        Animal a2 = new Gato();
        a1.hacerSonido(); // Guau
        a2.hacerSonido(); // Miau
    }
}
```

### Ejercicio propuesto
Crea una clase base `Instrumento` con un método `tocar()`. Implementa dos subclases `Guitarra` y `Piano` que sobrescriban el método para mostrar mensajes diferentes. Crea un arreglo de instrumentos y recorrelo llamando a `tocar()` en cada uno.

---

## Resumen comparativo

| Principio        | Definición                                       | Beneficio principal                | Caso típico                        |
|------------------|--------------------------------------------------|------------------------------------|------------------------------------|
| Abstracción      | Ocultar detalles internos y mostrar lo esencial  | Simplifica interacción con objetos | Interfaces (`List`, `Connection`)  |
| Encapsulación    | Ocultar datos y controlarlos mediante métodos    | Seguridad e integridad de datos    | Bancos, sistemas médicos           |
| Herencia         | Reutilizar atributos y métodos de una superclase | Reutilización y jerarquías         | Vehículos, usuarios                |
| Polimorfismo     | Diferentes comportamientos bajo misma interfaz   | Flexibilidad y extensibilidad      | Animales, pagos, colecciones       |

---

## Ejercicios adicionales

1. **Abstracción:**  
   Define una interfaz `Notificable` con el método `notificar(String mensaje)`. Implementa dos clases: `EmailNotificador` y `SMSNotificador`.

2. **Encapsulación:**  
   Crea una clase `Producto` con atributos privados `nombre` y `precio`. Agrega métodos para modificar el precio solo si es positivo.

3. **Herencia y polimorfismo:**  
   Crea una jerarquía de clases para `Figura` (`Circulo`, `Rectangulo`, `Triangulo`) y un método `calcularArea()` polimórfico.

---

**¡Practica estos conceptos en tus propios proyectos para dominar la Programación Orientada a Objetos!**