# Project Stack Plan

## Objective

Build the QualificaSP Delivery API as a production-ready backend for delivery
management, with an HTTP API, asynchronous event processing, automated tests,
containerized local development, and a CI/CD path to the cloud.

## Baseline Already Present

- Java 21
- Spring Boot 4.0.6
- Maven Wrapper
- Docker Compose
- GitHub Actions as the CI platform

## Proposed Stack

### Application

- Language/runtime: Java 21
- Framework: Spring Boot 4
- API: Spring Web MVC with REST and JSON
- Validation: Bean Validation via `spring-boot-starter-validation`
- API contract: OpenAPI/Swagger documentation
- Configuration: Spring profiles and environment variables; no secrets in Git
- Health and metrics: Spring Boot Actuator with Micrometer

### Data and Messaging

- Primary database: PostgreSQL
- Persistence: Spring Data JPA with Flyway migrations
- Messaging: Apache Kafka for delivery lifecycle events
- Local infrastructure: Docker Compose with PostgreSQL and Kafka
- Event contract: Versioned event schemas, explicit keys, retries, and a
  dead-letter topic

### Service Architecture

- Start with a modular Spring Boot application and clear domain boundaries so
  the initial delivery stays simple to operate.
- Define the first boundary around the delivery/order lifecycle. Extract a
  separate microservice only when it has an independent scaling, deployment,
  or ownership need.
- Keep synchronous REST commands separate from asynchronous Kafka event
  handlers, with idempotent consumers and correlation IDs.

### Testing

- Unit/TDD: JUnit 5, Mockito, and AssertJ for domain and application logic
- BDD: Cucumber scenarios for business-critical delivery flows
- Integration: Spring Boot Test with Testcontainers for PostgreSQL and Kafka
- API: MockMvc or RestAssured for endpoint and contract coverage
- Quality gates: Checkstyle/Spotless and JaCoCo coverage thresholds in CI

### Delivery and Operations

- Containers: Multi-stage Docker image built from the Maven project
- Local orchestration: Docker Compose
- Production target: Vercel for the public backend API, using Vercel
  Functions and environment-based configuration
- Managed data services: Use a Vercel-compatible hosted PostgreSQL provider
  and a managed Kafka provider rather than running stateful infrastructure on
  Vercel
- Runtime constraint: Vercel Functions are request-driven and do not host
  long-running Spring Boot processes or Kafka consumers. Confirm the runtime
  strategy before implementation: either adapt the backend to a Vercel-native
  runtime or isolate asynchronous consumers in a separately deployed worker.
- CI: GitHub Actions for build, test, static checks, image build, and
  vulnerability scanning
- CD: Connect the protected production branch to Vercel, use preview
  deployments for pull requests, and configure environment-specific secrets
  with rollback to a previous deployment

## Delivery Phases

1. Foundation: Rename the generated application metadata, establish package
   structure, add REST, validation, persistence, Flyway, Actuator, and local
   PostgreSQL configuration.
2. Core delivery flow: Define the delivery aggregate, status transitions,
   database schema, REST commands/queries, error responses, and API contract.
3. Events: Publish delivery lifecycle events to Kafka and implement an
   idempotent consumer with retry and dead-letter handling.
4. Test strategy: Add unit tests first, then Cucumber business scenarios,
   Testcontainers integration tests, and endpoint/contract tests.
5. Operational readiness: Add structured logs, correlation IDs, metrics,
   health probes, database migration checks, and Docker Compose documentation.
6. Continuous integration: Add GitHub Actions for formatting, tests,
   coverage, dependency/security scanning, and container image publication.
7. Vercel deployment: Confirm the Java/Spring Boot runtime strategy, connect
  the API to Vercel, configure hosted PostgreSQL/Kafka credentials, and
  document preview, production, and rollback workflows.
8. Service extraction review: Measure load and ownership boundaries before
  splitting modules into independently deployed services or adding a separate
  worker for asynchronous processing.

## Initial Definition of Done

- A versioned REST API can create, retrieve, and update a delivery through
  valid domain transitions.
- Invalid requests return consistent, documented error responses.
- Delivery lifecycle events are published once from the application’s point of
  view and consumers safely tolerate redelivery.
- Unit, BDD, integration, and API tests run locally and in GitHub Actions.
- The application and its local dependencies start with Docker Compose.
- Health, logs, metrics, configuration, and deployment instructions are
  documented before the first cloud release.

## Decisions To Confirm Before Implementation

- Exact delivery actors and statuses, including cancellation and failure rules.
- Whether authentication is required in the first release; if so, use an OIDC
  provider and JWT validation rather than custom credential handling.
- Vercel-compatible runtime choice: retain Java/Spring Boot with a separately
  hosted worker, or use a Vercel-native runtime for the API.
- Expected throughput, retention, and ordering requirements for Kafka topics.
- Data retention, privacy, and audit requirements for delivery records.
