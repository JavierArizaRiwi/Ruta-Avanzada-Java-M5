# Ejercicio Práctico – Semana 5 
**Tema:** Cierre de Java SE y preparación para Spring Boot con arquitectura por capas, manejo de archivos y excepciones personalizadas  
**Código:** ST-M5.1-Sem4-Práctica  

---

## Objetivo  
Aplicar los conceptos de arquitectura por capas, JDBC avanzado, manejo de archivos y excepciones personalizadas desarrollando una miniaplicación **de registro y exportación de pedidos** que sirva como base para migrar luego a Spring Boot.

---

## Estructura de Carpetas
```bash
src/
 ├── model/
 ├── dao/
 ├── service/
 ├── controller/
 ├── util/
 └── Main.java
```

---

## 📋 Requisitos
1. Registrar pedidos con JDBC.
2. Listar pedidos desde base de datos.
3. Exportar pedidos a CSV.
4. Validar datos con excepciones personalizadas.
5. Implementar arquitectura por capas.
6. Separar claramente modelo, servicio, repositorio y presentación para que la migración a Spring Boot sea directa.
7. Preparar la lógica para sustituir JDBC manual por `JpaRepository` o `JdbcTemplate` más adelante.

---

**Entrega:** Código fuente con estructura modular y archivo CSV generado.
**Evaluación:** Correcta implementación de capas, validaciones, manejo de excepciones y claridad para migrar a Spring Boot.