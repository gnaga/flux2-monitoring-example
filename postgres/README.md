# PostgreSQL Hub and Spoke Architecture

This directory contains the PostgreSQL deployment configuration using a hub and spoke pattern with Flux GitOps.

## Architecture Overview

```
                    HUB CLUSTER (monitoring)
    ┌─────────────────────────────────────────────────────────┐
    │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────┐ │
    │  │   PostgreSQL    │  │   Monitoring    │  │  Flux   │ │
    │  │   Controller    │  │   Stack         │  │ System  │ │
    │  └─────────────────┘  └─────────────────┘  └─────────┘ │
    └─────────────────────────────────────────────────────────┘
                              │
                              │ Manages via GitOps
                              ▼
                    SPOKE CLUSTERS
    ┌─────────────────────────────────────────────────────────┐
    │  ┌─────────────────┐              ┌─────────────────┐  │
    │  │   PRODUCTION    │              │     STAGING     │  │
    │  │   ┌───────────┐ │              │  ┌───────────┐  │  │
    │  │   │ PostgreSQL│ │              │  │ PostgreSQL│  │  │
    │  │   │ Instance  │ │              │  │ Instance  │  │  │
    │  │   └───────────┘ │              │  └───────────┘  │  │
    │  └─────────────────┘              └─────────────────┘  │
    └─────────────────────────────────────────────────────────┘

Deployment Flow:
1. Hub deploys PostgreSQL controller
2. Hub manages spoke clusters via Flux
3. Each spoke gets its own PostgreSQL instance
4. All instances are monitored centrally
```

## Configuration

### Hub Cluster (monitoring)
- **Namespace**: `postgres-monitoring`
- **Database**: `monitoring_db`
- **Storage**: 5Gi
- **Resources**: 500m CPU, 512Mi Memory

### Production Cluster
- **Namespace**: `postgres-production`
- **Database**: `production_db`
- **Storage**: 20Gi
- **Resources**: 1000m CPU, 1Gi Memory

### Staging Cluster
- **Namespace**: `postgres-staging`
- **Database**: `staging_db`
- **Storage**: 10Gi
- **Resources**: 500m CPU, 512Mi Memory

## Deployment Order

1. **Monitoring Controllers** - Deploys Prometheus, Grafana, etc.
2. **PostgreSQL Controller** - Deploys PostgreSQL in monitoring cluster
3. **Production Cluster** - Deploys PostgreSQL in production cluster
4. **Staging Cluster** - Deploys PostgreSQL in staging cluster

## Security Notes

⚠️ **WARNING**: The passwords in this configuration are for demonstration purposes only. In production, use proper secret management solutions like:

- External Secrets Operator
- Sealed Secrets
- HashiCorp Vault
- Azure Key Vault
- AWS Secrets Manager

## Monitoring

Each PostgreSQL instance includes:
- Health checks (liveness and readiness probes)
- Resource limits and requests
- Persistent storage
- Service discovery
- Proper labeling for monitoring

## Scaling

To scale PostgreSQL instances:
1. Update the `replicas` field in the deployment
2. Consider using StatefulSets for production workloads
3. Implement proper backup and recovery procedures
4. Use connection pooling for high availability
