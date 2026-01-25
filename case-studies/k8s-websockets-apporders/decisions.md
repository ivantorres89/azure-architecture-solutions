# Architecture and Design Decisions

---

## Asynchronous Processing Model

### Decision
All order processing is performed asynchronously.

### Rationale
- Prevents frontend blocking
- Absorbs traffic spikes
- Decouples ingestion from processing
- Enables independent scaling of system components

This aligns with event-driven architecture patterns used in high-throughput systems.

---

## CorrelationId vs OrderId

### Decision
Use two distinct identifiers:
- CorrelationId for technical workflow tracking
- OrderId as the database-generated business identifier

### Rationale
- Orders must be accepted before database interaction
- Asynchronous workflows require correlation across queues, Redis, and WebSockets
- Business identity remains a persistence concern
- Improves observability and traceability across distributed components

---

## API Gateway Selection (Azure API Management)

### Decision
Use Azure API Management as an **edge API gateway** in front of the Kubernetes cluster.

### Rationale
APIM is used as a **perimeter and governance layer**, not as a business gateway.

Its responsibilities are limited to:
- JWT authentication at the edge
- Rate limiting and quotas
- Traffic shaping and backend protection
- Acting as a stable public entry point

All business authorization and routing logic remains inside the Kubernetes cluster.

---

## Integration with External Identity Providers

### Decision
Decouple identity management from backend services and integrate authentication at the API gateway layer.

### Rationale
Azure API Management is **identity-provider agnostic** and supports validation of JWTs issued by any OAuth 2.0 / OpenID Connect–compliant provider.

This allows the system to integrate with:
- Cloud-native IdPs
- Corporate identity platforms
- Hybrid or multi-cloud identity solutions

### Example Identity Providers
Typical examples of compatible IdPs include:
- Azure Active Directory / Entra ID
- Keycloak
- Auth0
- Okta

In all cases:
- The IdP issues JWT access tokens
- APIM validates token authenticity, issuer, and audience
- User identity claims are forwarded to backend services

The Identity Provider itself remains outside the scope of this architecture.

---

## Application Runtime

### Decision
Deploy all backend services on Azure Kubernetes Service.

### Rationale
- Enables stateless service design
- Supports horizontal scaling
- Provides workload isolation
- Aligns with modern cloud-native practices

Only a single cluster is shown for conceptual clarity.  
The architecture supports zonal and multi-cluster deployments without requiring logical changes.

---

## State Management

### Decision
All services are stateless.

### Rationale
- Simplifies scaling and deployments
- Improves fault tolerance
- Reduces coupling between services

State is externalized to:
- Azure SQL Database (business state)
- Redis (session and correlation state)

---

## Redis Usage

### Decision
Use Redis as a short-lived correlation and session registry.

### Rationale
- Enables real-time notification in asynchronous workflows
- Avoids sticky sessions for WebSocket connections
- Supports horizontal scaling of notification services
- TTL-based cleanup prevents stale state accumulation

Redis is not used as a message broker or system of record.

---

## Messaging Strategy

### Decision
Use Azure Service Bus queues with FIFO semantics.

### Rationale
- Reliable delivery with retry support
- Built-in dead-letter queues for poison messages
- Well suited for transactional, ordered workflows

Topics and fan-out are intentionally excluded to keep the workflow simple and predictable.

---

## Failure Handling

### Decision
Adopt at-least-once delivery with defensive consumers.

### Rationale
- Consumers are idempotent
- Transient failures are retried automatically
- Poison messages are isolated via dead-letter queues

Dead-letter queues are treated as an operational concern and monitored separately.

---

## Design Philosophy

This architecture prioritizes:
- Clear responsibility boundaries
- Predictable behavior under load
- Operational safety over theoretical optimization

Capacity planning and resource sizing are intentionally deferred to operational metrics rather than upfront assumptions.