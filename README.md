# PetClinic Vets Service

![Tests](https://img.shields.io/badge/tests-passing-brightgreen.svg)
![Java](https://img.shields.io/badge/Java-17+-orange?logo=openjdk)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.x-brightgreen?logo=springboot)
![Maven](https://img.shields.io/badge/Maven-4.x-red?logo=apache-maven)
![Spring Cloud](https://img.shields.io/badge/Spring_Cloud-Eureka-blue?logo=spring)

A Spring Boot microservice that manages veterinarian data and their specialties within the Spring PetClinic application ecosystem.

## Project Overview

The PetClinic Vets Service is a core microservice in the [Spring PetClinic](https://spring.io/projects/spring-petclinic) distributed architecture. It provides a REST API for querying veterinarian information, including each vet's associated specialties (e.g., dentistry, radiology).

This service is part of a multi-service deployment alongside sibling services:
- **petclinic-api-gateway** — API gateway routing requests to downstream services
- **petclinic-customers-service** — Manages pet owner and pet data
- **petclinic-visits-service** — Manages pet visit records

The service registers itself with Netflix Eureka for service discovery, integrates with Spring Cloud Config for centralized configuration, and supports distributed tracing via Zipkin.

## Features

- **REST API for Veterinarian Data** — Exposes `GET /vets` (all veterinarians) and `GET /vets/{vetId}` (single vet by ID) endpoints.
- **JPA Entity Modeling** — Defines `Vet` and `Specialty` entities with a many-to-many relationship.
- **Spring Data JPA Repository** — Uses `VetRepository` for database access.
- **Caching with Caffeine** — Supports in-memory caching in the `production` profile (see [Caching Behavior](#caching-behavior)).
- **Service Discovery** — Registers with Netflix Eureka (see [Integration with the PetClinic Gateway](#integration-with-the-petclinic-gateway)).
- **Centralized Configuration** — Integrates with Spring Cloud Config Server.
- **Observability** — Includes Spring Boot Actuator, Micrometer metrics, and Zipkin tracing (see [Actuator Endpoints](#actuator-endpoints)).
- **Chaos Engineering** — Includes Chaos Monkey integration for resilience testing.

## Requirements

- **Java 17** or higher
- **Maven 3.9+** (or use the included Maven wrapper if available)
- **Database**: HSQLDB is included as a runtime dependency for development; MySQL connector is also available for production use.
- **Service Discovery**: Netflix Eureka server (required for distributed mode).

With these prerequisites satisfied, you can proceed with installation.

## Installation

```bash
# Clone the repository
git clone https://github.com/audoclyphia-evals/petclinic-vets-service.git
cd petclinic-vets-service

# Build the project and run tests
mvn clean install

# Skip tests if needed
mvn clean install -DskipTests
```

The build produces a JAR artifact (`spring-petclinic-vets-service.jar`). You can now start the service.

## Quick Start

Once built, start the service and verify it is running:

1. Run the application:
```bash
mvn spring-boot:run
```

2. Verify the service by calling the vet list endpoint:
```bash
curl http://localhost:8080/vets
```

Expected response (JSON array of vet objects):
```json
[
  {
    "id": 1,
    "firstName": "James",
    "lastName": "Carter",
    "specialties": []
  },
  {
    "id": 2,
    "firstName": "Helen",
    "lastName": "Leary",
    "specialties": [
      { "id": 1, "name": "radiology" }
    ]
  }
]
```

For more advanced usage, including caching and gateway integration, see the [Usage](#usage) section.

## Usage

### Retrieving All Veterinarians

The primary endpoint returns all veterinarians with their associated specialties:
```bash
curl -s http://localhost:8080/vets | jq .
```

### Retrieving a Single Veterinarian

Fetch a specific veterinarian by their ID. Returns the vet object with a `200 OK` status, or `404 Not Found` if no vet exists:
```bash
curl -s http://localhost:8080/vets/1 | jq .
```

If the vet ID does not exist, the endpoint returns an empty response with HTTP status `404`.

### Caching Behavior

When run with the `production` profile, vet list responses are cached using Caffeine. Caching is not active in the default profile. The cache configuration can be customized via `VetsProperties`, which supports cache TTL and heap size settings.

To run with the production profile:
```bash
mvn spring-boot:run -Dspring-boot.run.profiles=production
```

### Integration with the PetClinic Gateway

When deployed as part of the full PetClinic stack, the Vets Service is accessed through the PetClinic API Gateway. The gateway uses Netflix Eureka to discover the Vets Service instance and routes requests to its endpoints, providing a unified entry point. For a broader understanding of the system design, refer to the [System Architecture Documentation](ARCHITECTURE.md).

### Actuator Endpoints

Spring Boot Actuator provides operational endpoints for health checks, metrics, and application info at `/actuator`:
```bash
curl http://localhost:8080/actuator/health
```

## API Reference

The service's REST API is summarized below. For complete details on request parameters, response schemas, and error codes, refer to the [API Documentation](api_documentation.yaml).

### Endpoints

| Method | Path          | Description                                      |
|--------|---------------|--------------------------------------------------|
| GET    | `/vets`       | Retrieve a list of all veterinarians with caching. |
| GET    | `/vets/{vetId}`| Retrieve a single veterinarian by ID.            |

### OpenAPI Specification

```yaml
openapi: 3.0.3
info:
  title: PetClinic Vets Service API
  description: Auto-generated API documentation
  version: 1.0.0
paths:
  /vets:
    get:
      summary: Retrieve all vets
      description: Retrieves a list of all veterinarians with caching enabled.
      operationId: showResourcesVetList
      tags:
      - Vets
      responses:
        '200':
          description: Successful retrieval of vet list
        '400':
          description: Bad request
        '401':
          description: Unauthorized access
        '500':
          description: Internal server error
  /vets/{vetId}:
    get:
      summary: Retrieve a vet by ID
      description: Fetches a vet by ID, returns the vet or 404 if not found.
      operationId: getVetById
      tags:
      - Vets
      parameters:
      - name: vetId
        in: path
        required: true
        schema:
          type: integer
        description: Unique identifier of the vet
      responses:
        '200':
          description: Vet found and returned successfully
        '404':
          description: Vet not found
tags:
- name: Vets
```

## Additional Documentation

- [System Architecture Documentation](ARCHITECTURE.md) - Overview of the Spring PetClinic application architecture and component interactions.
- [API Documentation](api_documentation.yaml) - Detailed OpenAPI specification file.