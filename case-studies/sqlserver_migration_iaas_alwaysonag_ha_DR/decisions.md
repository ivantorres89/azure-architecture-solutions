# Architecture Decision Records (ADRs)

This document captures the **final architecture decisions** for the **SQL Server IaaS migration using Always On Availability Groups** case study, based on the scenario described in `README.md` and the target diagram.

Format: **Context → Decision → Consequences**.

---

## ADR-001 — Use Always On Availability Groups (AG) for HA/DR (IaaS)

**Status:** Accepted

### Context
The database is a Tier-1 OLTP workload requiring **automatic failover within the primary region** and **near-zero data loss** during common infrastructure failures, without relying on shared storage.

### Decision
Use **SQL Server Always On Availability Groups** as the primary resilience mechanism:
- **Synchronous** replicas across Availability Zones in the primary region for HA.
- An **asynchronous** replica in a secondary region for DR.

### Consequences
- Enables database-level replication without shared storage dependencies.
- Requires **SQL Server Enterprise Edition** and operational maturity (replicas, quorum, monitoring).
- Application connectivity must target the **AG listener**, not individual instances.

---

## ADR-002 — Automatic failover is intra-region only

**Status:** Accepted

### Context
Cross-region automatic failover with a stretched WSFC/AG can introduce quorum complexity, split-brain risk, and increased latency across regions.

### Decision
Configure **automatic failover only between the synchronous replicas in the primary region**. Treat the secondary region replica as **DR** with **manual or orchestrated failover** (runbook-driven).

### Consequences
- Predictable and safe HA failover inside the region.
- DR recovery requires an explicit operational procedure and testing.
- RTO for regional outages is higher than intra-region failover but aligns with typical enterprise DR practice.

---

## ADR-003 — Eliminate shared storage; use local Premium SSD managed disks

**Status:** Accepted

### Context
Shared storage adds a single point of failure and is difficult to make zone- and geo-resilient for SQL FCI-style designs.

### Decision
Use **local managed disks** (Premium SSD) per SQL VM and rely on **AG database replication** for data redundancy.

### Consequences
- Removes dependency on shared SMB storage and improves failure isolation.
- Requires a clear backup/restore strategy (Azure Backup / native SQL backups to Storage).
- Each replica must be sized for independent storage capacity and IO requirements.

---

## ADR-004 — Client connectivity via AG listener behind an internal Load Balancer

**Status:** Accepted

### Context
Clients require a stable endpoint regardless of which replica is primary.

### Decision
Expose the **AG listener** through an **internal Standard Load Balancer** in the primary region, with health probing and backend pool membership for participating replicas.

### Consequences
- Stable connection endpoint for the application tier.
- Additional configuration complexity (ILB, probe ports, WSFC/AG listener settings).
- Load balancer becomes part of the critical path and must be monitored.

---

## ADR-005 — Separate VNets per region; controlled replication connectivity

**Status:** Accepted

### Context
Replication traffic must be secured and controlled across regions, with clear blast radius boundaries.

### Decision
Use **separate VNets per region** and provide controlled connectivity for replication and management (peering/VPN/ExpressRoute), restricted by NSGs and required SQL/WSFC ports.

### Consequences
- Clear fault domains and isolation between regions.
- Requires network design for DNS, routing, and port governance.
- Cross-region data transfer costs must be considered.

---

## ADR-006 — Reject VM-level DR (ASR) as the primary database strategy

**Status:** Accepted

### Context
VM-level replication (ASR) is coarse-grained and typically yields higher RPO/RTO for database workloads compared to database-native replication.

### Decision
Use **AG replication** for database continuity and failover, rather than making ASR the primary DR mechanism for SQL.

### Consequences
- Better control over data replication and database recovery behavior.
- Still requires separate considerations for OS/config drift (patching, hardening, automation).
- Demands robust monitoring and regular failover testing.

---
