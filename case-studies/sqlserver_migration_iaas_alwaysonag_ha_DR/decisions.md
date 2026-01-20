# Architectural Decisions & Trade-offs

This document outlines the key architectural decisions taken to design a
highly available and resilient SQL Server architecture on Azure using
Always On Availability Groups.

---

## 1. Use of SQL Server Always On Availability Groups (AG)

**Decision**  
Use **SQL Server Always On Availability Groups** as the core HA and DR mechanism.

**Rationale**
- Enables automatic failover without shared storage
- Supports both zone-level and region-level resilience
- Provides low RTO and near-zero RPO
- Aligns with cloud-native design principles for Azure IaaS

---

## 2. Elimination of Shared Storage

**Decision**  
Avoid shared storage entirely.

**Rationale**
- Availability Groups replicate data at the database level
- Each SQL node maintains its own local storage
- Removes dependency on Azure FileStorage Premium

**Benefits**
- No storage bottleneck
- No storage recovery during DR
- Improved scalability and reliability

---

## 3. High Availability via Availability Zones

**Decision**  
Distribute AG replicas across **Availability Zones** in the primary region.

**Rationale**
- Protects against datacenter-level failures
- Provides automatic intra-region failover
- No manual intervention required

---

## 4. Disaster Recovery via Cross-Region Availability Group Replication

**Decision**  
Configure an AG replica in a secondary Azure region with automatic failover.

**Rationale**
- Native SQL Server replication mechanism
- Faster recovery compared to VM-based DR solutions
- No cluster rebuild required during failover

**Outcome**
- True active/passive (or active/active) regional DR
- Predictable and low RTO/RPO

---

## 5. Why Failover Cluster Instance (FCI) Was Rejected

**Rejected Option**  
SQL Server Always On Failover Cluster Instance (FCI)

**Reasons for Rejection**

- **Shared storage dependency**  
  FCI requires shared writable storage, which cannot be replicated or geo-redundant in Azure.

- **No automatic cross-region failover**  
  FCI does not support automatic failover across regions.

- **Higher RTO for DR**  
  Disaster recovery requires VM replication, storage restore, and cluster rebuild.

- **Operational complexity in Azure**  
  Load balancers, storage recovery, and cluster management increase failure points.

- **Not aligned with cloud-native resilience**  
  FCI is suitable for lift-and-shift but suboptimal for modern Azure architectures.

---

## 6. Why Azure Site Recovery (ASR) Was Not Used

**Rejected Option**  
Azure Site Recovery for SQL failover

**Reasons for Rejection**
- ASR operates at VM level, not database level
- Slower recovery compared to AG failover
- Requires additional orchestration and validation
- Unnecessary when AG provides native replication

---

## Final Assessment

This architecture prioritizes:
- Automatic failover
- Low RTO and RPO
- Regional fault isolation
- Cloud-aligned design

In exchange for:
- Higher SQL Server licensing cost
- Increased architectural complexity
- Application readiness requirements

This design represents a **modern, cloud-optimized SQL Server IaaS architecture**,
suitable for business-critical workloads with strict availability objectives.
