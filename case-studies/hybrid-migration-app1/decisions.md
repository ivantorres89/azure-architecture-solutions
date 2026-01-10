# Architectural Decisions & Rationale – App1 Migration

This document captures the key architectural decisions made during the migration of App1 to Azure, including the reasoning behind each choice.

---

## 1. Separation of DB1 and DB2 into PaaS Databases

**Decision**  
Migrate DB1 and DB2 as **separate Azure SQL Databases** instead of a single SQL Server instance.

**Rationale**
- Databases are logically independent
- Azure SQL Database provides:
  - Built-in high availability
  - Automatic patching
  - Reduced operational overhead
- Enables independent scaling and cost control

**Trade-off**
- Slightly higher cost compared to shared IaaS SQL Server
- Accepted due to availability and compliance requirements

---

## 2. Azure SQL Business Critical Tier

**Decision**  
Use **Business Critical** tier with zone redundancy.

**Rationale**
- Required **RPO = 0** and **RTO ≈ 0**
- Automatic failover across availability zones
- Low I/O latency due to local SSD storage

**Rejected Alternatives**
- General Purpose: insufficient availability
- SQL on VM: higher admin effort

---

## 3. Dedicated Hosts for App1

**Decision**  
Deploy App1 on **Azure Dedicated Hosts**.

**Rationale**
- Explicit requirement: no shared physical infrastructure
- Compliance and isolation guarantees
- Supports VM Scale Sets and Availability Zones

---

## 4. Multi-AZ Deployment with VM Scale Sets

**Decision**  
Deploy App1 across **three Availability Zones**, each with its own VM Scale Set.

**Rationale**
- Required to survive the loss of two zones
- Enables independent fault domains
- Supports automatic scaling and resilience

---

## 5. Managed Identity for App1

**Decision**  
Use **system-assigned managed identity** for App1.

**Rationale**
- Eliminates credential management
- Integrates natively with Azure RBAC
- Least-privilege access model

**Rejected Alternatives**
- Service principals (manual secret rotation)
- Shared keys (security risk)

---

## 6. Azure Storage with Immutable Blob Policies

**Decision**  
Store application data in Azure Storage with **immutable blob policies (WORM)**.

**Rationale**
- Mandatory 3-year write-once requirement
- Meets regulatory and audit constraints
- Azure-native compliance feature

---

## 7. Private Access to Storage

**Decision**  
Disable public access to Azure Storage endpoints.

**Rationale**
- Prevents data exfiltration
- Enforces access through identity and networking controls
- Aligns with Zero Trust principles

---

## 8. Centralized RBAC at Management Group Level

**Decision**  
Apply RBAC roles at **management group scope**.

**Rationale**
- Consistent permissions across subscriptions
- Reduced administrative overhead
- Clear separation of duties

---

## 9. ExpressRoute for Hybrid Connectivity

**Decision**  
Maintain ExpressRoute connectivity between on-premises and Azure.

**Rationale**
- Predictable latency
- Traffic routed over Microsoft backbone
- Required for enterprise-grade hybrid scenarios

---

## Summary

The final architecture prioritizes:
- Security and compliance over raw cost
- Platform-managed services over self-managed infrastructure
- Scalability and fault tolerance by design

These decisions align with Azure Well-Architected Framework best practices and reflect real-world enterprise constraints.
