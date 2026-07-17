# Ruta Java: de cero a experto

Este repositorio es una ruta de aprendizaje progresiva para dominar Java: desde instalar el JDK y escribir el primer programa hasta diseñar aplicaciones por capas, trabajar con bases de datos, concurrencia y el ecosistema empresarial.

> Versión base recomendada: **Java 21 LTS** para seguir todos los ejemplos estables. Usa **Java 17 LTS** si tu entorno lo exige y estudia **Java 25 LTS** como evolución actual. Cada guía indica cuándo una característica necesita una versión concreta.

## Cómo estudiar este repositorio

1. Sigue la [ruta de aprendizaje completa](RUTA_APRENDIZAJE.md) en orden.
2. Escribe y ejecuta los ejemplos; leer código no sustituye practicarlo.
3. Resuelve los ejercicios sin mirar una solución y después refactoriza.
4. Conserva un proyecto integrador que evolucione con cada nivel.
5. No avances si no puedes explicar el tema con tus propias palabras.

## Mapa rápido

| Nivel | Resultado esperado | Documentos principales |
|---|---|---|
| 0. Preparación | Entender JDK, JVM, terminal, Maven y el ciclo del código | [Fundamentos](COMPLEMENTOS/README_Fundamentos_Java.md), [Sesión 1](SESIONES/Semana1/README_Sesion_S1Dia1.md) |
| 1. Lenguaje | Usar tipos, operadores, control de flujo, métodos, arrays y colecciones | [Fundamentos](COMPLEMENTOS/README_Fundamentos_Java.md), [Arrays y colecciones](COMPLEMENTOS/README_Arrays.md) |
| 2. Orientación a objetos | Modelar objetos con invariantes, composición, interfaces y polimorfismo | [POO](SESIONES/Semana2/README_Sesion_S2Dia2.md), [Interfaces](SESIONES/Semana4/README_Sesion_S4Dia1.md) |
| 3. Java funcional | Dominar lambdas, `Optional`, Streams y collectors | [Streams y programación funcional](COMPLEMENTOS/README_Streams_Java.md) |
| 4. Java moderno | Elegir entre clases, records, enums y tipos sellados; entender versiones | [Records y Java moderno](COMPLEMENTOS/README_Records_JavaModerno.md), [Versiones de Java](COMPLEMENTOS/README_Versiones_Java.md) |
| 5. Calidad y datos | Manejar errores, archivos, pruebas, Maven y JDBC | [Pruebas](COMPLEMENTOS/README_Pruebas_Java.md), [Excepciones](COMPLEMENTOS/README_ManejoExcepciones.md), [JDBC](SESIONES/Semana3/README_Sesion_S3Dia2.md) |
| 6. Diseño | Aplicar SOLID, patrones, MVC y arquitectura por capas | [Patrones](SESIONES/Semana3/README_Sesion_S3Dia1.md), [Arquitectura final](FINAL/README_Final_Arquitectura.md) |
| 7. Experto | Razonar sobre concurrencia, rendimiento y plataformas empresariales | [Concurrencia](COMPLEMENTOS/README_Hilos_y_Concurrencia.md), [Puente a Spring Boot](COMPLEMENTOS/README_Puente_SpringBoot.md) |

## Proyecto integrador

La ruta usa un sistema académico como hilo conductor. El proyecto evoluciona así:

```text
Consola y variables
  -> clases y objetos
  -> colecciones y Streams
  -> arquitectura por capas
  -> persistencia JDBC
  -> pruebas y manejo de errores
  -> aplicación de escritorio / API empresarial
```

Al terminar deberías poder justificar cada decisión: por qué una colección y no otra, por qué composición en lugar de herencia, dónde termina una transacción, cuándo un Stream mejora la intención y cuándo un bucle es más claro.

## Índices especializados

- [Ruta curricular y checklist de dominio](RUTA_APRENDIZAJE.md)
- [Java 8 a 25: versiones, LTS y compatibilidad](COMPLEMENTOS/README_Versiones_Java.md)
- [Java 17 vs Java 21: comparación enfocada](COMPLEMENTOS/README_Java17_vs_Java21.md)
- [Documentación con Javadoc](COMPLEMENTOS/README_DocumentacionJavaDoc.md)
- [Pruebas con JUnit](COMPLEMENTOS/README_Pruebas_Java.md)
- [Generar un JAR ejecutable](COMPLEMENTOS/README_GenerarEjecutable.md)
- [DTO, Mapper, Repository y Service](COMPLEMENTOS/README_Componentes_JAVA.md)
- [Java SE, Jakarta EE y Spring Boot](COMPLEMENTOS/README_JAVAEE_SPRINGB.md)

## Convenciones didácticas

- Los ejemplos base se escriben para Java 21, salvo que se indique otra versión.
- “Java” es el lenguaje y la plataforma; “JDK” es el kit de desarrollo; “JVM” ejecuta bytecode.
- `Stream` significa la API funcional de colecciones; no debe confundirse con streams de entrada/salida como `InputStream`.
- Un `record` ofrece inmutabilidad superficial: sus componentes no se reasignan, pero un componente mutable todavía puede cambiar internamente.
- La herencia se usa para relaciones de subtipo reales; la reutilización por sí sola no es motivo suficiente.

## Estado de la ruta

La documentación original se conserva y se integra como sesiones y profundizaciones. La ruta central corrige el orden pedagógico y distingue contenido fundamental, moderno, empresarial y legado. Consulta la sección “Mapa del contenido existente” en [RUTA_APRENDIZAJE.md](RUTA_APRENDIZAJE.md) para saber cuándo estudiar cada archivo.
