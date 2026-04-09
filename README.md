# 📦 Microservicio Producto

Este proyecto implementa el **Microservicio Producto**, responsable de gestionar productos dentro de una arquitectura de microservicios en evolución.

---

## 🧱 Estado del proyecto

Actualmente incluye:

- API REST funcional para productos
- Persistencia con MySQL
- Configuración por perfiles (`dev`, `prod`)
- Migraciones versionadas con Flyway en `prod`
- Contenerización con Docker
- Documentación OpenAPI/Swagger en `dev`
- Preparado para integración futura con:
  - Config Server
  - Eureka
  - API Gateway

---

## 🏗️ Arquitectura (visión)

```text
Client → Gateway → Microservicios → Eureka → Config Server
```

Este repositorio implementa únicamente el microservicio **Producto**.

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

---

## ⚙️ Stack tecnológico base 2026

- Java 17
- Spring Boot 3.5.12
- Maven 3.9+
- MySQL 8.4
- Docker
- Docker Compose
- Flyway
- SpringDoc OpenAPI

Antes de ejecutar el proyecto, asegúrate de tener instalado:

```bash
java -version
mvn -v
docker -v
docker compose version
```

## Dependencias

- Spring Web
- Spring Data JPA
- Validation
- Lombok
- MySQL Driver
- Flyway
- Spring Boot Actuator
- Spring Boot DevTools
- SpringDoc OpenAPI WebMVC UI

---

## 📌 Dominio gestionado

La entidad principal es `Producto` y actualmente contiene:

- `id`
- `nombre`
- `descripcion`
- `idCategoria`

Tabla actual:

```sql
productos
```

Migración base:

```text
src/main/resources/db/migration/V1__create_productos_table.sql
```

---

## 🔌 Puertos utilizados

| Servicio | Puerto expuesto |
|----------|------------------|
| Aplicación (dev) | 9091 |
| Aplicación (prod) | 9096 |
| MySQL (dev) | 3391 |
| MySQL (prod) | 3396 |

---

## 🔄 Diferencia entre DEV y PROD

| Modo | Ejecución | Base de datos | Puerto app | Swagger | Flyway |
|------|-----------|---------------|------------|---------|--------|
| DEV | Maven | MySQL local o Docker | 9091 | habilitado | deshabilitado |
| PROD | Docker Compose | Docker | 9096 | deshabilitado | habilitado |

---

## 🧪 Endpoints principales

Base path:

```text
/api/v1/productos
```

Operaciones disponibles:

- `POST /api/v1/productos`
- `GET /api/v1/productos`
- `GET /api/v1/productos/{id}`
- `PUT /api/v1/productos/{id}`
- `DELETE /api/v1/productos/{id}`

Ejemplo de payload de creación:

```json
{
  "nombre": "Laptop Lenovo",
  "descripcion": "Equipo para laboratorio",
  "idCategoria": 1
}
```

---

## Base de datos y migraciones

Convención actual:

- Los cambios de esquema deben quedar en SQL versionado.
- Flyway ejecuta automáticamente scripts en `src/main/resources/db/migration` cuando arranca `prod`.
- Ejemplo actual: `V1__create_productos_table.sql`.
- En `prod`, Hibernate no crea tablas; solo valida el esquema existente.
- En `dev`, Hibernate usa `ddl-auto: update` y Flyway está deshabilitado.

Flujo recomendado del equipo:

1. Diseñar o ajustar la tabla en SQL.
2. Probar el cambio en `dev`.
3. Crear una nueva versión SQL si corresponde (`V2`, `V3`, etc.).
4. Aplicar el cambio en `prod`.
5. Arrancar la app en `prod` y validar.

No modificar scripts ya ejecutados; crear siempre una nueva versión.

---

# 🚀 Ejecución en modo desarrollo (dev)

## 🔹 1. Clonar repositorio

Ejemplo:

```bash
git clone https://github.com/261dist/producto.git

cd producto
```

---

## 🔹 2. Levantar base de datos dev

```bash
docker compose -f docker-compose-dev.yml up -d
```

Esto levanta MySQL dev en el puerto `3391` con la base `db_producto`.

Si no usas Docker, también puedes apuntar a un MySQL local siempre que coincida con la configuración de `src/main/resources/application-dev.yml`.

---

## 🔹 3. Ejecutar aplicación

```bash
mvn spring-boot:run
```

Perfil activo por defecto:

```text
dev
```

---

## 🌐 Acceso DEV

API Productos:

```text
http://localhost:9091/api/v1/productos
```

Swagger:

```text
http://localhost:9091/swagger-ui/index.html
```

Health:

```text
http://localhost:9091/actuator/health
```

---

# 🐳 Ejecución en modo producción (prod)

## 🔹 1. Crear archivo `.env`

```env
PRODUCTO_MYSQL_ROOT_PASSWORD=root
PRODUCTO_MYSQL_DATABASE=db_producto

SPRING_PROFILES_ACTIVE=prod

PRODUCTO_DB_HOST=mysql-producto
PRODUCTO_DB_PORT=3306
PRODUCTO_DB_NAME=db_producto
PRODUCTO_DB_USERNAME=root
PRODUCTO_DB_PASSWORD=root
```

---

## 🔹 2. Levantar servicios

```bash
docker compose -f docker-compose.yml up -d
```

Esto levanta:

- MySQL prod en el puerto `3396`
- La aplicación `producto` en el puerto `9096`

---

## 🌐 Acceso PROD

API Productos:

```text
http://localhost:9096/api/v1/productos
```

Health:

```text
http://localhost:9096/actuator/health
```

Swagger:

```text
deshabilitado en prod
```

---

# 📈 Escalado de la aplicación (múltiples instancias)

## 🔹 Opción rápida. No detener el entorno previo

Busca el contenedor de la aplicación con `docker ps -a`:

```bash
docker run --name producto22 --network producto-net --env-file .env -p 9099:9096 producto-prod-producto
```

## 🔹 Verificar

```bash
docker ps
```

---

## 🔹 Probar

- http://localhost:9096/api/v1/productos
- http://localhost:9099/api/v1/productos

---

## 🔹 Finalizar

```bash
docker stop producto22
docker rm producto22
docker rmi producto-prod-producto
```

O limpiar el entorno completo:

```bash
docker rm -f producto1 producto2 producto3
docker compose -f docker-compose.yml down
```

---

## 🔹 Ejecución sin `.env` (opcional)

### PowerShell

```powershell
docker run --name producto33 --network producto-net -p 9098:9096 `
  -e SPRING_PROFILES_ACTIVE=prod `
  -e PRODUCTO_DB_HOST=mysql-producto `
  -e PRODUCTO_DB_PORT=3306 `
  -e PRODUCTO_DB_NAME=db_producto `
  -e PRODUCTO_DB_USERNAME=root `
  -e PRODUCTO_DB_PASSWORD=root `
  producto-service
```

---

# 🔗 Integración futura

## Config Server dev

```properties
SPRING_CONFIG_IMPORT=optional:configserver:http://config-server:7071
```

## Eureka dev

```properties
EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://registry-server:8761/eureka
```

## Gateway

```yaml
uri: lb://producto
```

---

# ⚠️ Alcance actual

Este proyecto no incluye aún:

- API Gateway
- Eureka
- Config Server
- Balanceador

---

# 🧠 Concepto clave

Este proyecto es un microservicio que:

- es independiente
- puede escalar
- expone su propia API REST
- se integrará progresivamente al ecosistema completo

---

# 📌 Nota final

Este repositorio forma parte de una arquitectura de microservicios en evolución.

# 🔧 Anexo PR (flujo de trabajo con Git)

Este flujo permite trabajar con ramas, enviar cambios y versionar el proyecto de forma ordenada.

---

## 🔹 1. Actualizar repositorio

```bash
git branch
git pull origin main
```

---

## 🔹 2. Crear rama de trabajo

```bash
git checkout -b tarea/avance
```

Atención: no trabajes directamente sobre `main`.

---

## 🔹 3. Realizar cambios

```bash
git add .
git commit -m "feat: avance"
git push -u origin tarea/avance
```

---

## 🔹 4. Volver a `main` y limpiar rama

```bash
git checkout main
git pull origin main

git branch -d tarea/avance
git push origin --delete tarea/avance
```

---

## 🔹 5. Crear tag (versión estable)

```bash
git tag -a vs01-producto-base -m "versión base"
git push origin vs01-producto-base
```

---

## 🔹 6. Eliminar tag (si es necesario)

```bash
git tag -d vs01-producto-base
git push origin --delete vs01-producto-base
```

---

## 📚 Documentación adicional

Ver documentación operativa en:

https://upeuoficial.github.io/carrera-sistemas-docs-operativos/