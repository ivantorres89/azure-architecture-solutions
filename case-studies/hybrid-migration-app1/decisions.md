# Architecture Decision Records (ADRs)

This document captures the **final architecture decisions** for the **App1 hybrid migration** case study, based on the requirements and target diagram described in `README.md`.

Format: **Context → Decision → Consequences**.

---

## ADR-001 — Hybrid connectivity via ExpressRoute (private peering)

**Status:** Accepted

### Context
App1 must remain connected to on-premises systems with **predictable latency** and without exposing traffic over the public internet.

### Decision
Use **ExpressRoute** (private peering) to connect on-premises to the Azure VNet hosting App1 and its dependencies, terminating on an **ExpressRoute gateway**.

### Consequences
- Lower latency and higher reliability than internet VPN.
- Additional circuit and gateway costs; requires network-provider coordination.
- Network routing/DNS becomes an explicit operational concern (change control + monitoring).

---

## ADR-002 — Compute isolation with Azure Dedicated Hosts

**Status:** Accepted

### Context
A hard compliance requirement mandates **no shared physical infrastructure** with other tenants.

### Decision
Run App1 VMs on **Azure Dedicated Hosts**, grouped per Availability Zone (host groups aligned to zones).

### Consequences
- Strong isolation guarantees and compliance alignment.
- Higher baseline cost and capacity planning requirements (host-level reservations/limits).
- Scale-out is constrained by available host capacity; scaling events must be pre-planned.

---

## ADR-003 — Zonal VM Scale Sets for N+2 zone resilience

**Status:** Accepted

### Context
App1 must survive the loss of **two Availability Zones** in the selected region.

### Decision
Deploy **three zonal VM Scale Sets** (one per zone), each pinned to its zone, and size for **N+2** capacity across zones.

### Consequences
- Resilience against zone-level failures with predictable fault domains.
- More infrastructure to manage (multiple VMSS, per-zone capacity and patching strategy).
- Requires careful capacity planning (minimum instances per zone, scaling policies).

---

## ADR-004 — Split DB1 and DB2 into separate Azure SQL Databases

**Status:** Accepted

### Context
DB1 and DB2 are logically independent and benefit from independent scaling, cost control, and blast-radius isolation.

### Decision
Migrate DB1 and DB2 to **two separate Azure SQL Databases** (PaaS), rather than co-hosting on a single SQL Server VM.

### Consequences
- Independent scaling and operational isolation per database.
- Cross-database operations must be handled at the application layer (no implicit instance-level coupling).
- Cost can be slightly higher than shared IaaS, accepted for availability and reduced ops.

---

## ADR-005 — Business Critical tier + zone redundancy for DB1/DB2

**Status:** Accepted

### Context
The workload requires **low-latency OLTP** and stringent availability targets (RPO≈0, low RTO).

### Decision
Use **Azure SQL Database – Business Critical** with **zone redundancy** in an Azure region that supports Availability Zones.

### Consequences
- Built-in HA with automatic failover across zones.
- Higher cost than General Purpose; accepted for latency and availability.
- Capacity planning must consider service tier limits and cost governance.

---

## ADR-006 — Immutable data retention using Azure Blob immutability (WORM)

**Status:** Accepted

### Context
Regulatory requirements mandate **write-once, read-many (WORM)** retention for 3 years.

### Decision
Store application files in **Azure Storage (Blob)** with **immutability policies** (time-based retention) for 3 years, using dedicated containers for immutable content.

### Consequences
- Enforced retention and auditability at the platform level.
- Deletions/overwrites are restricted; lifecycle management must be intentional.
- Increased storage cost over time; requires retention governance (tiers, archival strategy).

---

## ADR-007 — Private-only access to Storage and SQL via Private Link

**Status:** Accepted

### Context
The target state requires **no public endpoints** for Storage and database access.

### Decision
Disable public network access where supported and use:
- **Private Endpoints** for Azure Storage and Azure SQL Database
- **Private DNS zones** for name resolution inside the VNet

### Consequences
- Strong reduction of data-exfiltration surface area.
- Requires DNS design and operations (split-horizon DNS, private zone linking).
- Troubleshooting connectivity becomes more network/DNS-centric.

---

## ADR-008 — Use Managed Identity for service-to-service access

**Status:** Accepted

### Context
Static credentials (service principals, keys) increase secret-handling risk and operational burden.

### Decision
Use **system-assigned Managed Identity** on the App1 VM instances to access Azure resources (Storage RBAC, database auth where applicable), enforcing least privilege.

### Consequences
- No secrets to rotate for platform access paths.
- Requires RBAC role design and audit (identity governance becomes first-class).
- Some legacy integrations may still require secrets (handled separately with strict controls).

---

## ADR-009 — Centralized governance at Management Group scope

**Status:** Accepted

### Context
Multiple subscriptions and environments require consistent, scalable governance.

### Decision
Apply **RBAC and governance controls at Management Group scope**, and enforce **Conditional Access** for production administration (MFA + compliant device).

### Consequences
- Consistent controls across subscriptions with reduced drift.
- Faster onboarding/offboarding and clearer separation of duties.
- Requires disciplined role design and periodic access reviews.

---
