# OWASP Top 10 - Demostracion de Incidentes de Ciberseguridad

Aplicacion academica desarrollada con Spring Boot para demostrar vulnerabilidades frecuentes y sus mitigaciones. El proyecto incluye una interfaz web local y colecciones de Bruno para reproducir los escenarios HTTP.

> **Uso exclusivamente educativo.** Los endpoints vulnerables estan pensados para ejecutarse en un entorno local y aislado.

## Contenido

- **Proyecto 1:** demostracion de sobreexposicion de datos y fuga de stacktrace por mala configuracion (OWASP A02).
- **Proyecto 2:** validacion de integridad de componentes de software y proveedor confiable en la cadena de suministro (OWASP A03).
- **Proyecto 3:** demostracion de XSS almacenado y del riesgo de keylogging, con salida escapada como mitigacion.

Las implementaciones activas son los proyectos 2, 3 y 4. La página `proyecto1.html` se conserva como material visual heredado, pero no tiene actualmente un backend Java ni una colección Bruno asociada.

## Tecnologias

- Java 17
- Spring Boot 3.3.4
- Spring Web
- Spring Data JPA / Hibernate
- MySQL como base de datos configurada por defecto
- H2 disponible como alternativa en memoria
- Maven
- Bruno para las pruebas HTTP

## Requisitos

- JDK 17 o superior
- Maven 3.8+ (o usar el Maven Wrapper si se agrega posteriormente)
- MySQL 8+ para la configuracion actual
- Bruno, opcional, para ejecutar las colecciones de prueba

## Configuracion

La configuracion activa se encuentra en `src/main/resources/application.properties` y utiliza MySQL:

- Base de datos: `ciberseguridad_db`
- Puerto de la aplicacion: `8080`
- Usuario configurado: `root`

La aplicacion intenta crear la base de datos automaticamente mediante `createDatabaseIfNotExist=true`. Verifica que MySQL este iniciado y que las credenciales configuradas sean validas.

### Alternativa: H2 en memoria

Para ejecutar el proyecto sin MySQL, comenta la configuracion de MySQL y habilita estas propiedades en `application.properties`:

```properties
spring.datasource.url=jdbc:h2:mem:ciberseguridad_db;MODE=MySQL;DB_CLOSE_DELAY=-1;DATABASE_TO_LOWER=TRUE
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.h2.console.enabled=true
```

En un entorno real no se deben guardar contrasenas en el repositorio. Usa variables de entorno o un gestor de secretos y cambia las credenciales expuestas en la configuracion local.

## Ejecucion

Desde la raiz del proyecto:

```bash
mvn clean spring-boot:run
```

Tambien se puede compilar y ejecutar el JAR:

```bash
mvn clean package
java -jar target/incidentes-0.0.1-SNAPSHOT.jar
```

La aplicacion queda disponible en:

- Portal web: http://localhost:8080/
- Proyecto 1: http://localhost:8080/proyecto1.html
- Proyecto 2: http://localhost:8080/proyecto2.html
- Proyecto 3: http://localhost:8080/proyecto3.html

## Endpoints disponibles en el codigo fuente

### Proyecto 1 - OWASP A02

| Metodo | Ruta | Resultado |
|---|---|---|
| `GET` | `/api/vulnerable/users/{id}` | Devuelve la entidad completa y puede exponer campos sensibles. Un ID inexistente puede revelar un stacktrace. |
| `GET` | `/api/mitigated/users/{id}` | Devuelve un `UserDTO` sanitizado. Para un ID inexistente responde `404` con JSON controlado. |

Usuarios de prueba cargados por `data.sql`: IDs `1` y `2`.

### Proyecto 2 - OWASP A03

| Metodo | Ruta | Resultado |
|---|---|---|
| `POST` | `/api/components/vulnerable` | Persiste el componente sin validar dominio ni hash. |
| `POST` | `/api/components/mitigated` | Valida proveedor confiable y coincidencia de SHA-256. Rechaza casos invalidos con `400`. |
| `GET` | `/api/components` | Lista los componentes auditados, ordenados por fecha descendente. |
| `DELETE` | `/api/components` | Elimina todos los registros de componentes. |

El endpoint mitigado acepta dominios incluidos en la lista blanca del servicio, entre ellos `repo.maven.apache.org`, `registry.npmjs.org`, `pypi.org` y `trusted-enterprise.internal`.

### Proyecto 3 - XSS / Stored XSS y keylogging

| Metodo | Ruta | Resultado |
|---|---|---|
| `POST` | `/api/xss/vulnerable/comments` | Persiste el contenido sin escapar para demostrar el riesgo de Stored XSS. |
| `POST` | `/api/xss/mitigated/comments` | Escapa HTML antes de persistir el comentario. |
| `GET` | `/api/xss/comments` | Lista los comentarios del laboratorio. |
| `DELETE` | `/api/xss/comments` | Elimina todos los comentarios. |

La demostracion de keylogging cuenta teclas solo en el navegador y no las envia ni las persiste.

## Colecciones Bruno

Para importar las pruebas en Bruno, abre la carpeta `bruno/` como coleccion. Sus tres carpetas coinciden con las tres carpetas Java implementadas:

- `bruno/proyecto1/`: entidad expuesta, stacktrace expuesto, DTO y error `404` sanitizado.
- `bruno/proyecto2/`: ingesta vulnerable, aprobacion, rechazos por proveedor/hash, listado y limpieza.
- `bruno/proyecto3/`: Stored XSS vulnerable, salida mitigada, listado y limpieza.

Todas las solicitudes apuntan por defecto a `http://localhost:8080`.

## Pruebas automatizadas

Ejecuta las pruebas de Spring Boot con:

```bash
mvn test
```

Las pruebas actuales cubren la carga del contexto, el manejo mitigado de usuarios inexistentes, los casos de aprobación y rechazo de componentes del Proyecto 2 y el escape de contenido del Proyecto 3.

## Estructura principal

```text
src/main/java/com/ciberseguridad/incidentes/
  CiberseguridadApplication.java
  proyecto1/       # usuarios, DTO y manejo de errores
  proyecto2/       # componentes de software y validacion de integridad
  proyecto3/       # comentarios, XSS vulnerable y salida mitigada

src/main/resources/
  application.properties
  data.sql
  static/           # portal web y paginas de cada proyecto

bruno/
  proyecto1/
  proyecto2/
  proyecto3/
```

## Consideraciones de seguridad

La aplicacion reproduce deliberadamente comportamientos inseguros, como la exposicion de entidades, trazas y la ingesta sin validacion. No debe desplegarse en Internet ni utilizar datos reales. Antes de cualquier uso fuera del laboratorio, elimina los endpoints vulnerables, desactiva la inclusion de stacktraces y mueve los secretos de configuracion fuera del codigo fuente.
