# Sistema de Gestión de Casos de Investigación — Microservicios 🕵️‍♂️📦

Sistema backend basado en microservicios para la gestión de casos de investigación: casos, personas involucradas, evidencia y tareas asociadas. Construido con Spring Boot, Spring Cloud, Kafka y Keycloak, con autenticación y autorización por roles.

## Arquitectura

El sistema está compuesto por los siguientes servicios, todos registrados en un servidor Eureka y expuestos a través de un API Gateway centralizado:

| Servicio | Puerto | Responsabilidad |
|---|---|---|
| **eureka-service** | 8761 | Registro y descubrimiento de servicios |
| **api-gateway** | 8080/8085 | Punto de entrada único, enrutamiento y seguridad (JWT/Keycloak) |
| **case-service** | 8081 | Gestión de casos de investigación (`/cases/**`) |
| **people-service** | 8082 | Gestión de personas involucradas (`/people/**`) |
| **evidence-service** | 8083 | Gestión de evidencia (`/evidences/**`) |
| **task-service** | 8084 | Gestión de tareas asociadas a los casos (`/tasks/**`) |
| **keycloak** | 8090 | Autenticación y autorización (roles: `ADMIN`, `DETECTIVE`, `ANALYST`) |
| **postgres** | 5432 | Base de datos relacional (una BD por servicio) |
| **kafka + zookeeper** | 9092 | Mensajería asíncrona entre servicios (eventos, ej. `CaseCreatedEvent`) |

### Flujo de una petición

```
Cliente → API Gateway (valida JWT contra Keycloak)
              ↓
   ┌──────────┼──────────┬──────────┐
case-service  people-service  evidence-service  task-service
   ↓              ↓              ↓                ↓
 casedb        peopledb      evidencedb        taskdb
```

Cada microservicio tiene su propia base de datos (patrón *database-per-service*) y se comunica de forma asíncrona vía Kafka cuando es necesario (por ejemplo, al crearse un caso).

### Seguridad y roles

El API Gateway valida el JWT emitido por Keycloak y aplica autorización según el rol contenido en el token:

- `/cases/**` → `ADMIN`, `DETECTIVE`, `ANALYST`
- `/people/**` → `ADMIN`, `DETECTIVE`
- `/evidences/**` → `ADMIN`, `DETECTIVE`, `ANALYST`
- `/tasks/**` → `ADMIN`, `DETECTIVE`, `ANALYST`
- `/actuator/**` → público (health checks)

## Stack tecnológico

- **Java 21**
- **Spring Boot** 3.2.5 / 3.3.5
- **Spring Cloud** 2023.0.3 (Gateway, Eureka Client, LoadBalancer)
- **Spring Security** (OAuth2 Resource Server, JWT)
- **PostgreSQL** + JPA/Hibernate
- **Apache Kafka** (mensajería basada en eventos)
- **Keycloak** 23.0.0 (identidad y roles)
- **Docker Compose** para orquestar todo el entorno

## Estructura del proyecto

```
MicroserviciosLogistica/
├── eureka-service/       # Servidor de descubrimiento
├── api-gateway/          # Gateway + seguridad JWT
├── case-service/         # CRUD de casos + eventos Kafka
├── people-service/       # CRUD de personas
├── evidence-service/     # CRUD de evidencia
├── task-service/         # CRUD de tareas
├── init-db.sql           # Creación de las 4 bases de datos
└── docker-compose.yml    # Orquestación de todos los servicios
```

## Cómo levantar el proyecto

### Requisitos
- Docker y Docker Compose
- Java 21 y Maven (si quieres correr algún servicio fuera de Docker)

### Pasos

```bash
git clone https://github.com/samumedigo1411/MicroserviciosLogistica.git
cd MicroserviciosLogistica
docker-compose up --build
```

Servicios disponibles:
- Eureka Dashboard: `http://localhost:8761`
- API Gateway: `http://localhost:8085`
- Keycloak: `http://localhost:8090`

> ⚠️ Antes de correrlo, revisa la sección de **Seguridad** abajo: las credenciales actuales están hardcodeadas y deberías moverlas a variables de entorno, incluso en desarrollo.

## Endpoints principales (vía Gateway)

- `POST/GET /cases/...` — gestión de casos
- `POST/GET /people/...` — gestión de personas
- `POST/GET /evidences/...` — gestión de evidencia
- `POST/GET /tasks/...` — gestión de tareas

Todos requieren un JWT válido emitido por Keycloak, excepto los endpoints de `/actuator/**`.

## ⚠️ Seguridad — pendiente antes de producción

Actualmente las credenciales de Postgres y Keycloak están **hardcodeadas** en `docker-compose.yml` y en cada `application.yml` (`postgres/postgres`, `admin/admin`). Esto es aceptable únicamente para desarrollo local, pero debe corregirse antes de cualquier despliegue real. Ver instrucciones abajo.

## Estado del proyecto

Proyecto académico / parcial de implementación de microservicios 🚧
