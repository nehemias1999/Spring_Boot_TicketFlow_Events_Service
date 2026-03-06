# 🎫 TicketFlow — Event Catalog Service

<div align="center">

![Java](https://img.shields.io/badge/Java-17-orange?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4.3-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![MapStruct](https://img.shields.io/badge/MapStruct-1.6.3-E94E1B?style=for-the-badge)
![Maven](https://img.shields.io/badge/Maven-Wrapper-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)

**Event catalog microservice for a Ticket Reservation System.**

Built with **Hexagonal Architecture**, **Vertical Slicing**, and industry best practices.

</div>

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Entity Model](#-entity-model)
- [API Endpoints](#-api-endpoints)
- [Request & Response Examples](#-request--response-examples)
- [Error Handling](#-error-handling)
- [Validation Rules](#-validation-rules)
- [Getting Started](#-getting-started)
- [Configuration](#-configuration)
- [Design Patterns & Best Practices](#-design-patterns--best-practices)

---

## 🌐 Overview

**TicketFlow Event Catalog Service** is a microservice responsible for managing the event catalog within the TicketFlow Ticket Reservation System. It serves as the **storefront** where all available events (concerts, sports, theater, conferences, etc.) are listed, created, updated, and soft-deleted.

This service exposes a **RESTful API** that other microservices (e.g., Ticket Service) consume to retrieve event information using the unique event ID.

### What does it do?

- 📋 **Create** new event catalog entries with full validation
- 🔍 **Retrieve** a single event by its unique business ID
- 📄 **List** all active events with **pagination** support
- ✏️ **Update** existing event information
- 🗑️ **Soft-delete** events (logical deletion — records are never physically removed)
- 🛡️ **Validate** all incoming data with comprehensive error responses

---

## 🏛️ Architecture

This project follows **Hexagonal Architecture** (Ports & Adapters) combined with **Vertical Slicing**, organizing code by business feature rather than technical layer.

```
┌─────────────────────────────────────────────────────────────────┐
│                     INFRASTRUCTURE LAYER                        │
│                                                                 │
│  ┌──────────────────┐                 ┌──────────────────────┐  │
│  │  CatalogController│                │ CatalogPersistence   │  │
│  │  (Inbound Adapter)│                │ Adapter (Outbound)   │  │
│  └────────┬─────────┘                 └──────────┬───────────┘  │
│           │                                      │              │
│           │  ┌───────────────────────────────┐   │              │
│           │  │      APPLICATION LAYER        │   │              │
│           │  │                               │   │              │
│           ▼  │  ┌────────────────────────┐   │   │              │
│    ICatalogService   │  CatalogService   │   │   │              │
│    (Inbound Port)──▶ │  (Business Logic)  │   │   │              │
│              │  └──────────┬─────────────┘   │   │              │
│              │             │                 │   │              │
│              │             ▼                 │   │              │
│              │  ICatalogPersistencePort ─────┼───▶              │
│              │  (Outbound Port)              │                  │
│              │                               │                  │
│              └───────────────────────────────┘                  │
│                                                                 │
│           ┌──────────────────────────────────────┐              │
│           │           DOMAIN LAYER               │              │
│           │  Catalog (Model) · Exceptions        │              │
│           └──────────────────────────────────────┘              │
└─────────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

| Layer | Responsibility |
|---|---|
| **Domain** | Pure business model (`Catalog`), domain exceptions, port interfaces |
| **Application** | Use case orchestration (`CatalogService`), DTOs, MapStruct mappers |
| **Infrastructure** | REST controller (inbound adapter), JPA persistence (outbound adapter), entity mapping |
| **Shared** | Cross-cutting concerns (global exception handler, error response DTO) |

---

## 🛠️ Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| **Java** | 17 | Programming language |
| **Spring Boot** | 3.4.3 | Application framework |
| **Spring Data JPA** | — | Database persistence & repository abstraction |
| **Spring Validation** | — | Bean validation with Jakarta constraints |
| **MySQL** | 8.0+ | Relational database |
| **MapStruct** | 1.6.3 | Compile-time object mapping (DTO ↔ Domain ↔ Entity) |
| **Lombok** | — | Boilerplate code reduction (getters, setters, builders) |
| **Lombok-MapStruct Binding** | 0.2.0 | Ensures Lombok and MapStruct work together correctly |
| **Maven Wrapper** | — | Build tool (no local Maven installation required) |

---

## 📁 Project Structure

```
src/main/java/com/ticketflow/event_service/
├── CatalogServiceApplication.java              # Spring Boot entry point
│
├── catalog/                                     # 🔷 Catalog vertical slice
│   ├── domain/                                  # Domain layer
│   │   ├── model/
│   │   │   └── Catalog.java                     # Core domain model (POJO)
│   │   ├── port/
│   │   │   ├── in/
│   │   │   │   └── ICatalogService.java         # Inbound port interface
│   │   │   └── out/
│   │   │       └── ICatalogPersistencePort.java # Outbound port interface
│   │   └── exception/
│   │       ├── CatalogNotFoundException.java    # 404 exception
│   │       └── CatalogAlreadyExistsException.java # 409 exception
│   │
│   ├── application/                             # Application layer
│   │   ├── service/
│   │   │   └── CatalogService.java              # Business logic implementation
│   │   ├── mapper/
│   │   │   └── ICatalogApplicationMapper.java   # MapStruct: DTO ↔ Domain
│   │   └── dto/
│   │       ├── request/
│   │       │   ├── CreateCatalogRequest.java    # Create request DTO
│   │       │   └── UpdateCatalogRequest.java    # Update request DTO
│   │       └── response/
│   │           └── CatalogResponse.java         # Response DTO
│   │
│   └── infrastructure/                          # Infrastructure layer
│       └── adapter/
│           ├── in/web/
│           │   └── CatalogController.java       # REST controller (inbound adapter)
│           └── out/persistence/
│               ├── CatalogEntity.java           # JPA entity
│               ├── ICatalogJpaRepository.java   # Spring Data JPA repository
│               ├── CatalogPersistenceAdapter.java # Outbound adapter implementation
│               └── mapper/
│                   └── ICatalogPersistenceMapper.java # MapStruct: Entity ↔ Domain
│
└── shared/                                      # 🔶 Shared cross-cutting concerns
    └── infrastructure/exception/
        ├── GlobalExceptionHandler.java          # @RestControllerAdvice
        └── ApiErrorResponse.java                # Standardized error response
```

---

## 📊 Entity Model

### Catalog

The `Catalog` entity represents an event available in the ticket reservation system.

| Field | Type | Constraints | Description |
|---|---|---|---|
| `id` | `String` | PK, max 20 chars | Unique business identifier (e.g., `"EVT-001"`) |
| `title` | `String` | Not null, 3–150 chars | Event name (e.g., `"Lollapalooza 2026"`) |
| `description` | `String` | Not null, max 500 chars | Short summary of the event |
| `date` | `String` | Not null | Date and time (e.g., `"2026-10-15 20:00"`) |
| `location` | `String` | Not null, max 200 chars | Venue (e.g., `"Estadio River Plate"`) |
| `basePrice` | `BigDecimal` | Not null, ≥ 0, max 10.2 digits | Base reference price |
| `deleted` | `boolean` | Not null, default `false` | Soft-delete flag |
| `createdAt` | `LocalDateTime` | Not null, immutable | Creation timestamp |
| `updatedAt` | `LocalDateTime` | Nullable | Last update timestamp (`null` until first update) |

### Database Table: `catalogs`

```sql
CREATE TABLE catalogs (
    id          VARCHAR(20)     NOT NULL PRIMARY KEY,
    title       VARCHAR(150)    NOT NULL,
    description VARCHAR(500)    NOT NULL,
    date        VARCHAR(255)    NOT NULL,
    location    VARCHAR(200)    NOT NULL,
    base_price  DECIMAL(12,2)   NOT NULL,
    deleted     BOOLEAN         NOT NULL DEFAULT FALSE,
    created_at  DATETIME        NOT NULL,
    updated_at  DATETIME        NULL
);
```

> **Note:** The table is auto-created by Hibernate (`ddl-auto: update`). The database itself is also auto-created via the `createDatabaseIfNotExist=true` JDBC parameter.

---

## 🔌 API Endpoints

Base URL: `http://localhost:8081/api/v1/catalogs`

| Method | Endpoint | Description | Status |
|---|---|---|---|
| `POST` | `/api/v1/catalogs` | Create a new catalog entry | `201 Created` |
| `GET` | `/api/v1/catalogs/{id}` | Get a catalog entry by ID | `200 OK` |
| `GET` | `/api/v1/catalogs?page=0&size=10` | Get all catalogs (paginated) | `200 OK` |
| `PUT` | `/api/v1/catalogs/{id}` | Update an existing catalog entry | `200 OK` |
| `DELETE` | `/api/v1/catalogs/{id}` | Soft-delete a catalog entry | `204 No Content` |

---

## 📝 Request & Response Examples

### Create Catalog — `POST /api/v1/catalogs`

**Request Body:**
```json
{
  "id": "EVT-001",
  "title": "Lollapalooza 2026",
  "description": "The biggest music festival in South America featuring international and local artists",
  "date": "2026-10-15 20:00",
  "location": "Hipódromo de San Isidro, Buenos Aires",
  "basePrice": 45000.00
}
```

**Response (`201 Created`):**
```json
{
  "id": "EVT-001",
  "title": "Lollapalooza 2026",
  "description": "The biggest music festival in South America featuring international and local artists",
  "date": "2026-10-15 20:00",
  "location": "Hipódromo de San Isidro, Buenos Aires",
  "basePrice": 45000.00,
  "createdAt": "2026-03-05T22:30:00",
  "updatedAt": null
}
```

### Get Catalog by ID — `GET /api/v1/catalogs/EVT-001`

**Response (`200 OK`):**
```json
{
  "id": "EVT-001",
  "title": "Lollapalooza 2026",
  "description": "The biggest music festival in South America featuring international and local artists",
  "date": "2026-10-15 20:00",
  "location": "Hipódromo de San Isidro, Buenos Aires",
  "basePrice": 45000.00,
  "createdAt": "2026-03-05T22:30:00",
  "updatedAt": null
}
```

### Get All Catalogs (Paginated) — `GET /api/v1/catalogs?page=0&size=5`

**Response (`200 OK`):**
```json
{
  "content": [
    {
      "id": "EVT-001",
      "title": "Lollapalooza 2026",
      "description": "The biggest music festival in South America...",
      "date": "2026-10-15 20:00",
      "location": "Hipódromo de San Isidro, Buenos Aires",
      "basePrice": 45000.00,
      "createdAt": "2026-03-05T22:30:00",
      "updatedAt": null
    }
  ],
  "pageable": { ... },
  "totalElements": 1,
  "totalPages": 1,
  "size": 5,
  "number": 0,
  "first": true,
  "last": true,
  "empty": false
}
```

### Update Catalog — `PUT /api/v1/catalogs/EVT-001`

**Request Body:**
```json
{
  "title": "Lollapalooza Argentina 2026",
  "description": "Updated: The biggest music festival featuring over 100 artists across 5 stages",
  "date": "2026-10-16 18:00",
  "location": "Hipódromo de San Isidro, Buenos Aires",
  "basePrice": 55000.00
}
```

**Response (`200 OK`):**
```json
{
  "id": "EVT-001",
  "title": "Lollapalooza Argentina 2026",
  "description": "Updated: The biggest music festival featuring over 100 artists across 5 stages",
  "date": "2026-10-16 18:00",
  "location": "Hipódromo de San Isidro, Buenos Aires",
  "basePrice": 55000.00,
  "createdAt": "2026-03-05T22:30:00",
  "updatedAt": "2026-03-05T23:15:00"
}
```

### Delete Catalog — `DELETE /api/v1/catalogs/EVT-001`

**Response:** `204 No Content` (empty body)

---

## ⚠️ Error Handling

All errors return a standardized `ApiErrorResponse`:

```json
{
  "timestamp": "2026-03-05T22:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "Catalog with id 'EVT-999' not found",
  "path": "/api/v1/catalogs/EVT-999"
}
```

| HTTP Status | Exception | When |
|---|---|---|
| `400 Bad Request` | `MethodArgumentNotValidException` | Request body fails validation |
| `404 Not Found` | `CatalogNotFoundException` | Catalog ID does not exist or was soft-deleted |
| `409 Conflict` | `CatalogAlreadyExistsException` | Creating a catalog with a duplicate ID |
| `500 Internal Server Error` | `Exception` | Unexpected server error |

---

## ✅ Validation Rules

### Create Request (`POST`)

| Field | Rules |
|---|---|
| `id` | Required, max 20 characters |
| `title` | Required, 3–150 characters |
| `description` | Required, max 500 characters |
| `date` | Required (non-blank string) |
| `location` | Required, max 200 characters |
| `basePrice` | Required, ≥ 0, max 10 integer + 2 decimal digits |

### Update Request (`PUT`)

| Field | Rules |
|---|---|
| `title` | Required, 3–150 characters |
| `description` | Required, max 500 characters |
| `date` | Required (non-blank string) |
| `location` | Required, max 200 characters |
| `basePrice` | Required, ≥ 0, max 10 integer + 2 decimal digits |

> **Note:** The `id` is not included in the update request — it is taken from the URL path variable and cannot be changed.

---

## 🚀 Getting Started

### Prerequisites

- **Java 17** or higher
- **MySQL 8.0** or higher (running on `localhost:3306`)
- **Git**

> **Maven is NOT required** — the project includes the Maven Wrapper (`mvnw`).

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/Spring_Boot_TicketFlow_Events_Service.git
cd Spring_Boot_TicketFlow_Events_Service/event-service/event-service
```

### 2. Configure the Database

Make sure MySQL is running. The application will **automatically create** the database `ticketflow_events` if it doesn't exist.

Default credentials (configurable in `application.yml`):
```
Host:     localhost:3306
Database: ticketflow_events (auto-created)
Username: root
Password: root
```

### 3. Build the Project

```bash
./mvnw clean compile
```

### 4. Run the Application

```bash
./mvnw spring-boot:run
```

The service will start on **port 8081**: `http://localhost:8081`

### 5. Test the API

```bash
# Create a catalog entry
curl -X POST http://localhost:8081/api/v1/catalogs \
  -H "Content-Type: application/json" \
  -d '{
    "id": "EVT-001",
    "title": "Lollapalooza 2026",
    "description": "The biggest music festival in South America",
    "date": "2026-10-15 20:00",
    "location": "Hipódromo de San Isidro, Buenos Aires",
    "basePrice": 45000.00
  }'

# Get all catalogs
curl http://localhost:8081/api/v1/catalogs

# Get by ID
curl http://localhost:8081/api/v1/catalogs/EVT-001

# Update
curl -X PUT http://localhost:8081/api/v1/catalogs/EVT-001 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Lollapalooza Argentina 2026",
    "description": "Updated festival description",
    "date": "2026-10-16 18:00",
    "location": "Hipódromo de San Isidro, Buenos Aires",
    "basePrice": 55000.00
  }'

# Soft-delete
curl -X DELETE http://localhost:8081/api/v1/catalogs/EVT-001
```

---

## ⚙️ Configuration

All configuration is in `src/main/resources/application.yml`:

| Property | Default | Description |
|---|---|---|
| `server.port` | `8081` | Application port |
| `spring.datasource.url` | `jdbc:mysql://localhost:3306/ticketflow_events?createDatabaseIfNotExist=true&...` | MySQL connection URL (auto-creates DB) |
| `spring.datasource.username` | `root` | Database username |
| `spring.datasource.password` | `root` | Database password |
| `spring.jpa.hibernate.ddl-auto` | `update` | Auto-create/update tables on startup |
| `spring.jpa.show-sql` | `true` | Log SQL queries to console |

---

## 🧩 Design Patterns & Best Practices

| Pattern / Practice | Where Applied |
|---|---|
| **Hexagonal Architecture** | Ports (`ICatalogService`, `ICatalogPersistencePort`) & Adapters (`CatalogController`, `CatalogPersistenceAdapter`) |
| **Vertical Slicing** | All catalog-related code grouped under `catalog/` package |
| **DTO Pattern** | `CreateCatalogRequest`, `UpdateCatalogRequest`, `CatalogResponse` — separating API contracts from domain |
| **Repository Pattern** | `ICatalogJpaRepository` abstracts data access behind Spring Data JPA |
| **Adapter Pattern** | `CatalogPersistenceAdapter` adapts JPA to the domain's outbound port |
| **Builder Pattern** | `Catalog` and `CatalogEntity` use Lombok `@Builder` for clean object construction |
| **Mapper Pattern (MapStruct)** | `ICatalogApplicationMapper`, `ICatalogPersistenceMapper` — compile-time type-safe mapping |
| **Soft Delete** | Records are never physically deleted; a `deleted` flag excludes them from queries |
| **Global Exception Handling** | `@RestControllerAdvice` with standardized `ApiErrorResponse` |
| **Bean Validation** | Jakarta validation annotations on request DTOs |
| **Interface Segregation** | Separate inbound (`ICatalogService`) and outbound (`ICatalogPersistencePort`) ports |
| **Dependency Inversion** | Domain depends on abstractions (ports); infrastructure implements them |
| **Structured Logging** | SLF4J via Lombok `@Slf4j` at all layers with appropriate log levels |
| **Naming Convention** | Interfaces prefixed with `I` (e.g., `ICatalogService`) for clear identification |

---

## 📄 License

This project is for educational and portfolio purposes.

---

<div align="center">

**Built with ❤️ by TicketFlow Team**

</div>
