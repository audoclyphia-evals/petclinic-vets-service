# Vets Service Architecture

A microservice providing veterinarian data through a REST API for the Pet Clinic system.

![Tests](https://img.shields.io/badge/tests-passing-brightgreen.svg)

The Vets Service manages veterinarian information, including their specialties, and exposes it via a RESTful endpoint. It is one of several microservices in the Pet Clinic project — alongside the Customers Service, Visits Service, and API Gateway — and integrates with a service discovery server for dynamic endpoint registration. The service caches its response for performance optimization in production environments.

## Overview

The Vets Service is responsible for serving veterinarian data within the Pet Clinic microservices architecture. It provides a single REST endpoint that returns a list of all veterinarians, each with their associated specialties.

### Main Components

| Component | Package | Purpose |
|---|---|---|
| `VetsServiceApplication` | (root) | Application entry point with discovery client and configuration properties enabled |
| `VetResource` | `web` | REST controller handling the `/vets` endpoint |
| `VetRepository` | `model` | Spring Data JPA repository interface for the `Vet` entity |
| `Vet` | `model` | JPA entity representing a veterinarian, mapped to the `vets` table |
| `Specialty` | `model` | JPA entity representing a vet's specialty, mapped to the `specialties` table |
| `CacheConfig` | `system` | Configuration class enabling caching under the `production` profile |
| `VetsProperties` | `system` | Typesafe configuration properties bound to the `vets` prefix |

## Features

- **REST API** — Exposes a `GET /vets` endpoint that returns all veterinarians with their specialties
- **Response Caching** — Caches the vet list under the `vets` cache key, activated by the `production` profile
- **Service Discovery Integration** — Registers with a discovery server via `@EnableDiscoveryClient` for dynamic service registration
- **Typesafe Configuration** — Manages service-specific settings (cache TTL, heap size) through `VetsProperties`, bound to the `vets` configuration prefix
- **JPA Data Model** — Defines `Vet` and `Specialty` entities with relationships, mapped to relational database tables
- **Sorted Specialties** — Returns each veterinarian's specialties sorted alphabetically by name

## Requirements

- Java 17 or higher
- Apache Maven 3.9+
- A relational database accessible via Spring Data JPA
- A service discovery server (required for production deployment)

## Installation

### Clone the repository

```bash
git clone https://github.com/audoclyphia-evals/petclinic-vets-service.git
cd petclinic-vets-service
```

### Build the project

```bash
mvn clean package
```

### Run the service

```bash
mvn spring-boot:run
```

## Quick Start

1. Build the project:

    ```bash
    mvn clean package
    ```

2. Start the service:

    ```bash
    java -jar target/petclinic-vets-service.jar
    ```

3. Query the vets endpoint:

    ```bash
    curl http://localhost:8080/vets
    ```

    Expected response (JSON array of veterinarian objects):

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
        "specialties": []
      }
    ]
    ```

## Usage

### Retrieve All Veterinarians

Send a GET request to the `/vets` endpoint:

```bash
curl -s http://localhost:8080/vets | jq
```

The response is a JSON array where each element contains:

| Field | Type | Description |
|---|---|---|
| `id` | `Integer` | Unique veterinarian identifier |
| `firstName` | `String` | Veterinarian's first name |
| `lastName` | `String` | Veterinarian's last name |
| `specialties` | `List<Specialty>` | List of specialties, sorted by name |

Each specialty object contains:

| Field | Type | Description |
|---|---|---|
| `id` | `Integer` | Unique specialty identifier |
| `name` | `String` | Specialty name |

### Caching Behavior

The `showResourcesVetList` method is annotated with `@Cacheable("vets")`, meaning the first call to `GET /vets` populates the cache. Subsequent calls return the cached result until the cache expires.

Caching is only active when the `production` Spring profile is enabled, as controlled by `CacheConfig`:

```java
@Configuration
@EnableCaching
@Profile("production")
class CacheConfig {
}
```

### Configuration Properties

Cache settings can be configured via the `vets` prefix in application properties:

| Property | Type | Description |
|---|---|---|
| `vets.cache.ttl` | `int` | Cache time-to-live in seconds |
| `vets.cache.heapSize` | `int` | Maximum number of entries in the cache heap |

These properties are mapped by the `VetsProperties` record:

```java
@ConfigurationProperties(prefix = "vets")
public record VetsProperties(
    Cache cache
) {
    public record Cache(
        int ttl,
        int heapSize
    ) {
    }
}
```