# Architecture & Design Decisions

---

## 1. Fully Asynchronous Architecture

### Decision
All order processing is fully asynchronous.

### Rationale
- Prevents frontend blocking
- Absorbs traffic spikes
- Enables independent scaling of ingestion and processing
- Aligns with event-driven architecture best practices

---

## 2. Separation of CorrelationId and OrderId

### Decision
Use two distinct identifiers:
- `CorrelationId` for asynchronous workflow tracking
- `OrderId` as the database-generated business identifier

### Rationale
- Orders must be accepted before database interaction
- Correlation is required across queues, Redis, and WebSockets
- Database identity should remain a persistence concern
- Improves traceability and observability

---

## 3. Azure Service Bus Queues (FIFO)

### Decision
Use Azure Service Bus Queues with FIFO semantics.

### Rationale
- Reliable message delivery
- Built-in retries and dead-letter queues
- Session support for FIFO ordering
- Better suited for transactional workflows than Event Grid

---

## 4. Kubernetes as the Application Runtime

### Decision
Deploy all backend services on Azure Kubernetes Service.

### Rationale
- Horizontal scalability
- Stateless service model
- Clear separation of responsibilities
- Industry-standard orchestration platform

### Scope Note
Only a single AKS cluster is shown in the diagram for clarity.
The architecture supports:
- Zonal node pools
- Multi-cluster active-active setups
These are intentionally omitted from the diagram to avoid unnecessary complexity.

---

## 5. Stateless Services and Externalized State

### Decision
All services are stateless.

### Rationale
- Enables horizontal scaling
- Simplifies deployments and rollbacks
- Improves fault tolerance

State is externalized to:
- Azure SQL Database (business state)
- Redis (correlation and session state)

---

## 6. Redis as a Correlation and Session Registry

### Decision
Use Redis as a short-lived registry mapping `CorrelationId` to WebSocket connection metadata.

### Rationale
- Enables real-time notification in async workflows
- Avoids sticky sessions
- Supports horizontal scaling of WebSocket servers
- TTL ensures automatic cleanup

Redis is **not** used as:
- A message broker
- A source of truth
- A business data store

---

## 7. Real-Time Communication via WebSockets

### Decision
Implement a dedicated WebSocket service for real-time notifications.

### Rationale
- Clear separation of concerns
- Decouples order processing from user communication
- Improves scalability and resilience

---

## 8. Database Choice: Azure SQL Database – Business Critical

### Decision
Use Azure SQL Database Business Critical tier.

### Rationale
- Predictable low latency
- High IOPS for sustained OLTP workloads
- Built-in high availability
- Strong consistency guarantees

Cost optimization is intentionally secondary to reliability.

---

## 9. Failure Handling and Reliability

- At-least-once message delivery
- Idempotent consumers
- Dead-letter queues for poison messages
- Correlation-based traceability

---

