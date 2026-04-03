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
- Spring Validation
- Lombok
- MySQL Driver
- Flyway Core + Flyway MySQL
- Spring Boot Actuator
- Spring Boot DevTools
- SpringDoc OpenAPI WebMVC UI
- H2 Database para pruebas

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
| Aplicación (dev) | 8083 |
| Aplicación (prod) | 8084 |
| MySQL (dev) | 3309 |
| MySQL (prod) | 3310 |

---

## 🔄 Diferencia entre DEV y PROD

| Modo | Ejecución | Base de datos | Puerto app | Swagger | Flyway |
|------|-----------|---------------|------------|---------|--------|
| DEV | Maven | MySQL local o Docker | 8083 | habilitado | deshabilitado |
| PROD | Docker Compose | Docker | 8084 | deshabilitado | habilitado |

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

Esto levanta MySQL dev en el puerto `3309` con la base `db_producto`.

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
http://localhost:8083/api/v1/productos
```

Swagger:

```text
http://localhost:8083/swagger-ui.html
```

Health:

```text
http://localhost:8083/actuator/health
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

- MySQL prod en el puerto `3310`
- La aplicación `producto` en el puerto `8084`

---

## 🌐 Acceso PROD

API Productos:

```text
http://localhost:8084/api/v1/productos
```

Health:

```text
http://localhost:8084/actuator/health
```

Swagger:

```text
deshabilitado en prod
```

---

# 📈 Escalado de la aplicación (múltiples instancias)

## 🔹 Paso 1. Detener entorno previo

Antes de iniciar el escalado, detener los servicios que estuvieran levantados previamente:

```bash
docker compose -f docker-compose.yml down
```

Si existieran contenedores manuales anteriores, también eliminarlos:

```bash
docker rm -f producto1 producto2 producto3
```

---

## 🔹 Paso 2. Levantar solo MySQL

Levantar únicamente la base de datos del entorno de producción:

```bash
docker compose -f docker-compose.yml up -d mysql-producto
```

---

## 🔹 Paso 3. Construir imagen

Generar la imagen Docker de la aplicación:

```bash
docker build -t producto-service .
```

---

## 🔹 Paso 4. Ejecutar instancias

Nota: en este escenario se usa la red `producto-net` definida en Docker Compose.

### Instancia 1

```bash
docker run --name producto1 --network producto-net --env-file .env -p 8084:8084 producto-service
```

### Instancia 2

```bash
docker run --name producto2 --network producto-net --env-file .env -p 8085:8084 producto-service
```

---

## 🔹 Paso 5. Verificar

```bash
docker ps
```

---

## 🔹 Paso 6. Probar

- http://localhost:8084/api/v1/productos
- http://localhost:8085/api/v1/productos

---

## 🔹 Paso 7. Finalizar

```bash
docker stop producto1
docker rm producto1
docker rmi producto-service
```

O limpiar el entorno completo:

```bash
docker rm -f producto1 producto2 producto3
docker compose -f docker-compose.yml down
```

---

## 🔹 Ejecución sin `.env` (opcional)

### Bash

```bash
docker run -d --name producto1 \
  --network producto-net \
  -p 8084:8084 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e PRODUCTO_DB_HOST=mysql-producto \
  -e PRODUCTO_DB_PORT=3306 \
  -e PRODUCTO_DB_NAME=db_producto \
  -e PRODUCTO_DB_USERNAME=root \
  -e PRODUCTO_DB_PASSWORD=root \
  producto-service
```

### PowerShell

```powershell
docker run --name producto3 --network producto-net -p 8086:8084 `
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

## Config Server

```properties
SPRING_CONFIG_IMPORT=optional:configserver:http://config-server:7071
```

## Eureka

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
git tag -a vs01 -m "versión estable"
git push origin vs01
```

---

## 🔹 6. Eliminar tag (si es necesario)

```bash
git tag -d vs01
git push origin --delete vs01
```

---

## 📚 Documentación adicional

Ver documentación operativa en:

https://upeuoficial.github.io/carrera-sistemas-docs-operativos/