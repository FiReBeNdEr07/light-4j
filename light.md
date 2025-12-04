# Light-4j Petstore API - Codebase Summary

## Overview

This is a **Petstore REST API** microservice built using the **Light-4j framework** (version 2.0.29), following the OpenAPI 3.0 specification. The project demonstrates a production-ready microservice architecture with comprehensive middleware chain, security, and containerization support.

**Key Technologies:**
- **Framework:** Light-4j 2.0.29
- **Java Version:** 11
- **Server:** Undertow 2.2.4.Final
- **Build Tool:** Maven
- **API Specification:** OpenAPI 3.0
- **Security:** OAuth2 with JWT

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Request                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Undertow HTTP Server                          │
│                   (HTTP: 8080 / HTTPS: 8443)                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Middleware Chain                              │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────────┐ │
│  │ Exception  │→│  Metrics   │→│Traceability│→│ Correlation  │ │
│  │  Handler   │ │  Handler   │ │  Handler   │ │   Handler    │ │
│  └────────────┘ └────────────┘ └────────────┘ └──────────────┘ │
│                                                                  │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────────┐ │
│  │  OpenAPI   │→│    JWT     │→│    Body    │→│  Sanitizer   │ │
│  │  Handler   │ │   Verify   │ │  Handler   │ │   Handler    │ │
│  └────────────┘ └────────────┘ └────────────┘ └──────────────┘ │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Routing Layer                                 │
│                 (Path & Method Matching)                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Business Handlers                             │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │ PetsGetHandler   │  │ PetsPostHandler  │                     │
│  └────────┬─────────┘  └────────┬─────────┘                     │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │PetsPetIdGet      │  │PetsPetIdDelete   │                     │
│  │Handler           │  │Handler           │                     │
│  └────────┬─────────┘  └────────┬─────────┘                     │
└───────────┼──────────────────────┼───────────────────────────────┘
            │                      │
            ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Service Layer                                 │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │ PetsGetService   │  │ PetsPostService  │                     │
│  └────────┬─────────┘  └────────┬─────────┘                     │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │PetsPetIdGet      │  │PetsPetIdDelete   │                     │
│  │Service           │  │Service           │                     │
│  └────────┬─────────┘  └────────┬─────────┘                     │
└───────────┼──────────────────────┼───────────────────────────────┘
            │                      │
            ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Data Models                                   │
│              ┌──────────┐  ┌──────────┐                          │
│              │   Pet    │  │  Error   │                          │
│              └──────────┘  └──────────┘                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
light-4j/
├── src/
│   ├── main/
│   │   ├── java/com/networknt/petstore/
│   │   │   ├── handler/           # HTTP Request Handlers
│   │   │   │   ├── PetsGetHandler.java
│   │   │   │   ├── PetsPostHandler.java
│   │   │   │   ├── PetsPetIdGetHandler.java
│   │   │   │   └── PetsPetIdDeleteHandler.java
│   │   │   ├── service/           # Business Logic Services
│   │   │   │   ├── PetsGetService.java
│   │   │   │   ├── PetsPostService.java
│   │   │   │   ├── PetsPetIdGetService.java
│   │   │   │   └── PetsPetIdDeleteService.java
│   │   │   └── model/             # Data Transfer Objects
│   │   │       ├── Pet.java
│   │   │       └── Error.java
│   │   └── resources/
│   │       ├── config/            # Configuration Files
│   │       │   ├── openapi.yaml   # API Specification
│   │       │   ├── handler.yml    # Middleware Chain Config
│   │       │   ├── server.yml     # Server Configuration
│   │       │   ├── security.yml   # OAuth2/JWT Settings
│   │       │   └── ...
│   │       └── logback.xml        # Logging Configuration
│   └── test/
│       └── java/com/networknt/petstore/
│           └── handler/           # Unit Tests
├── docker/
│   ├── Dockerfile                 # Standard Docker Build
│   └── Dockerfile-Slim            # Optimized Docker Build
├── kubernetes.yml                 # K8s Deployment Config
├── pom.xml                        # Maven Build Configuration
└── build.sh                       # Build Script
```

---

## API Endpoints

### 1. List All Pets
```
GET /v1/pets
Query Parameters: limit (optional)
Security: OAuth2 - read:pets
Response: 200 - Array of Pet objects
```

### 2. Create a Pet
```
POST /v1/pets
Request Body: Pet object (JSON)
Security: OAuth2 - read:pets, write:pets
Response: 201 - Created
```

### 3. Get Pet by ID
```
GET /v1/pets/{petId}
Path Parameters: petId
Security: OAuth2 - read:pets
Response: 200 - Pet object
```

### 4. Delete Pet by ID
```
DELETE /v1/pets/{petId}
Path Parameters: petId
Header: key (required)
Security: OAuth2 - write:pets
Response: 200 - Pet object (deleted)
```

---

## Request Flow Diagram

```
┌──────────┐
│  Client  │
└────┬─────┘
     │ 1. HTTPS Request
     ▼
┌─────────────────┐
│ ExceptionHandler│ ← Catches all exceptions
└────┬────────────┘
     │ 2. Proceed
     ▼
┌─────────────────┐
│ MetricsHandler  │ ← Collects metrics
└────┬────────────┘
     │ 3. Proceed
     ▼
┌─────────────────┐
│TraceabilityHandler│ ← Adds trace IDs
└────┬────────────┘
     │ 4. Proceed
     ▼
┌─────────────────┐
│CorrelationHandler│ ← Adds correlation IDs
└────┬────────────┘
     │ 5. Proceed
     ▼
┌─────────────────┐
│  OpenApiHandler │ ← Validates against spec
└────┬────────────┘
     │ 6. Valid Request
     ▼
┌─────────────────┐
│ JwtVerifyHandler│ ← Validates OAuth2 JWT
└────┬────────────┘
     │ 7. Authenticated
     ▼
┌─────────────────┐
│   BodyHandler   │ ← Parses request body
└────┬────────────┘
     │ 8. Body Parsed
     ▼
┌─────────────────┐
│SanitizerHandler │ ← Sanitizes input
└────┬────────────┘
     │ 9. Clean Input
     ▼
┌─────────────────┐
│  Router/Handler │ ← Business logic
└────┬────────────┘
     │ 10. Calls Service
     ▼
┌─────────────────┐
│  Service Layer  │ ← Processes request
└────┬────────────┘
     │ 11. Returns Response
     ▼
┌──────────┐
│  Client  │ ← JSON Response
└──────────┘
```

---

## Data Models

### Pet Model
```java
{
  "id": Long (required),
  "name": String (required),
  "tag": String (optional)
}
```

### Error Model
```java
{
  "code": Integer (required),
  "message": String (required)
}
```

---

## Component Details

### Handlers
- **Purpose:** Process HTTP requests and delegate to services
- **Pattern:** Implements `LightHttpHandler` interface
- **Responsibilities:**
  - Extract request parameters (query, path, headers)
  - Create `RequestEntity` object
  - Call corresponding service
  - Build HTTP response from `ResponseEntity`

### Services
- **Purpose:** Contain business logic
- **Pattern:** Implements `HttpService<Input, Output>` interface
- **Responsibilities:**
  - Process business logic
  - Return `ResponseEntity` with status, headers, and body
  - Currently returns mock data (hardcoded JSON)

### Models
- **Purpose:** Data Transfer Objects (DTOs)
- **Pattern:** POJOs with Jackson annotations
- **Features:**
  - JSON serialization/deserialization
  - Equals/hashCode implementation
  - toString with indentation

---

## Middleware Chain

The middleware chain is configured in `handler.yml`:

| Order | Middleware | Purpose |
|-------|------------|---------|
| 1 | ExceptionHandler | Global exception handling |
| 2 | MetricsHandler | Performance metrics collection |
| 3 | TraceabilityHandler | Distributed tracing support |
| 4 | CorrelationHandler | Request correlation tracking |
| 5 | OpenApiHandler | API specification validation |
| 6 | JwtVerifyHandler | OAuth2 JWT token validation |
| 7 | BodyHandler | Request body parsing |
| 8 | SanitizerHandler | Input sanitization |
| 9 | ValidatorHandler | Request/response validation |
| 10 | AuditHandler | Audit logging |

---

## Configuration Files

### Server Configuration (`server.yml`)
- **HTTP Port:** 8080 (enabled by default for testing)
- **HTTPS Port:** 8443 (disabled by default)
- **HTTP/2:** Enabled
- **TLS:** Configurable with keystore/truststore

### Security Configuration
- **OAuth2 Provider:** Token URL at `https://localhost:6882/token`
- **Grant Type:** Client Credentials
- **Scopes:**
  - `read:pets` - Read pet information
  - `write:pets` - Modify pet information

### OpenAPI Specification
- **Version:** 3.0.0
- **Title:** Swagger Petstore
- **License:** MIT
- **Base URL:** http://petstore.swagger.io/v1

---

## Build & Deployment

### Build Commands
```bash
# Full build with fat jar
./mvnw clean install -Prelease

# Quick build without fat jar
./mvnw clean install exec:exec

# Run the server
java -jar target/server.jar
```

### Docker Build
```bash
# Standard build
docker build -f docker/Dockerfile -t petstore:latest .

# Slim build (optimized)
docker build -f docker/Dockerfile-Slim -t petstore:slim .

# Using build script
./build.sh 0.0.1
```

### Kubernetes Deployment
- Deployment configuration available in `kubernetes.yml`
- Supports container orchestration
- Health checks and readiness probes configured

---

## Dependencies

### Core Light-4j Modules
- `config` - Configuration management
- `utility` - Common utilities
- `security` - Security features
- `client` - HTTP client
- `audit` - Audit logging
- `info` - Server info endpoint
- `health` - Health check endpoint
- `status` - Status management
- `exception` - Exception handling
- `body` - Request body handling
- `mask` - Data masking
- `dump` - Request/response dumping

### External Dependencies
- Jackson 2.12.1 - JSON processing
- SLF4J 1.7.25 - Logging facade
- Logback 1.2.3 - Logging implementation
- Jose4j 0.6.3 - JWT/JWE/JWS support
- Undertow 2.2.4.Final - Web server
- JSON Schema Validator 1.0.49 - Schema validation

---

## Testing

### Test Structure
- **Location:** `src/test/java/com/networknt/petstore/handler/`
- **Test Server:** `TestServer.java` for integration testing
- **Test Classes:**
  - `PetsGetHandlerTest.java`
  - `PetsPostHandlerTest.java`
  - `PetsPetIdGetHandlerTest.java`
  - `PetsPetIdDeleteHandlerTest.java`

### Testing Approach
```bash
# Test endpoint with curl (OAuth2 disabled for testing)
curl -k https://localhost:8443/v1/pets
```

---

## Key Features

### 1. **Production Ready**
- Comprehensive middleware chain
- Exception handling
- Metrics and monitoring
- Distributed tracing
- Audit logging

### 2. **Security First**
- OAuth2 with JWT
- Request/response validation
- Input sanitization
- TLS/SSL support

### 3. **Performance Optimized**
- Undertow non-blocking server
- HTTP/2 support
- Efficient routing
- Minimal overhead

### 4. **Cloud Native**
- Docker containerization
- Kubernetes support
- 12-factor app principles
- Environment-based configuration

### 5. **Developer Friendly**
- OpenAPI specification
- Comprehensive configuration
- Clear separation of concerns
- Extensive documentation

---

## Design Patterns Used

1. **Handler Pattern:** Separate handlers for each endpoint
2. **Service Layer Pattern:** Business logic isolation
3. **Chain of Responsibility:** Middleware chain processing
4. **DTO Pattern:** Model objects for data transfer
5. **Dependency Injection:** Service injection in handlers
6. **Template Method:** Base handler implementation

---

## Current Implementation Status

### ✅ Implemented
- OpenAPI 3.0 specification
- Complete handler structure
- Service layer with mock data
- Data models (Pet, Error)
- Middleware chain configuration
- Build and deployment scripts
- Docker support
- Kubernetes configuration

### 🔄 Mock/Placeholder
- Services return hardcoded JSON (no database)
- No actual OAuth2 integration (disabled for testing)
- No persistent storage

### 📝 Next Steps (Potential)
- Database integration (e.g., PostgreSQL, MongoDB)
- Real OAuth2 provider integration
- Complete CRUD operations with persistence
- Additional business logic
- Enhanced test coverage
- Production monitoring setup

---

## Quick Start Guide

1. **Build the project:**
   ```bash
   ./mvnw clean install
   ```

2. **Run the server:**
   ```bash
   java -jar target/server.jar
   ```

3. **Test an endpoint:**
   ```bash
   curl -k https://localhost:8443/v1/pets
   ```

4. **Expected Response:**
   ```json
   [
     {"id":1,"name":"catten","tag":"cat"},
     {"id":2,"name":"doggy","tag":"dog"}
   ]
   ```

---

## Documentation & Resources

- **Light-4j Framework:** https://doc.networknt.com/
- **Tutorial:** https://doc.networknt.com/tutorial/rest/openapi/petstore/
- **Business Handler Guide:** https://doc.networknt.com/development/business-handler/rest/

---

## Summary

This Light-4j Petstore API demonstrates a **modern, production-ready microservice architecture** with:
- Clean separation between handlers, services, and models
- Comprehensive middleware for security, monitoring, and validation
- OpenAPI-first design approach
- Cloud-native deployment capabilities
- Excellent foundation for building scalable REST APIs

The codebase follows best practices for microservice development and provides a solid template for building enterprise-grade APIs with the Light-4j framework.
