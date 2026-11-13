# Eureka Server — Service Registry

Service registry for the microservices, built with **Spring Cloud Netflix Eureka Server**. Orders, Payments and the API Gateway register here and use it to discover each other without hard-coded addresses.

![Java](https://img.shields.io/badge/Java-17-orange) ![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5-green) ![Eureka](https://img.shields.io/badge/Spring%20Cloud-Netflix%20Eureka-brightgreen)

## Tech stack
- Java 17, Spring Boot 3.5, Spring Cloud 2025.0
- Spring Cloud Netflix Eureka Server
- Maven

## Configuration
- Runs on port `8081`.
- Standalone mode: the server does not register itself or fetch the registry (`register-with-eureka=false`, `fetch-registry=false`).
- Dashboard: `http://localhost:8081`

## Architecture

This service is part of a small **food-delivery backend built with Java and Spring Cloud microservices**:

| Repository | Responsibility | Default port |
|---|---|---|
| [spring-microservices-eureka-server](https://github.com/gomeschristopher/spring-microservices-eureka-server) | Service registry (Netflix Eureka) | `8081` |
| [spring-microservices-gateway-api](https://github.com/gomeschristopher/spring-microservices-gateway-api) | API Gateway (Spring Cloud Gateway) with discovery-based routing | `8082` |
| [spring-microservices-orders-api](https://github.com/gomeschristopher/spring-microservices-orders-api) | Orders REST API (`pedidos-ms`) | `8080` (default) |
| [spring-microservices-payments-api](https://github.com/gomeschristopher/spring-microservices-payments-api) | Payments REST API (`payments-ms`) | random (`server.port=0`) |

```mermaid
flowchart LR
    Client -->|HTTP| Gateway[API Gateway :8082]
    Gateway -->|lb://| Orders[pedidos-ms]
    Gateway -->|lb://| Payments[payments-ms]
    Orders -. register .-> Eureka[Eureka Server :8081]
    Payments -. register .-> Eureka
    Gateway -. discover .-> Eureka
    Payments -->|OpenFeign + Circuit Breaker| Orders
    Payments -->|publish| EX{{RabbitMQ fanout<br/>pagamentos.ex}}
    EX --> Q[(pagamentos.detalhes-pedido)]
    Q --> Orders
    Q -. on failure .-> DLQ[(pagamentos.detalhes-pedido-dlq)]
    Orders --> DB1[(MySQL)]
    Payments --> DB2[(MySQL)]
```

**Communication patterns**
- **Synchronous:** Payments calls Orders through an **OpenFeign** client resolved by Eureka, protected by a **Resilience4j circuit breaker** with a fallback.
- **Asynchronous:** when a payment is created, Payments publishes an event to a **RabbitMQ fanout exchange**; Orders consumes it with listener retries and a **dead-letter queue**.

## Running locally

Start this service first, then the API Gateway and the business services.

```bash
./mvnw spring-boot:run
```

## About this project

Study project built while following Alura's microservices training (example domain: *AluraFood*), used to practice service discovery, API gateway routing, inter-service communication, resilience and messaging with the Spring ecosystem.

**Author:** Christopher Gomes — [LinkedIn](https://www.linkedin.com/in/christophgomes/) · [GitHub](https://github.com/gomeschristopher)
