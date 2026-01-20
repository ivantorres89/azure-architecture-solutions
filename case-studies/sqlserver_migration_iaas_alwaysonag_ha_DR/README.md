# SQL Server Migration to Azure (IaaS) – Always On Availability Groups (HA & DR)

## Overview

This case study describes the migration of a mission-critical SQL Server workload
from an on-premises datacenter to **Microsoft Azure using IaaS**, with a focus on
**true high availability and disaster recovery across Azure regions**.

Unlike traditional lift-and-shift approaches, this architecture intentionally
adopts **SQL Server Always On Availability Groups (AG)** to achieve:
- Automatic failover across availability zones and regions
- Elimination of shared storage dependencies
- Low RTO and near-zero RPO

---

## Business Context

**Company:** Contoso Digital Services  
**Industry:** SaaS / Online Services  
**Workload:** Core transactional SQL Server database  
**Criticality:** Tier-1 system with global users

The business requires:
- Continuous availability
- Minimal downtime during failures
- Resilience against full regional outages

The application and database were reviewed and approved for:
- SQL Server Enterprise Edition
- Multi-node, multi-replica architectures
- Listener-based connectivity

---

## Key Requirements

### Availability
- High availability within the primary Azure region
- Zone-level fault tolerance
- Automatic failover without manual intervention

### Disaster Recovery
- Automatic failover to a secondary Azure region
- Near-zero data loss (RPO ≈ 0)
- RTO measured in minutes

### Architecture Constraints
- Infrastructure-as-a-Service (IaaS)
- SQL Server Enterprise Edition
- Application supports multi-node SQL architectures
- No dependency on shared storage

---

## Target Architecture Summary

### Primary Region (West US)
- Azure Virtual Network
- SQL Server Always On Availability Group
- Multiple SQL Server replicas distributed across **Availability Zones**
- Azure Load Balancer / AG Listener for client connectivity

### Secondary Region (East US)
- Azure Virtual Network
- SQL Server Availability Group replica (synchronous or asynchronous)
- Automatic failover configured
- Independent storage per SQL node

### High Availability & Disaster Recovery Model
- HA achieved via Availability Zones
- DR achieved via cross-region AG replication
- No reliance on Azure Site Recovery for SQL failover

---

## Architecture Diagram

*(Diagram to be added – SQL Server Always On AG with zonal HA and cross-region DR)*

---

## What This Case Study Demonstrates

- Cloud-optimized SQL Server IaaS architecture
- Clear distinction between HA and DR responsibilities
- Elimination of shared storage limitations
- Automatic failover across regions
- Architect-level trade-off analysis

For detailed design decisions and trade-offs, see **decisions.md**.
