# Azure Architecture Solutions

A curated set of Azure architecture case studies focused on migrations and technical designs. Each case study is self-contained and documents the problem context, target architecture, and key design choices.

This repository focuses on architecture artifacts (diagrams and design documentation), not on full application source code.

## Repository layout

- `case-studies/`
  - One folder per scenario.
  - Typical contents:
    - `README.md` (or `README_arch.md`): problem statement and target architecture summary
    - `decisions.md`: concrete design decisions, ADRs and trade-offs for the scenario
    - `*.jpg`: architecture diagram
    - `*.drawio`: editable diagram source

## How to navigate

1. Open a case study folder from the index below.
2. Read the case overview (`README*.md`) to understand requirements and constraints.
3. Review the diagram (`.jpg`) and open the editable source (`.drawio`) when needed.
4. Read `decisions.md` for the rationale behind the main architectural choices.

## Case studies index

| Case study | Scenario | Primary services / patterns | Focus areas |
|---|---|---|---|
| [Event-driven order processing (AKS + WebSockets + Redis backplane)](./case-studies/eventdriven-k8s-websockets-redisbackplane-orderprocessing/README_arch.md) | Event-driven microservices with real-time UX | AKS, messaging, SignalR/WebSockets, Redis, SQL as system of record | Asynchronous workflows, real-time notifications, scalability |
| [Hybrid migration – App1](./case-studies/hybrid-migration-app1/) | Legacy multi-tier app migrated from on-prem to Azure | VM Scale Sets, Azure SQL Database, Storage, Managed Identity | Hybrid constraints, security/compliance, HA and scalability |
| [Multi-region App Service – App1](./case-studies/multiregion-appservice-app1/) | Multi-region web app fronted by global DNS routing | Traffic Manager, Application Gateway (WAF), App tier per region | Resilience to regional outages, latency, L7 security |
| [SQL Server on IaaS – Always On AG (HA & DR)](./case-studies/sqlserver_migration_iaas_alwaysonag_ha_DR/) | SQL Server migration using Availability Groups | SQL Server on VMs, Availability Zones, cross-region replication | Intra-region HA, cross-region DR, operational runbooks |
| [SQL Server on IaaS – Lift & shift (HA & DR)](./case-studies/sqlserver_migration_iaas_liftandshift_ha_DR/) | Minimal-change SQL Server migration to IaaS | SQL Server FCI on VMs, Azure Files, Load Balancer, ASR/Backup | Compatibility-first, HA/DR with IaaS primitives |
| [SQL Server to PaaS – Azure SQL Managed Instance (HA & DR)](./case-studies/sqlserver_migration_paas_managedinstance/) | SQL Server modernization to managed PaaS | Azure SQL Managed Instance (Business Critical), failover groups | Reduced ops overhead, HA by platform, DR design |
| [SQL Server to PaaS – Azure SQL Database](./case-studies/sqlserver_migration_paas_sqldatabase/) | Database modernization to fully managed database | Azure SQL Database (Business Critical), failover groups | Performance, HA/DR by design, simplified operations |

## Design principles used across case studies

The designs are generally aligned with common enterprise guidance, including:
- Reliability: fault domains, zone-aware deployments, and regional recovery planning
- Security: least privilege, controlled ingress/egress, and secure identity patterns
- Operational excellence: clear ownership boundaries, automation-friendly components, and observability
- Performance and cost awareness: right-sizing, tier selection, and scaling strategies

## Notes

- Diagrams are provided as both a rendered image (`.jpg`) and an editable source (`.drawio`).
- Service names and SKUs are representative; adapt to your subscription constraints, regions, and policy requirements.
