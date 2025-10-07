# Arquitectura Java EE clásica con JSF, ServiceLocal, ServiceImpl y Facade (JPA)

Este documento explica, paso a paso, cómo estructurar un proyecto Java EE (Jakarta EE) multi-módulo con JSF en la capa web, una capa de negocio basada en interfaces ServiceLocal, su implementación EJB @Stateless, y una capa Facade que accede directamente a JPA mediante EntityManager (sin patrón DAO). Incluye responsabilidades de cada capa, anotaciones clave, ejemplo de código y empaquetado EAR para despliegue en WildFly/JBoss.

---

## 1. Visión general de la arquitectura

```
JSF (xhtml) → ManagedBean (web) → ServiceLocal (api) → ServiceImpl @Stateless (ejb) → Facade @Stateless (ejb) → JPA (EntityManager) → BD
                       |                               ↑
                       |________ DTOs / Excepciones ___|
```

- web (WAR): vistas JSF (Facelets) y ManagedBeans CDI/JSF. Solo conoce contratos (interfaces ServiceLocal) y modelos de intercambio (DTOs, Excepciones) expuestos en un JAR de API.
- api (JAR): interfaces de negocio (ServiceLocal), DTOs y excepciones. Es el contrato estable entre web y negocio.
- ejb (EJB-JAR): implementa las interfaces del API. Incluye la lógica de negocio en ServiceImpl y el acceso a datos en Facade usando JPA (EntityManager).
- ear (EAR): ensambla web.war, ejb.jar y api.jar en un único artefacto desplegable en el servidor de aplicaciones.

---

## 2. Estructura multi-módulo (Maven + EAR)

```
raiz-proyecto/
├─ pom.xml                    (packaging: pom)  ← padre
├─ api/                       (packaging: jar)  ← ServiceLocal + DTOs + Excepciones
├─ ejb/                       (packaging: ejb)  ← ServiceImpl + Facade + JPA + persistence.xml
├─ web/                       (packaging: war)  ← JSF/Facelets + ManagedBeans
└─ ear/                       (packaging: ear)  ← ensambla WAR + EJB + API
```

### 2.1. Responsabilidades por módulo
- api/: contrato estable. Lo consume web/ y lo implementa ejb/. Versionarlo bien minimiza roturas.
- ejb/: lógica de negocio (ServiceImpl) y acceso a datos (Facade con EntityManager). Transacciones CMT por defecto.
- web/: solo orquesta y presenta. No accede a JPA ni a entidades del dominio; usa DTOs.
- ear/: empaquetado final para WildFly/JBoss.

---

## 3. Anotaciones y conceptos clave

### 3.1. EJB y CDI
- @Stateless: bean sin estado. Transacción REQUIRED por defecto (CMT).
- @Local: interfaz visible dentro del mismo EAR/JVM.
- @Remote: interfaz accesible vía RMI/IIOP desde otra JVM (opcional).
- @EJB: inyección de EJBs (la forma clásica en Java EE).
- @Inject: inyección CDI (si integras EJBs con CDI vía productores o beans compatibles).
- @TransactionAttribute: ajusta la semántica transaccional (ej.: REQUIRES_NEW, MANDATORY).
- @PersistenceContext: inyecta un EntityManager JPA gestionado por el contenedor.

### 3.2. JSF y CDI (capa web)
- @Named: expone el bean para EL en JSF (#{bean}).
- @RequestScoped, @ViewScoped, @SessionScoped: ciclo de vida del ManagedBean (recomendado @ViewScoped para páginas con postbacks).
- Facelets .xhtml: plantillas de vista con componentes JSF.

### 3.3. JPA
- @Entity, @Table, @Id, @GeneratedValue: mapeo ORM.
- @Column, @OneToMany, @ManyToOne, etc.: relaciones y columnas.
- EntityManager: API de persistencia (persist, merge, find, remove, JPQL/Criteria).

---

## 4. API (JAR): contratos y modelos de intercambio

Interface de negocio (ServiceLocal)
api/src/main/java/com/empresa/app/api/UsuarioServiceLocal.java
```java
package com.empresa.app.api;

import java.util.List;
import javax.ejb.Local;

@Local
public interface UsuarioServiceLocal {
    List<UsuarioDTO> listar();
    Long crear(UsuarioDTO dto) throws NegocioException;
    void actualizar(Long id, UsuarioDTO dto) throws NegocioException;
    void eliminar(Long id) throws NegocioException;
}
```

DTO y excepción
api/.../UsuarioDTO.java
```java
package com.empresa.app.api;
import java.io.Serializable;

public class UsuarioDTO implements Serializable {
  private Long id;
  private String nombre;
  private String email;
  // getters/setters
}
```
api/.../NegocioException.java
```java
package com.empresa.app.api;
public class NegocioException extends Exception {
  public NegocioException(String msg){ super(msg); }
}
```

Sugerencia: mantén DTOs independientes de entidades JPA (evita exponer lazy fields y acoplar la vista al dominio).

---

## 5. EJB (EJB-JAR): implementación de negocio y acceso JPA

### 5.1. Entidad JPA
ejb/src/main/java/com/empresa/app/ejb/entity/Usuario.java
```java
package com.empresa.app.ejb.entity;

import javax.persistence.*;

@Entity @Table(name = "usuarios")
public class Usuario {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  private String nombre;
  private String email;
  // getters/setters
}
```

### 5.2. Facade: acceso directo a JPA (sin DAO)
ejb/.../facade/UsuarioFacade.java
```java
package com.empresa.app.ejb.facade;

import com.empresa.app.ejb.entity.Usuario;
import javax.ejb.Stateless;
import javax.persistence.*;
import java.util.List;

@Stateless
public class UsuarioFacade {

  @PersistenceContext(unitName = "appPU")
  private EntityManager em;

  public List<Usuario> findAll() {
    return em.createQuery("SELECT u FROM Usuario u", Usuario.class).getResultList();
  }

  public Long create(Usuario u) {
    em.persist(u);
    return u.getId();
  }

  public void update(Long id, Usuario data) {
    Usuario u = em.find(Usuario.class, id);
    if (u == null) throw new EntityNotFoundException();
    u.setNombre(data.getNombre());
    u.setEmail(data.getEmail());
  }

  public void delete(Long id) {
    Usuario u = em.find(Usuario.class, id);
    if (u != null) em.remove(u);
  }

  public Usuario find(Long id) { return em.find(Usuario.class, id); }
}
```

### 5.3. ServiceImpl: lógica de negocio y orquestación
ejb/.../service/UsuarioServiceBean.java
```java
package com.empresa.app.ejb.service;

import com.empresa.app.api.*;
import com.empresa.app.ejb.entity.Usuario;
import com.empresa.app.ejb.facade.UsuarioFacade;

import javax.ejb.EJB;
import javax.ejb.Stateless;
import java.util.List;
import java.util.stream.Collectors;

@Stateless
public class UsuarioServiceBean implements UsuarioServiceLocal {

  @EJB
  private UsuarioFacade facade;

  @Override
  public List<UsuarioDTO> listar() {
    return facade.findAll().stream().map(this::toDTO).collect(Collectors.toList());
  }

  @Override
  public Long crear(UsuarioDTO dto) throws NegocioException {
    validar(dto);
    return facade.create(toEntity(dto));
  }

  @Override
  public void actualizar(Long id, UsuarioDTO dto) throws NegocioException {
    validar(dto);
    facade.update(id, toEntity(dto));
  }

  @Override
  public void eliminar(Long id) throws NegocioException {
    facade.delete(id);
  }

  // ---- helpers
  private void validar(UsuarioDTO d) throws NegocioException {
    if (d.getNombre() == null || d.getNombre().isBlank())
      throw new NegocioException("Nombre requerido");
  }
  private UsuarioDTO toDTO(Usuario u){
    UsuarioDTO d = new UsuarioDTO();
    d.setId(u.getId()); d.setNombre(u.getNombre()); d.setEmail(u.getEmail());
    return d;
  }
  private Usuario toEntity(UsuarioDTO d){
    Usuario u = new Usuario();
    u.setNombre(d.getNombre()); u.setEmail(d.getEmail());
    return u;
  }
}
```

### 5.4. persistence.xml (JTA + DataSource del servidor)
ejb/src/main/resources/META-INF/persistence.xml
```xml
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.2">
  <persistence-unit name="appPU" transaction-type="JTA">
    <jta-data-source>java:jboss/datasources/AppDS</jta-data-source>
    <class>com.empresa.app.ejb.entity.Usuario</class>
    <properties>
      <property name="hibernate.show_sql" value="false"/>
      <property name="hibernate.format_sql" value="true"/>
      <property name="hibernate.hbm2ddl.auto" value="validate"/>
    </properties>
  </persistence-unit>
</persistence>
```

- jta-data-source: nombre JNDI del DataSource configurado en WildFly/JBoss.
- hbm2ddl.auto: en producción, usa validate o none. Para desarrollo podrías usar update con cuidado.

---

## 6. Web (WAR): JSF + ManagedBean (inyecta ServiceLocal)

web/src/main/java/com/empresa/app/web/mb/UsuarioMB.java
```java
package com.empresa.app.web.mb;

import com.empresa.app.api.*;
import javax.ejb.EJB;
import javax.enterprise.context.ViewScoped;
import javax.inject.Named;
import java.io.Serializable;
import java.util.List;

@Named("usuarioMB")
@ViewScoped
public class UsuarioMB implements Serializable {

  @EJB
  private UsuarioServiceLocal service; // contrato del JAR api

  private List<UsuarioDTO> lista;
  private UsuarioDTO actual = new UsuarioDTO();

  public void init(){ lista = service.listar(); }

  public void guardar(){
    try {
      if (actual.getId() == null) service.crear(actual);
      else service.actualizar(actual.getId(), actual);
      init();
      actual = new UsuarioDTO();
    } catch (NegocioException e) { /* FacesMessage */ }
  }

  public void eliminar(Long id){
    try { service.eliminar(id); init(); } catch (NegocioException e) { /* FacesMessage */ }
  }

  // getters/setters
}
```

Vista Facelets (fragmento)
web/src/main/webapp/usuario.xhtml
```xhtml
<f:metadata><f:viewAction action="#{usuarioMB.init}" /></f:metadata>

<h:form>
  <h:panelGrid columns="2">
    <h:outputLabel value="Nombre"/>
    <h:inputText value="#{usuarioMB.actual.nombre}" required="true" />

    <h:outputLabel value="Email"/>
    <h:inputText value="#{usuarioMB.actual.email}" />
  </h:panelGrid>

  <h:commandButton value="Guardar" action="#{usuarioMB.guardar}" />
</h:form>

<h:dataTable value="#{usuarioMB.lista}" var="u">
  <h:column><f:facet name="header">Nombre</f:facet>#{u.nombre}</h:column>
  <h:column><f:facet name="header">Email</f:facet>#{u.email}</h:column>
  <h:column>
    <h:commandLink value="Eliminar" action="#{usuarioMB.eliminar(u.id)}"/>
  </h:column>
</h:dataTable>
```

Scopes recomendados
- @ViewScoped: ideal para pantallas con postbacks múltiples dentro de la misma vista.
- @RequestScoped: cada petición crea un bean nuevo; útil para acciones simples.
- @SessionScoped: mantiene estado a nivel de sesión (usar con moderación).

---

## 7. POMs y empaquetado EAR

7.1. raiz/pom.xml (padre)
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <packaging>pom</packaging>
  <modules>
    <module>api</module>
    <module>ejb</module>
    <module>web</module>
    <module>ear</module>
  </modules>
</project>
```

7.2. web/pom.xml (depende de api)
```xml
<dependency>
  <groupId>com.empresa</groupId>
  <artifactId>api</artifactId>
  <version>${project.version}</version>
</dependency>
```

7.3. ejb/pom.xml (depende de api)
```xml
<dependency>
  <groupId>com.empresa</groupId>
  <artifactId>api</artifactId>
  <version>${project.version}</version>
</dependency>
```

7.4. ear/pom.xml (ensambla WAR + EJB + API)
```xml
<packaging>ear</packaging>
<dependencies>
  <dependency><groupId>com.empresa</groupId><artifactId>ejb</artifactId><type>ejb</type></dependency>
  <dependency><groupId>com.empresa</groupId><artifactId>web</artifactId><type>war</type></dependency>
  <dependency><groupId>com.empresa</groupId><artifactId>api</artifactId><type>jar</type></dependency>
</dependencies>
<build>
  <plugins>
    <plugin>
      <artifactId>maven-ear-plugin</artifactId>
      <configuration>
        <modules>
          <webModule><groupId>com.empresa</groupId><artifactId>web</artifactId><contextRoot>/app</contextRoot></webModule>
          <ejbModule><groupId>com.empresa</groupId><artifactId>ejb</artifactId></ejbModule>
          <jarModule><groupId>com.empresa</groupId><artifactId>api</artifactId></jarModule>
        </modules>
      </configuration>
    </plugin>
  </plugins>
</build>
```

Build y deploy
```bash
mvn clean install
# ear/target/*.ear → copiar a wildfly/standalone/deployments/
```

---

## 8. JNDI y resolución de EJB

- Dentro del mismo EAR, basta @EJB UsuarioServiceLocal service; en el WAR.
- Para clientes remotos (otra JVM), usa @Remote y el nombre JNDI global (p.ej. java:global/ear/ejb/UsuarioServiceBean!com.empresa.app.api.UsuarioServiceRemote).

---

## 9. Buenas prácticas y evolución

1. Contratos estables: evitar cambios rupturistas en api. Si es necesario, versiona (api-v2) y migra gradualmente.
2. DTOs claros: no exponer entidades en la vista. Evita problemas de lazy y acoplamientos.
3. Transacciones: mantenerlas en ServiceImpl (@Stateless CMT). El Facade participa en la misma transacción.
4. Validaciones: valida en ServiceImpl (y en la vista). Considera Bean Validation (@NotNull, etc.) en DTOs/entidades.
5. Excepciones: separa NegocioException (errores esperados) de técnicos (PersistenceException).
6. Registro/Logs: usa SessionContext o logging para trazabilidad.
7. Configuración: DataSource en servidor, credenciales fuera del código. hbm2ddl.auto=validate en prod.
8. Pruebas: para integración, considera Arquillian (opcional) o pruebas con contenedor embebido.

---

## 10. Errores comunes a evitar
- Inyectar EJBs en clases no gestionadas por el contenedor (usa CDI o define beans apropiadamente).
- Exponer entidades JPA a la vista y disparar LazyInitializationException.
- Mezclar responsabilidades en el ManagedBean (la lógica debe vivir en ServiceImpl).
- Agotar la sesión por usar @SessionScoped para todo.
- Configurar mal el persistence.xml o duplicar JARs en WEB-INF/lib.

---

## 11. Checklist de despliegue (WildFly/JBoss)
1. Definir DataSource (AppDS) en la consola admin o standalone.xml.
2. Verificar persistence.xml (jta-data-source, clases, hbm2ddl.auto).
3. Empaquetar con mvn clean install (generar .ear).
4. Copiar ear/target/*.ear a standalone/deployments/.
5. Revisar el log del servidor para JNDI, migraciones y errores de arranque.
6. Probar la URL del WAR (/app) y las páginas JSF.

---

Con esta guía tienes un README de referencia listo para implementar y escalar la arquitectura JSF + ServiceLocal + ServiceImpl + Facade + JPA en Java EE.