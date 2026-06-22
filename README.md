# PetClinic Vets Service

![Tests](https://img.shields.io/badge/tests-passing-brightgreen.svg)
![Java](https://img.shields.io/badge/Java-17+-orange?logo=openjdk)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.x-brightgreen?logo=springboot)
![Maven](https://img.shields.io/badge/Maven-4.x-red?logo=apache-maven)
![Spring Cloud](https://img.shields.io/badge/Spring_Cloud-Eureka-blue?logo=spring)

A Spring Boot microservice that manages veterinarian data and their specialties within the Spring PetClinic application ecosystem.

## Overview

The PetClinic Vets Service is a core microservice in the [Spring PetClinic](https://spring.io/projects/spring-petclinic) distributed architecture. It provides a REST API for querying veterinarian information, including each vet's associated specialties (e.g., dentistry, radiology).

This service is part of a multi-service PetClinic deployment alongside sibling services:
- **petclinic-api-gateway** — API gateway routing requests to downstream services
- **petclinic-customers-service** — Manages pet owner and pet data
- **petclinic-visits-service** — Manages pet visit records

The Vets Service registers itself with Netflix Eureka for service discovery, integrates with Spring Cloud Config for centralized configuration, and supports distributed tracing via Zipkin.

## Features

- **REST API for Veterinarian Data** — Exposes a `GET /vets` endpoint that returns all veterinarians with their specialties (see [API Documentation](#api-documentation) for details).
- **JPA Entity Modeling** — Defines `Vet` and `Specialty` entities with proper relational mappings, including a many-to-many relationship between vets and specialties.
- **Spring Data JPA Repository** — Uses `VetRepository` for database access with zero-boilerplate query methods.
- **Caching with Caffeine** — Supports in-memory caching of vet data (enabled in the `production` profile) to reduce database load (see [Usage](#usage) for configuration details).
- **Service Discovery** — Registers with Netflix Eureka for dynamic service location in a distributed deployment.
- **Centralized Configuration** — Integrates with Spring Cloud Config Server for externalized configuration management.
- **Observability** — Includes Spring Boot Actuator, Micrometer with Prometheus metrics, and Zipkin distributed tracing support.
- **Chaos Engineering** — Includes Chaos Monkey integration for resilience testing.

## Requirements

- **Java 17** or higher
- **Maven 3.9+** (or use the included Maven wrapper if available)
- **Database**: HSQLDB is included as a runtime dependency for development; MySQL connector is also available for production use.
- **Service Discovery**: Netflix Eureka server (required for service registration in distributed mode).

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

The build produces a JAR artifact (`spring-petclinic-vets-service.jar`).

## Quick Start

Once built, you can start the service and verify it is running:

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

## Usage

With the service running, you can interact with it as described below. For a complete reference of all request and response schemas, see the [API Documentation](#api-documentation).

### Retrieving All Veterinarians

The primary endpoint returns all veterinarians with their associated specialties:
```bash
curl -s http://localhost:8080/vets | jq .
```

### Caching Behavior

In the `production` profile, vet list responses are cached using Caffeine. The cache configuration can be customized via `VetsProperties`, which supports cache TTL and heap size settings. Caching is not active in the default profile.

To run with the production profile:
```bash
mvn spring-boot:run -Dspring-boot.run.profiles=production
```

### Integration with the PetClinic Gateway

When deployed as part of the full PetClinic stack, the Vets Service is accessed through the API Gateway. The gateway uses Netflix Eureka to discover the Vets Service instance and routes requests to the `GET /vets` endpoint, providing a unified entry point for clients.

### Actuator Endpoints

Spring Boot Actuator is enabled, providing operational endpoints for health checks, metrics, and application info at `/actuator`:
```bash
curl http://localhost:8080/actuator/health
```

## API Documentation

The following OpenAPI specification describes the REST endpoints exposed by this service.

**OpenAPI Specification (`api_documentation.yaml`)**
```yaml
openapi: 3.0.3
info:
  title: PetClinic Vets Service API
  description: API for retrieving veterinarian information within the PetClinic application.
  version: 1.0.0
paths:
  /vets:
    get:
      summary: Get all veterinarians
      description: Retrieves a list of all veterinarians from the repository, including their specialties. When the `production` profile is active, the response is served from cache.
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
tags:
- name: Vets
```