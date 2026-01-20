# SQL Server Migration to Azure (PaaS) – Azure SQL Database

## Overview

This case study describes the migration of an on-premises SQL Server workload
to **Azure SQL Database (PaaS)** using the **Business Critical** service tier.

The primary goals of this migration are:
- Reduce operational overhead
- Achieve high availability and disaster recovery by design
- Optimize performance for mission-critical OLTP workloads
- Modernize the platform without requiring full SQL Server compatibility

---

## Business Context

**Company:** Contoso Retail Platform  
**Industry:** E-commerce  
**Workload:** Transaction processing and customer data platform  
**Criticality:** Mission-critical (24×7)

The existing SQL Server environment supports:
- Modern application architecture
- Single-database workloads
- Stateless application tier
- Strict performance and availability requirements

The organization wants to move away from VM-based SQL Server management
and adopt a **cloud-native PaaS database** with built-in resilience.

---

## Key Requirements

### Functional Requirements
- T-SQL support for standard CRUD operations
- Stored procedures (no CLR, no SQL Agent)
- Single-database architecture
- No cross-database queries

### Availability Requirements
- High availability within a single Azure region
- Automatic failover without manual intervention
- Transparent to the application

### Disaster Recovery Requirements
- Disaster recovery across Azure regions
- Low RPO (seconds)
- Low RTO (minutes)
- Managed failover mechanism

### Performance Requirements
- Optimized for high-throughput OLTP workloads
- Low latency
- SSD-backed storage
- Read-only replicas for reporting workloads

### Operational Requirements
- No OS or VM management
- Automatic patching and backups
- Minimal DBA operational effort

---

## Target Architecture Summary

### Primary Region
- Azure SQL Database
- **Business Critical service tier**
- Built-in high availability with multiple replicas
- Read-only replicas for scale-out reads

### Secondary Region
- Azure SQL Database
- Auto-failover group configured
- Asynchronous replication
- Automatic regional failover

---

## High Availability & Disaster Recovery Model

| Capability | Implementation |
|----------|---------------|
| Intra-region HA | Built-in replicas (Business Critical tier) |
| Automatic failover | Yes |
| Read-only replicas | Yes |
| Cross-region DR | Auto-failover group |
| OS / VM management | Not required |

---

## Summary

Azure SQL Database provides a **fully managed, cloud-native database platform**
that meets the application’s availability, performance, and resilience requirements
while significantly reducing operational complexity compared to IaaS-based solutions.
