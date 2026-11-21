# NextGen CMS

NextGen CMS is a multi-tenant, microservice-based content management platform built with Java 21, Spring Boot 3, PostgreSQL, Redis, OpenSearch, and Next.js.

The platform separates content types and fields from Java domain classes so that schemas, forms, workflows, media, notifications, search, and audit behavior can be managed independently.

> This repository is under active development. Some admin UI actions and production integrations are not yet complete.

## Features

- Multi-tenant identity and access management
- JWT authentication and refresh-token support
- Configurable content types and field definitions
- Versioned content items
- Workflow definitions and state transitions
- Dynamic form definitions and submissions
- Media asset upload and management
- OpenSearch indexing and search
- Notification templates and delivery logs
- Audit event recording
- Next.js administration dashboard
- Spring Cloud API Gateway
- PostgreSQL database-per-service design
- Docker Compose local infrastructure

## Architecture

```text
Next.js Admin UI
       |
       v
Spring Cloud API Gateway :8080
       |
       +-- IAM Service          :8081
       +-- Schema Service       :8082
       +-- Content Service      :8083
       +-- Workflow Service     :8084
       +-- Media Service        :8085
       +-- Notification Service :8086
       +-- Audit Service        :8087
       +-- Search Service       :8088
       +-- Form Service         :8089

PostgreSQL | Redis | OpenSearch
```

Each Spring service owns its own PostgreSQL database and exposes REST APIs through the gateway. The IAM service validates and issues JWTs, and tenant context is carried with the `X-Tenant-Id` header.

## Technology Stack

| Area | Technology |
|---|---|
| Backend | Java 21, Spring Boot 3.3, Spring Cloud Gateway |
| Security | Spring Security, OAuth2 Resource Server, JWT |
| Persistence | PostgreSQL 16, Spring Data JPA, Flyway |
| Caching | Redis 7 |
| Search | OpenSearch 2.11 |
| Frontend | Next.js 16, React 19, TypeScript, Tailwind CSS |
| Build tools | Maven, npm |
| Local infrastructure | Docker Compose |

## Repository Structure

```text
cms-admin-ui/                Next.js administration dashboard
cms-common/                  Shared DTOs, exceptions, security, and utilities
cms-gateway/                 API gateway
cms-iam-service/             Authentication, users, roles, and tenants
cms-schema-service/          Content type and field definitions
cms-content-service/         Content items and versions
cms-workflow-service/        Workflow definitions and transitions
cms-media-service/           Media asset storage
cms-form-service/            Dynamic forms and submissions
cms-notification-service/    Templates and notification logs
cms-audit-service/           Audit event storage
cms-search-service/          OpenSearch integration
docs/                        Additional project documentation
docker-compose.yml           Local databases and middleware
start-services.sh            Build and local startup script
stop-services.sh             Local shutdown script
```

## Prerequisites

Install the following:

- Java 21
- Maven 3.9 or newer
- Node.js 20 or newer
- npm
- Docker with Docker Compose
- Bash

## Quick Start

### 1. Start the infrastructure

```bash
docker compose up -d
```

This starts the PostgreSQL databases, Redis, and OpenSearch. Use Docker Compose v2; if `docker compose` is unavailable, `docker-compose up -d` may be used.

### 2. Install the shared library

```bash
cd cms-common
mvn install -DskipTests
cd ..
```

### 3. Start all services

On Linux or macOS:

```bash
chmod +x start-services.sh stop-services.sh
./start-services.sh
```

The script builds `cms-common`, starts Docker infrastructure, launches every Spring service, and starts the Next.js admin UI. Logs are written to `logs/<module>.log`.

Stop the environment with:

```bash
./stop-services.sh
```

### 4. Open the admin UI

```text
http://localhost:3000
```

The API gateway is available at:

```text
http://localhost:8080
```

## Run Services Individually

Install `cms-common` first, then run a service from its module directory:

```bash
cd cms-iam-service
mvn spring-boot:run
```

Run the frontend separately:

```bash
cd cms-admin-ui
npm install
npm run dev
```

For frontend API configuration, set `NEXT_PUBLIC_API_URL` to the gateway URL:

```bash
NEXT_PUBLIC_API_URL=http://localhost:8080 npm run dev
```

## Service Ports and Databases

| Service | Port | Database |
|---|---:|---|
| API Gateway | 8080 | None |
| IAM | 8081 | `iamdb` on port 5434 |
| Schema | 8082 | `schemadb` on port 5435 |
| Content | 8083 | `contentdb` on port 5437 |
| Workflow | 8084 | `workflowdb` on port 5436 |
| Media | 8085 | `mediadb` on port 5438 |
| Notification | 8086 | `notificationdb` on port 5439 |
| Audit | 8087 | `auditdb` on port 5440 |
| Search | 8088 | OpenSearch on port 9200 |
| Form | 8089 | `formdb` on port 5441 |

Redis is available on port `6379`.

## Gateway API Routes

| Route prefix | Destination |
|---|---|
| `/api/v1/auth/**` | IAM authentication |
| `/api/v1/iam/**` | IAM user management |
| `/api/v1/schema/**` | Schema service |
| `/api/v1/content/**` | Content service |
| `/api/v1/workflow/**` | Workflow service |
| `/api/v1/media/**` | Media service |
| `/api/v1/forms/**` | Form service |

Protected API requests should include:

```http
Authorization: Bearer <access-token>
X-Tenant-Id: <tenant-id>
```

## Environment Configuration

Local database credentials are defined in `docker-compose.yml`. The Spring services use local defaults for development, while several values can be overridden with environment variables.

Common overrides include:

| Variable | Used for |
|---|---|
| `DB_URL` | IAM database URL |
| `DB_USERNAME` | IAM database username |
| `DB_PASSWORD` | IAM database password |
| `REDIS_HOST` | Redis host |
| `REDIS_PORT` | Redis port |
| `JWT_PRIVATE_KEY_PATH` | IAM JWT signing key |
| `JWT_PUBLIC_KEY_PATH` | IAM JWT verification key |
| `SCHEMA_SERVICE_URL` | Content-to-schema service URL |
| `WORKFLOW_SERVICE_URL` | Content-to-workflow service URL |
| `IAM_SERVICE_URL` | Gateway IAM route |
| `NEXT_PUBLIC_API_URL` | Frontend API gateway URL |

Do not use the local passwords or development JWT configuration in a production deployment.

## Development Status

Implemented areas include service boundaries, database migrations, JWT authentication, REST controllers, dynamic schema models, content and form workflows, media storage, and the initial admin dashboard.

Known development limitations:

- Some dashboard metrics are placeholder values.
- Several admin table actions, edit flows, and submission viewers are not yet connected.
- Public delivery and production media/CDN integrations may require additional work.
- Kafka and Kubernetes resources described in architectural documents are not all implemented in the current Compose setup.
- Automated test coverage is still limited.
- Local development scripts use broad process termination; prefer service-specific shutdown commands for shared environments.

## Documentation

- [Architecture overview](cms-architecture.md)
- [Database structure](db-structure.md)
- [Database technology selection](db-technology-selection.md)
- [Authentication flow](auth-flow.md)
- [CMS diagrams](cms-diagrams.md)
- [Implementation workflow](docs/workflow.md)
- [Open questions](open-questions.md)
- [Phase 1 plan](phase1-plan.md)
- [Phase 2 plan](phase2-plan.md)
- [Phase 3 plan](phase3-plan.md)
- [AI SDLC notes](ai_sdlc.md)
