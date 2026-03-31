# producto

Microservicio Spring Boot para la gestión de productos. Actualmente expone un CRUD REST para la entidad `Producto` y cuenta con documentación OpenAPI/Swagger.

Este repositorio **no es la plantilla base**: es una aplicación concreta (`producto`) creada a partir de la plantilla `catalogo` y adaptada al dominio de productos.

## Stack

- Java 17
- Spring Boot 3.5.12
- Spring Web
- Spring Data JPA
- Spring Validation
- Spring Boot Actuator
- Springdoc OpenAPI
- Flyway (core + mysql)
- MySQL
- Lombok
- Maven Wrapper (opcional)

## Estructura del proyecto

El proyecto sigue una arquitectura en capas simple:

- `controller`: expone los endpoints REST
- `service`: define contratos de negocio
- `service.impl`: implementa la lógica de negocio
- `repository`: acceso a datos con Spring Data JPA
- `entity`: entidades persistentes
- `dto`: objetos request/response
- `mapper`: conversión manual entre entidad y DTO
- `exception`: manejo global de errores
- `config`: configuración técnica, incluyendo OpenAPI


Ubicación recomendada para clases/equipos:

- Cada microservicio (`catalogo`, `producto`, `[otro-ms]`) vive en su propio repositorio Git.
- Clonar cada repositorio de microservicio dentro de la carpeta `services` para trabajo integrado local.
- Mantener la infraestructura en un único repositorio `infra` (Config Server, Registry, Gateway, etc.).
- Estructura sugerida:

```text
ProyectosMS2026/
  infra/
    config-server/
    registry-server/
    gateway/
  services/
    catalogo/
    producto/
    [otro-ms]/
```

## Requisitos

- JDK 17
- Maven no es obligatorio, porque el proyecto incluye `mvnw` y `mvnw.cmd`
- MySQL 8.4 (vía Docker Compose o instalación local)

Ver sección "Guía rápida" → "Paso obligatorio: Levanta la BD primero" para instrucciones de setup.

## Guía rápida para alumnos (autoestudio)

**Importante**: Todos los comandos siguientes se ejecutan desde dentro de la carpeta `producto` (donde está el `pom.xml`).

### Paso obligatorio: Levanta la BD primero

La aplicación necesita MySQL disponible. Antes de hacer cualquier cosa, levanta la base de datos usando una de estas opciones:

**Opción A: Con Docker (recomendado)**

```bash
docker compose -f docker-compose-dev.yml up -d
```

Espera ~3 segundos a que MySQL esté listo (verifica con `docker ps`).

**Opción B: Con Laragon, XAMPP, o MySQL local**

- Asegúrate de que MySQL esté corriendo en tu máquina local.
- Verifica la configuración en `src/main/resources/application-dev.yml` (host, puerto, usuario, contraseña).
- Confirma que la BD `db_producto` existe.

### Sigue este flujo en orden para trabajar sin depender del docente:

1. Ejecutar la aplicación en `dev` (elige una opción):

**Opción 1 (recomendada - VS Code):**
- Abre archivo `src/main/java/com/upeu/producto/ProductoApplication.java`
- Haz clic en el botón `Run` (▶) que aparece sobre la clase
- El servicio inicia con perfil `dev` automáticamente
- No necesitas escribir `mvn` ni `./mvnw` manualmente

**Opción 2 (terminal con Maven Wrapper):**

- Windows (CMD):

```bat
mvnw.cmd spring-boot:run
```

- Windows (PowerShell):

```powershell
.\mvnw.cmd spring-boot:run
```

```powershell
mvn spring-boot:run
```

- Linux / WSL:

```bash
./mvnw spring-boot:run
```

2. Verificar que está corriendo:

- Swagger: `http://localhost:8083/swagger-ui.html`
- Health: `http://localhost:8083/actuator/health`

3. Probar CRUD de `Producto` en Swagger o Postman.

4. Ejecutar pruebas (opcional): ver sección `Pruebas`.

Checklist mínimo antes de entregar tareas:

- BD levantada y corriendo (`docker ps` muestra el contenedor)
- Las pruebas pasan (`BUILD SUCCESS`)
- La app inicia sin errores
- Los endpoints CRUD responden correctamente
- Si hubo cambios de BD, existe script SQL versionado

## Cómo usar este proyecto como referencia

Este repositorio (`producto`) **no es la plantilla base**; es una implementación concreta creada a partir de `catalogo`.

Para crear un nuevo microservicio (por ejemplo, `ventas`) a partir de `catalogo`:

1. Crear un nuevo repositorio para el microservicio y usar `catalogo` como plantilla base.
2. Cambiar `spring.application.name` en `src/main/resources/application.yml`.
3. Ajustar puertos para evitar conflicto con otros microservicios (app y MySQL).
4. Crear nuevas entidades/DTOs/servicios/controladores manteniendo la misma arquitectura por capas.
5. Crear script SQL inicial de la nueva tabla en `src/main/resources/db/migration`.
6. Validar con pruebas y ejecución local.

Recomendación para clases:

- Mantener `catalogo` como “base estable”.
- Mantener este repo (`producto`) como implementación concreta separada.
- Cada alumno o equipo trabaja en una copia (`ventas`, `clientes`, etc.) sin romper la plantilla original.

## Configuración actual

Perfil activo por defecto:

- `dev`
- Se activa automáticamente al ejecutar con `Run` en VS Code o con `mvn spring-boot:run` en local

Puertos por entorno:

- `dev` (Java local): `8083`
- `prod` (Docker): `8084`

Configuración datasource en desarrollo:

- Host: `localhost`
- Puerto: `3309`
- Base de datos: `db_producto`
- Usuario: `root`

Configuración por entorno:

- Base: `src/main/resources/application.yml`
- Dev: `src/main/resources/application-dev.yml`
- Prod: `src/main/resources/application-prod.yml`

Política actual de base de datos:

- `dev`: JPA con `ddl-auto: update` y Flyway deshabilitado
- `prod`: Flyway habilitado y JPA con `ddl-auto: validate`
- El equipo trabaja con enfoque `DB-first`: los cambios de esquema deben quedar versionados en SQL

Para `docker-compose` de prod se usan variables desde `.env` (plantilla en `.env.example`).

Paso obligatorio antes de correr `prod`:

- Crear el archivo `.env` a partir de `.env.example` (el `.env` no se sube a GitHub).
- Windows (PowerShell): `Copy-Item .env.example .env`
- Linux/WSL: `cp .env.example .env`
- Luego ajustar valores de `.env` según tu entorno.
- Validación rápida: antes de `docker compose up -d`, confirma que existe `.env` en la raíz del proyecto.

## Ejecución en paralelo (recomendado)

Este repositorio está preparado para correr `dev` y `prod` al mismo tiempo sin conflicto:

- `dev` app local: `8083`
- `prod` app docker: `8084`
- `dev` mysql docker: `3309`
- `prod` mysql docker: `3310`

Comandos:

```bash
# DB de desarrollo
docker compose -f docker-compose-dev.yml up -d

# Stack productivo local (app + db)
docker compose up -d
```

Para futuros microservicios (`ventas`, `clientes`, etc.), usa el mismo patrón con puertos distintos y nombres de proyecto distintos.

## URLs y Documentación

**Modo `dev` (local)**:
- Swagger: `http://localhost:8083/swagger-ui.html`
- Health: `http://localhost:8083/actuator/health`
- API Productos: `http://localhost:8083/api/v1/productos`

**Modo `prod` (Docker)**:
- API Productos: `http://localhost:8084/api/v1/productos`
- Health: `http://localhost:8084/actuator/health`
- Swagger: deshabilitado

## Pruebas

Ejecutar pruebas:

```bat
mvnw.cmd test
```

En PowerShell (Windows), usar:

```powershell
.\mvnw.cmd test
```

En Linux / WSL:

```bash
./mvnw test
```

Estado validado actual:

- `BUILD SUCCESS`
- `Tests run: 8, Failures: 0, Errors: 0, Skipped: 0`

Nota:

- Actualmente las pruebas se ejecutan de forma manual con Maven (`mvnw.cmd test` o `./mvnw test`).
- En este repositorio no hay pipeline CI/CD activo en GitHub Actions.

## Base de datos y migraciones

Convención actual:

- Los cambios de esquema deben quedar en SQL versionado
- Flyway ejecuta automáticamente scripts en `src/main/resources/db/migration` cuando arranca `prod`
- Ejemplo actual: `V1__create_productos_table.sql`
- En `prod`, Hibernate no crea tablas; solo valida el esquema existente

Flujo recomendado del equipo:

1. Diseñar o ajustar la tabla en SQL
2. Probar el cambio en `dev`
3. Crear una nueva versión SQL si corresponde (`V2`, `V3`, etc.)
4. Aplicar el cambio en `prod`
5. Arrancar la app en `prod` y validar

No modificar scripts ya ejecutados; crear siempre una nueva versión.

