# SQL Server Migration to Azure (PaaS) – Azure SQL Managed Instance (HA & DR)

## Overview

This case study describes the migration of a **mission-critical on-premises SQL Server workload**
to **Azure SQL Managed Instance (PaaS)**, with a focus on:

- High availability within a region
- Disaster recovery across regions
- Minimal operational overhead
- High-performance OLTP workloads

The target architecture leverages **Azure SQL Managed Instance – Business Critical tier**
to provide built-in HA and simplified DR while preserving SQL Server compatibility.

---

## Business Context

**Company:** Contoso Digital Services  
**Industry:** Financial Services  
**Workload:** Core OLTP platform  
**Criticality:** Tier-1 (24×7 operations)

The existing SQL Server environment heavily relies on:
- Stored procedures
- CLR integration
- SQL Agent jobs
- Cross-database queries

The business wants to modernize infrastructure while:
- Reducing operational complexity
- Eliminating VM and OS management
- Preserving SQL Server feature compatibility

---

## Key Requirements

### Functional Requirements
- Support for stored procedures and SQL Agent jobs
- Support for CLR assemblies
- Minimal application changes

### Availability Requirements
- High availability within the primary region
- Automatic failover with no manual intervention
- Read scale-out capability

### Disaster Recovery Requirements
- Disaster recovery in a secondary Azure region
- Low RPO (near-zero)
- Low RTO (minutes)

### Performance Requirements
- Optimized for high-throughput OLTP workloads
- Low latency
- SSD-backed storage

---

## Target Architecture Summary

### Primary Region
- Azure SQL Managed Instance
- **Business Critical service tier**
- Built-in high availability with multiple replicas
- Read-only secondary replicas for scale-out reads

### Secondary Region
- Azure SQL Managed Instance
- Auto-failover group configured
- Asynchronous replication
- Automatic failover capability

---

## High Availability & Disaster Recovery Model

| Capability | Implementation |
|----------|---------------|
| Intra-region HA | Built-in replicas (Business Critical tier) |
| Automatic failover | Yes |
| Read replicas | Yes |
| Cross-region DR | Auto-failover group |
| OS / VM management | Not required |

---
