# Architecture Decision Records (ADRs)

This document captures the **final architecture decisions** for the **SQL Server migration to Azure SQL Managed Instance (PaaS)** case study, based on `README.md` and the target diagram.

Format: **Context → Decision → Consequences**.

---

## ADR-001 — Choose Azure SQL Managed Instance for SQL Server feature compatibility

**Status:** Accepted

### Context
The workload relies on SQL Server capabilities typically not available (or not desirable) in single-database PaaS offerings (e.g., instance-level features, SQL Agent jobs, CLR, cross-database patterns).

### Decision
Migrate to **Azure SQL Managed Instance** to maximize compatibility while removing VM/OS management.

### Consequences
- Lower migration risk and reduced refactoring compared to SQL on VMs or Azure SQL Database.
- Requires VNet integration (MI is deployed inside a subnet) and associated network planning.
- Platform limits still apply; some edge features may require remediation.

---

## ADR-002 — Use Business Critical tier for OLTP latency and built-in HA

**Status:** Accepted

### Context
The system is Tier-1 OLTP with strict latency and availability needs, and benefits from built-in replicas and read scale-out.

### Decision
Deploy Managed Instance in the **Business Critical** tier, enabling **zone redundancy** where supported.

### Consequences
- Built-in HA with automatic failover and SSD-backed performance characteristics.
- Higher cost than General Purpose; accepted for latency and availability.
- Requires capacity sizing against vCore, storage, and IO characteristics of the tier.

---

## ADR-003 — Restrict data plane access to private networking

**Status:** Accepted

### Context
Enterprise database access must be controlled and not exposed via public endpoints.

### Decision
Deploy Managed Instance with **private access** (VNet-injected subnet) and restrict access from application tiers via network controls (NSGs/UDRs as appropriate).

### Consequences
- Strong network isolation and reduced attack surface.
- Requires DNS and routing design for consumers (especially in hybrid or multi-VNet environments).
- Network troubleshooting becomes a key operational area.

---

## ADR-004 — Cross-region DR via Failover Group (managed geo-replication)

**Status:** Accepted

### Context
The business requires regional disaster recovery with low RPO/RTO without building custom replication and failover tooling.

### Decision
Configure a **Failover Group** between a primary and secondary Managed Instance in different regions, using:
- Read-write listener endpoint for the application
- Optional read-only endpoint for reporting/read scale-out

### Consequences
- Coordinated failover with stable endpoints (no connection-string rewrite during failover).
- Replication is asynchronous; some data loss window may exist depending on conditions.
- Requires DR testing, monitoring, and agreed failover authority/process.

---

## ADR-005 — Use built-in platform backups + optional long-term retention

**Status:** Accepted

### Context
Operational overhead must be minimized while meeting compliance needs for backup retention.

### Decision
Rely on **platform-managed automated backups**, and configure **long-term retention (LTR)** where required by policy.

### Consequences
- Reduced DBA operational burden (no custom backup infrastructure).
- Restore operations and retention governance must be documented and periodically tested.
- Costs increase with longer retention and larger database footprints.

---

## ADR-006 — Prefer identity-based access and auditing as first-class controls

**Status:** Accepted

### Context
Security and traceability require least privilege access and auditability.

### Decision
Use:
- **Entra ID (Azure AD) authentication** where feasible
- Centralized auditing/monitoring (e.g., Log Analytics / Storage) per organizational standards

### Consequences
- Improved access governance and reduced reliance on shared SQL credentials.
- Requires role mapping and admin break-glass procedures.
- Audit data storage/retention must be managed and costed.

---
