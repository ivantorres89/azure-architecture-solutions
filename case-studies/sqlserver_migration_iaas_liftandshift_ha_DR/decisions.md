# Architecture Decision Records (ADRs)

This document captures the **final architecture decisions** for the **SQL Server IaaS lift-and-shift (FCI) HA/DR** case study, based on the scenario described in `README.md` and the target diagram.

Format: **Context → Decision → Consequences**.

---

## ADR-001 — Keep SQL Server Failover Cluster Instance (FCI) for lift-and-shift

**Status:** Accepted

### Context
The existing platform is tightly coupled to **WSFC + FCI** and depends on instance-level behavior and OS-level agents. The primary objective is **minimal change** and reduced migration risk.

### Decision
Migrate using **SQL Server Always On Failover Cluster Instance (FCI)** on Azure VMs, preserving the existing topology and operational model.

### Consequences
- Maximum compatibility with existing operational practices and third-party tooling.
- Requires shared storage and careful cluster networking configuration in Azure.
- DR is inherently more complex than AG/PaaS approaches.

---

## ADR-002 — Shared storage via Azure Files Premium (SMB)

**Status:** Accepted

### Context
FCI requires shared writable storage. In Azure, the commonly supported shared storage option for FCI is SMB-based storage.

### Decision
Use **Azure Files Premium (SMB)** as the shared storage layer for the FCI in the primary region.

### Consequences
- Enables an FCI architecture without introducing Storage Spaces Direct complexity.
- Shared storage remains a single-region dependency (no native cross-region active replication).
- Storage performance/capacity must be provisioned and monitored independently.

---

## ADR-003 — HA is confined to a single region

**Status:** Accepted

### Context
Azure Files SMB mounts and the FCI shared storage model cannot be stretched across regions in a supported, writeable manner.

### Decision
Deliver **automatic HA within the primary region only** (node/host failures), using FCI failover and Azure infrastructure constructs (Availability Zones or Availability Set).

### Consequences
- Zero (or near-zero) data loss for intra-region failures.
- Regional outage recovery is DR, not HA, and requires additional steps.
- Architectural constraints are explicit and predictable for stakeholders.

---

## ADR-004 — Client connectivity via internal Load Balancer + DNS indirection

**Status:** Accepted

### Context
Clients need a stable name to connect to the SQL instance while the active node can change.

### Decision
Use an **internal Load Balancer** for the cluster/instance endpoint and publish a stable DNS name for the application to use.

### Consequences
- Stable endpoint for intra-region failover.
- Load balancer configuration is non-trivial and must be validated during failover tests.
- In regional DR scenarios, DNS/connection targeting may need explicit switching depending on the runbook.

---

## ADR-005 — DR via Azure Site Recovery for VMs, plus Azure Backup for Azure Files

**Status:** Accepted

### Context
The business requires recovery from a full regional outage, but accepts **RPO > 0** and a higher RTO typical of lift-and-shift DR.

### Decision
Use:
- **Azure Site Recovery (ASR)** to replicate SQL VMs to the DR region.
- **Azure Backup** to restore Azure Files Premium in the DR region (since Azure Files is not replicated by ASR).

### Consequences
- DR orchestration is multi-step (VM recovery + storage restore + cluster validation).
- RTO increases due to storage recovery and cluster bring-up.
- RPO depends on backup frequency and restore point selection.

---

## ADR-006 — Explicitly separate HA and DR responsibilities

**Status:** Accepted

### Context
Mixing HA and DR mechanisms can lead to unsupported designs and unclear recovery expectations.

### Decision
Adopt a deliberate split:
- **HA (within region):** FCI + Azure infra redundancy
- **DR (cross region):** ASR + Azure Backup + runbook-driven activation

### Consequences
- Clear operational playbooks and predictable failure domains.
- Higher DR effort than AG/PaaS, accepted to preserve lift-and-shift constraints.
- Requires regular DR drills to keep procedures reliable.

---
