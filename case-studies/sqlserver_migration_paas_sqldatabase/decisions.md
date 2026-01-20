# Architectural Decisions & Trade-offs

This document outlines the key architectural decisions taken to design
a highly available and resilient **Azure SQL Managed Instance (PaaS)** solution.

---

## 1. Use of Azure SQL Managed Instance (MI)

**Decision**  
Use **Azure SQL Managed Instance** as the database platform.

**Rationale**
- High compatibility with SQL Server
- Supports CLR, SQL Agent, stored procedures
- Eliminates OS and VM management
- Fully managed patching and backups

**Outcome**
- Minimal refactoring effort
- Reduced operational burden
- Faster migration compared to Azure SQL Database (single DB)

---

## 2. Business Critical Service Tier

**Decision**  
Deploy Managed Instance using the **Business Critical** tier.

**Rationale**
- SSD-backed local storage
- Optimized for OLTP workloads
- Built-in read-only replicas
- Lower latency compared to General Purpose

**Benefits**
- High transaction throughput
- Read scale-out support
- Better performance isolation

---

## 3. High Availability Model

**Decision**  
Rely on **built-in HA provided by Azure SQL Managed Instance**.

**Rationale**
- Business Critical tier uses multiple replicas
- Automatic failover handled by the platform
- No clustering or load balancer configuration required

**Key Advantage**
- HA is transparent to the application
- No manual failover logic

---

## 4. Disaster Recovery via Auto-Failover Groups

**Decision**  
Configure **Auto-Failover Groups** between two Azure regions.

**Rationale**
- Native, managed DR mechanism
- Asynchronous replication to secondary region
- Automatic or manual failover supported

**Outcome**
- Region-level disaster recovery
- Predictable RTO and low RPO
- No VM replication or storage recovery needed

---

## 5. Why SQL Server IaaS Was Rejected

**Rejected Option**  
SQL Server on Azure Virtual Machines (IaaS)

**Reasons for Rejection**
- Requires OS and patch management
- HA and DR must be designed and maintained manually
- Higher operational complexity
- Requires SQL clustering or Availability Groups

---

## 6. Why Failover Cluster Instance (FCI) Was Rejected

**Rejected Option**  
SQL Server Always On Failover Cluster Instance

**Reasons for Rejection**
- Requires shared storage
- No native cross-region failover
- Complex DR procedures
- Not cloud-optimized

---

## 7. Why Always On Availability Groups (AG) Were Rejected

**Rejected Option**  
SQL Server Always On Availability Groups (IaaS)

**Reasons for Rejection**
- Requires SQL Server Enterprise licensing
- Requires manual configuration and monitoring
- Still depends on VM-level management
- Higher cost and operational effort compared to PaaS

---

## Final Assessment

Azure SQL Managed Instance with Business Critical tier provides:

- Built-in HA
- Managed DR
- High OLTP performance
- Reduced operational risk
- Cloud-native resilience

In exchange for:
- Less granular infrastructure control
- Platform-imposed limits compared to full IaaS

This design represents a **production-ready, enterprise-grade PaaS SQL architecture**
suitable for mission-critical workloads with strict availability and performance requirements.
