# Architecture Decision Records (ADRs)

This document captures the **final architecture decisions** for the **multi-region App1 (App Service) architecture** case study, based on the target diagram in this folder.

Format: **Context → Decision → Consequences**.

---

## ADR-001 — Active/active multi-region deployment

**Status:** Accepted

### Context
The application must remain available during a regional outage and provide low latency to users in different geographies.

### Decision
Deploy App1 in **two regions** in an **active/active** topology (both regions serve production traffic under normal conditions).

### Consequences
- Improved availability and latency for global users.
- Requires that the application tier is **stateless** (or that state is externalized) to support safe failover.
- Operational overhead increases (multi-region deployments, monitoring, and incident response).

---

## ADR-002 — Global routing with Azure Traffic Manager (DNS-based)

**Status:** Accepted

### Context
A global entry point is required to route users to the healthiest/closest region with minimal architectural coupling.

### Decision
Use **Azure Traffic Manager** to route traffic at the DNS level to regional endpoints using a health probe + routing policy (performance/priority depending on SLOs).

### Consequences
- Simple, globally distributed routing with clear failover semantics.
- Failover depends on DNS TTL and probe configuration (not instant).
- Application must tolerate reconnections/region switches during failover events.

---

## ADR-003 — Regional L7 ingress via Application Gateway + WAF

**Status:** Accepted

### Context
Enterprise requirements include layer-7 routing control and protection against common web threats.

### Decision
Use **Application Gateway v2** in each region as the L7 ingress, with **WAF enabled** (central policy, consistent rule set across regions).

### Consequences
- Centralized L7 routing, TLS termination, and WAF enforcement per region.
- Adds cost and operational responsibility (WAF tuning, false positives, rule lifecycle).
- Provides a clear “regional perimeter” for observability and security controls.

---

## ADR-004 — Keep App1 stateless; externalize session/state

**Status:** Accepted

### Context
Multi-region traffic distribution and failover are unsafe if the application requires sticky sessions or stores state in-memory.

### Decision
Design App1 as **stateless**, and externalize state (sessions, caches, uploads) to dedicated backing services (e.g., distributed cache and durable storage), with a replication strategy appropriate to the RPO/RTO targets.

### Consequences
- Enables safe routing to any region and smooth recovery after failover.
- Requires explicit state design (consistency model, replication, and conflict behavior).
- Some features (uploads, long-running jobs) require additional patterns (queues, durable workflows).

---

## ADR-005 — Health probing layered per hop (DNS → Gateway → App)

**Status:** Accepted

### Context
Failover must be triggered based on real service health, not just VM/App Service liveness.

### Decision
Implement layered health checks:
- Traffic Manager probes the regional **Application Gateway** endpoint.
- Application Gateway probes the **App1 backend** health endpoint.

### Consequences
- Reduces false failovers and improves detection of partial failures.
- Requires operational discipline (SLO-backed probes, failure-mode testing).
- Misconfigured probes can cause cascading outages; monitoring and change control are mandatory.

---
