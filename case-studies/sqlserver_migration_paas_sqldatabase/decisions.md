# Architecture Decision Records (ADRs)

This document captures the **final architecture decisions** for the **SQL Server migration to Azure SQL Database (PaaS)** case study, based on `README.md` and the target diagram.

Format: **Context → Decision → Consequences**.

---

## ADR-001 — Choose Azure SQL Database for single-database, cloud-native workloads

**Status:** Accepted

### Context
The workload fits a **single-database** model and does not require instance-level features such as SQL Agent jobs, CLR, or cross-database queries.

### Decision
Migrate to **Azure SQL Database** (single database) to remove VM/OS management and adopt a cloud-native database platform.

### Consequences
- Significant reduction in operational overhead (patching, backups, HA managed by the platform).
- Some SQL Server features may need rework or replacement (by design).
- Enables simpler scaling and modernization patterns compared to IaaS.

---

## ADR-002 — Use Business Critical tier for HA + low-latency OLTP

**Status:** Accepted

### Context
The system is mission-critical OLTP with strict latency and availability requirements.

### Decision
Deploy Azure SQL Database in the **Business Critical** tier and enable **zone redundancy** where available.

### Consequences
- Built-in high availability with automatic failover.
- Higher cost than General Purpose; accepted for performance/availability.
- Service tier limits and cost governance must be actively managed.

---

## ADR-003 — Cross-region DR via Failover Group

**Status:** Accepted

### Context
Regional outage resilience is required with low RTO/RPO, and the solution must remain managed and repeatable.

### Decision
Configure an **Auto-failover group** between logical servers in two regions for coordinated geo-replication and failover.

### Consequences
- Stable read-write endpoint for the application during regional failover.
- Replication is asynchronous; a small data-loss window may exist depending on conditions.
- Requires DR drills, monitoring, and a clear failover decision process.

---

## ADR-004 — Use built-in read scale-out for reporting workloads

**Status:** Accepted

### Context
Read-heavy reporting can impact write latency on primary OLTP workloads.

### Decision
Use **read scale-out** (read-only replicas) for reporting/analytics traffic, keeping OLTP writes isolated on the primary.

### Consequences
- Better performance isolation and predictable OLTP latency.
- Read replicas are eventually consistent; reporting may lag slightly.
- Requires connection routing strategy for read-only traffic.

---

## ADR-005 — Restrict access to private networking where required

**Status:** Accepted

### Context
Enterprise security posture prefers private access patterns and reduced public exposure.

### Decision
Where network requirements apply, use **Private Endpoint** + private DNS and disable public access paths as appropriate for the environment.

### Consequences
- Reduced attack surface and better compliance alignment.
- Adds DNS/routing operational complexity.
- Connectivity troubleshooting requires network/DNS tooling and ownership.

---

## ADR-006 — Reject Azure SQL Managed Instance for this scenario

**Status:** Accepted

### Context
Managed Instance is optimized for high SQL Server compatibility and instance-level features, which are not required for this workload.

### Decision
Do not use **Azure SQL Managed Instance** in this case study; prefer Azure SQL Database for simplicity and cost/performance alignment.

### Consequences
- Leaner operational model and fewer network prerequisites than MI.
- Feature gaps are accepted because they are not needed by requirements.
- Keeps the architecture closer to cloud-native patterns (single database, managed HA/DR).

---
