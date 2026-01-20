# Architectural Decisions & Trade-offs

This document records the key architectural decisions made during the design of the
SQL Server IaaS HA/DR solution, including rationale and known trade-offs.

---

## 1. Use of SQL Server Failover Cluster Instance (FCI)

**Decision**  
Use **SQL Server Always On Failover Cluster Instance (FCI)** as the core HA mechanism.

**Rationale**
- Existing on-premises architecture already uses FCI
- Enables true lift-and-shift with minimal refactoring
- Preserves OS-level control and compatibility
- No dependency on SQL Server Enterprise features such as Availability Groups

**Trade-off**
- Requires shared storage
- More complex networking than PaaS
- Limited cross-region capabilities

---

## 2. High Availability Scope Limited to Single Region

**Decision**  
Implement **HA only within the primary Azure region**.

**Rationale**
- FCI requires **shared writable storage**
- Azure FileStorage Premium can only be mounted **within a single region**
- Cross-region shared storage is not supported by Azure

**Outcome**
- Automatic failover between SQL nodes within the region
- Zero data loss for node-level or host-level failures

---

## 3. Azure FileStorage Premium as Shared Storage

**Decision**  
Use **Azure FileStorage Premium (SMB)** for FCI shared disks.

**Rationale**
- Fully supported for SQL Server FCI in Azure
- Provides low-latency, high-IOPS storage
- Simplifies lift-and-shift compared to Storage Spaces Direct

**Trade-off**
- No geo-replication (GRS not supported)
- Storage must be rebuilt or restored during DR

---

## 4. Disaster Recovery Implemented with Azure Site Recovery (ASR)

**Decision**  
Use **Azure Site Recovery** for cross-region disaster recovery.

**Rationale**
- ASR replicates entire virtual machines
- Suitable for IaaS workloads
- Allows recovery in a paired Azure region

**Key Limitation**
- ASR does **not replicate Azure FileStorage**
- Only VM disks are replicated

---

## 5. Storage Recovery via Azure Backup

**Decision**  
Use **Azure Backup** for FileStorage recovery in DR region.

**Rationale**
- FileStorage Premium cannot be replicated across regions
- Backups provide point-in-time recovery
- Aligns with accepted RPO > 0

**Trade-off**
- Storage restore increases RTO
- Manual or semi-automated recovery steps required

---

## 6. Separation of HA and DR Responsibilities

**Decision**  
Explicitly separate **HA** and **DR** responsibilities.

| Capability | Technology |
|-----------|-----------|
| HA (node failure) | SQL Server FCI |
| HA (infrastructure) | Availability Set / Zones |
| DR (regional outage) | Azure Site Recovery + Backup |

**Rationale**
- Avoids unsupported architectures
- Aligns with Azure platform constraints
- Provides predictable failure domains

---

## 7. Why AlwaysOn Availability Groups Were Not Used

**Rejected Option**  
SQL Server Always On Availability Groups (AG)

**Reasons for Rejection**

- **Not a true lift-and-shift**  
  Availability Groups require changes to the SQL topology and often to application connection handling.

- **Application compatibility risk**  
  Existing applications were tightly coupled to a single SQL instance and relied on instance-level features not easily portable to AGs.

- **Licensing constraints**  
  Availability Groups require SQL Server Enterprise Edition, increasing operational costs.

- **Operational maturity**  
  The existing operations team had deep expertise with WSFC + FCI but limited experience managing AG failover, replicas, and listener behavior.

- **Migration risk over architectural elegance**  
  The primary objective was to reduce migration risk rather than maximize cloud-native features.

**Conclusion**  
While Availability Groups offer superior cross-region failover capabilities, they were intentionally excluded to preserve compatibility, reduce risk, and align with the lift-and-shift strategy.

---

## Final Assessment

This architecture intentionally accepts:
- Higher RTO compared to PaaS
- Manual intervention during DR

In exchange for:
- Maximum compatibility
- Minimal migration risk
- Full infrastructure control

This is a **realistic enterprise IaaS design**, not a theoretical or exam-only solution.
