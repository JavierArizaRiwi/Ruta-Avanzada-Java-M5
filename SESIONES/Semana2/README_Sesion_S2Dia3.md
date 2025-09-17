# Actividad Avanzada – Sistema de Gestión Académica con POO + Arrays

Duración estimada: 3 horas  
Nivel: Intermedio – Avanzado  

---

## Objetivo
Construir un sistema en Java que gestione estudiantes, profesores y cursos, aplicando los 4 principios de la POO y usando arrays para almacenar datos.

---

## Enunciado
Se pide implementar un sistema en Java que maneje estudiantes, profesores y cursos, cumpliendo con los siguientes requerimientos:

1. **Registrar estudiantes y profesores** (cada uno con sus atributos específicos).
2. **Asignar estudiantes a cursos** y cada curso debe estar dirigido por un profesor.
3. Permitir gestionar notas de estudiantes y calcular:
   - Promedio individual.
   - Promedio del curso.
4. Implementar búsquedas en arrays (por id, nombre, curso).
5. Aplicar polimorfismo para mostrar información detallada de personas (estudiantes/profesores).
6. Incluir al menos una clase abstracta y una interfaz.

---

## Modelado (UML textual)

- **Clase abstracta Persona**  
  - `String nombre`  
  - `int edad`  
  - `abstract void mostrarInfo();`  

- **Clase Estudiante extends Persona**  
  - `int id`  
  - `double[] notas`  
  - Métodos: `setNota()`, `calcularPromedio()`, `mostrarInfo()`  

- **Clase Profesor extends Persona**  
  - `String especialidad`  
  - `mostrarInfo()`  

- **Interfaz Gestionable**  
  - `void agregar(Object o)`  
  - `void listar()`  
  - `Object buscar(int id)`  

- **Clase Curso implements Gestionable**  
  - `String nombreCurso`  
  - `Profesor profesor`  
  - `Estudiante[] estudiantes`  
  - `int contador`  
  - Métodos: `agregar(Estudiante)`, `listar()`, `buscar(int id)`, `promedioCurso()`  

---

## Plan de trabajo (3 horas)

### Primera hora (POO + Setup)
- Crear `Persona`, `Estudiante`, `Profesor`.
- Encapsulación + constructores + polimorfismo con `mostrarInfo()`.

### Segunda hora (Arrays + Gestión de cursos)
- Crear `Curso` con array de `Estudiante`.
- Implementar interfaz `Gestionable`.
- Métodos: `agregar`, `listar`, `buscar`, `promedioCurso`.

### Tercera hora (Integración + Reto)
- Clase `Main` que:
  - Cree profesores, estudiantes y cursos.
  - Asigne estudiantes y calcule promedios.
  - Liste estudiantes y muestre info polimórfica.

---

## Código base (starter)

```java
abstract class Persona {
    private String nombre;
    private int edad;

    public Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }

    public String getNombre() { return nombre; }
    public int getEdad() { return edad; }

    public abstract void mostrarInfo();
}

class Estudiante extends Persona {
    private int id;
    private double[] notas;

    public Estudiante(String nombre, int edad, int id, int numNotas) {
        super(nombre, edad);
        this.id = id;
        this.notas = new double[numNotas];
    }

    public void setNota(int index, double nota) {
        if (index >= 0 && index < notas.length) {
            notas[index] = nota;
        }
    }

    public double calcularPromedio() {
        double suma = 0;
        for (double n : notas) suma += n;
        return notas.length > 0 ? suma / notas.length : 0;
    }

    public int getId() { return id; }

    @Override
    public void mostrarInfo() {
        System.out.println("Estudiante: " + getNombre() +
            " (ID: " + id + "), Promedio: " + calcularPromedio());
    }
}

class Profesor extends Persona {
    private String especialidad;

    public Profesor(String nombre, int edad, String especialidad) {
        super(nombre, edad);
        this.especialidad = especialidad;
    }

    @Override
    public void mostrarInfo() {
        System.out.println("Profesor: " + getNombre() +
            " | Especialidad: " + especialidad);
    }
}

interface Gestionable {
    void agregar(Object o);
    void listar();
    Object buscar(int id);
}

class Curso implements Gestionable {
    private String nombreCurso;
    private Profesor profesor;
    private Estudiante[] estudiantes;
    private int contador;

    public Curso(String nombreCurso, Profesor profesor, int capacidad) {
        this.nombreCurso = nombreCurso;
        this.profesor = profesor;
        this.estudiantes = new Estudiante[capacidad];
        this.contador = 0;
    }

    @Override
    public void agregar(Object o) {
        if (o instanceof Estudiante) {
            if (contador < estudiantes.length) {
                estudiantes[contador++] = (Estudiante) o;
            }
        }
    }

    @Override
    public void listar() {
        System.out.println("Curso: " + nombreCurso);
        profesor.mostrarInfo();
        for (int i = 0; i < contador; i++) {
            estudiantes[i].mostrarInfo();
        }
    }

    @Override
    public Object buscar(int id) {
        for (int i = 0; i < contador; i++) {
            if (estudiantes[i].getId() == id) return estudiantes[i];
        }
        return null;
    }

    public double promedioCurso() {
        double suma = 0;
        for (int i = 0; i < contador; i++) {
            suma += estudiantes[i].calcularPromedio();
        }
        return contador > 0 ? suma / contador : 0;
    }
}

public class Main {
    public static void main(String[] args) {
        Profesor prof = new Profesor("Carlos", 40, "POO");
        Curso curso = new Curso("Programación Orientada a Objetos", prof, 5);

        Estudiante e1 = new Estudiante("Ana", 20, 1, 3);
        e1.setNota(0, 4.0); e1.setNota(1, 3.5); e1.setNota(2, 4.2);

        Estudiante e2 = new Estudiante("Luis", 22, 2, 3);
        e2.setNota(0, 3.8); e2.setNota(1, 4.0); e2.setNota(2, 4.5);

        curso.agregar(e1);
        curso.agregar(e2);

        curso.listar();
        System.out.println("Promedio del curso: " + curso.promedioCurso());

        Estudiante buscado = (Estudiante) curso.buscar(2);
        if (buscado != null) buscado.mostrarInfo();
    }
}
