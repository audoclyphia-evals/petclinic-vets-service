# petclinic-vets-service

![Tests](https://img.shields.io/badge/tests-passing-brightgreen.svg)

Microservice providing veterinarian data for the Pet Clinic application, exposing a REST API to retrieve vet records with their associated specialties.

## Project Overview

The `petclinic-vets-service` is a standalone Spring Boot microservice within the Pet Clinic microservices architecture. It manages veterinarian data and exposes a REST API for consumers to query the list of available vets and their specialties. The service registers with a discovery client, enabling dynamic service discovery within the broader ecosystem.

This service is one of several backend services in the Pet Clinic project, alongside services for managing customers, visits, and an API gateway for routing requests.

## Features

The service provides the following key features:

- **REST API** — Exposes a `GET /vets` endpoint returning a list of all veterinarians with their specialties.
- **Data Caching** — Supports caching of vet list responses under the `vets` cache key when the `production` profile is active. Cache behavior is configurable via properties.
- **Typesafe Configuration** — Provides a `VetsProperties` configuration record with tunable cache settings (TTL and heap size) bound under the `vets` prefix.
- **Service Discovery** — Integrates with a service discovery mechanism via `@EnableDiscoveryClient`.
- **JPA Data Access** — Uses Spring Data JPA with a `VetRepository` interface extending `JpaRepository` for database access.
- **Domain Model** — Defines `Vet` and `Specialty` JPA entities representing veterinarians and their areas of specialization.

## Requirements

To build and run the service, you need:

- Java 17 or higher
- Maven 3.6+
- Access to a relational database (compatible with JPA/Hibernate)
- A service discovery server (for production deployment)

## Installation

Clone the repository and build the project with Maven.

```bash
# Clone the repository
git clone https://github.com/audoclyphia-evals/petclinic-vets-service.git
cd petclinic-vets-service

# Build the project
mvn clean package

# Run the service
mvn spring-boot:run
```

## Quickstart

1. Build and start the service as described in the [Installation](#installation) section.

2. Verify the service is running by querying the vets endpoint:

```bash
curl http://localhost:8080/vets
```

3. The service should return a JSON array of veterinarian objects. The exact response depends on database contents, but the structure reflects the `Vet` entity with nested `Specialty` objects, as shown in the example below:

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

With the service running, you can interact with its API. The primary operation is retrieving the list of all veterinarians.

### Retrieving the Vet List

The `GET /vets` endpoint returns all veterinarians from the database. The core implementation in the `VetResource` controller is:

```java
@RestController
@RequestMapping("/vets")
class VetResource {

    private final VetRepository vetRepository;

    VetResource(VetRepository vetRepository) {
        this.vetRepository = vetRepository;
    }

    @GetMapping
    @Cacheable("vets")
    public List<Vet> showResourcesVetList() {
        return vetRepository.findAll();
    }
}
```

As noted in the [Features](#features) section, when the `production` profile is active, responses are cached. Subsequent requests within the cache TTL window return the cached result without a database query.

### Domain Model

The service uses a JPA domain model. The `Vet` entity maps to the `vets` database table:

| Property | Type | Description |
|---|---|---|
| `id` | `Integer` | Unique identifier |
| `firstName` | `String` | Veterinarian's first name |
| `lastName` | `String` | Veterinarian's last name |
| `specialties` | `List<Specialty>` | Sorted list of associated specialties |

The `Specialty` entity maps to the `specialties` table:

| Property | Type | Description |
|---|---|---|
| `id` | `Integer` | Unique identifier |
| `name` | `String` | Specialty name |

### Configuration Properties

Cache behavior is configurable through the `vets` configuration prefix. The `VetsProperties` record binds these settings:

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

For example, in your `application.properties` or `application.yml`:

```properties
vets.cache.ttl=60
vets.cache.heapSize=100
```

## API Specification

The following is an excerpt from the OpenAPI specification for the `GET /vets` endpoint:

```yaml
openapi: 3.0.3
info:
  title: API Documentation
  description: Auto-generated API documentation
  version: 1.0.0
paths:
  /vets:
    get:
      summary: Retrieve a list of veterinarians
      description: Returns a list of all veterinarians from the database. Caching behavior is governed by the `vets` configuration properties.
      operationId: showResourcesVetList
      tags:
      - Vet
      responses:
        '200':
          description: List of veterinarians
        '500':
          description: Internal server error
tags:
- name: Vet
```