# HaloMap Architecture Documentation

## Overview

HaloMap uses a modern, cost-optimized Azure Landing Zone architecture designed for B2B SaaS applications.

## Design Principles

1. **Serverless-First**: Use consumption-based services to minimize costs
2. **Environment Isolation**: Separate dev and prod for security
3. **Multi-Tenant Ready**: Architecture supports multiple B2B customers
4. **Infrastructure as Code**: Everything managed via Terraform
5. **GitOps**: All changes via Git and CI/CD

## Architecture Layers

### 1. Governance Layer (Azure Landing Zone)

```
Tenant Root
└── alz (Azure Landing Zones)
    ├── platform (infrastructure)
    │   ├── connectivity (hub network, DNS)
    │   ├── management (monitoring, logging)
    │   ├── identity
    │   └── security
    └── landingzones (applications)
        ├── corp (internal apps)
        ├── online (external apps)
        └── sandbox (experiments)
```

**Key Components:**
- Management Groups for organizational structure
- Azure Policies for compliance & security
- RBAC roles and assignments
- Subscription organization

### 2. Networking Layer

```
Hub VNet (10.0.0.0/16) - DISABLED (cost optimization)
├── Private DNS Zones (~100 zones for Azure services)
└── Route tables

Spoke VNets - NOT YET DEPLOYED
├── Dev Spoke (10.1.0.0/16) - Future
└── Prod Spoke (10.3.0.0/16) - Future
```

**Current State:**
- Hub VNet exists but minimal resources (cost optimization)
- Private DNS zones ready for private endpoints
- No expensive resources (firewall, gateways removed)

### 3. Application Layer

#### Shared Resources
```
rg-halomap-shared-aue
├── Container Registry (ACR)
└── Shared Storage Account
```

#### Per Environment (Dev/Prod)
```
rg-halomap-{env}-aue
├── Key Vault (secrets)
├── Cosmos DB (serverless database)
├── Azure Functions (serverless compute)
├── Data Factory (ETL pipelines)
├── Storage Account (application data)
└── Application Insights (monitoring)
```

## Technology Stack

### Compute
- **Azure Functions (Consumption Plan)**
  - Pay per execution
  - Auto-scaling
  - Node.js 20 runtime

### Data
- **Cosmos DB (Serverless)**
  - NoSQL document database
  - Multi-tenant via partition keys
  - Pay per RU consumed

### Storage
- **Azure Storage Accounts**
  - Blob storage for files
  - Queue storage for messaging
  - Table storage for logs

### Integration
- **Azure Data Factory**
  - ETL pipelines
  - Data movement
  - Pay per pipeline run

### Security
- **Azure Key Vault**
  - Secrets management
  - Certificate storage
  - RBAC authorization

### Monitoring
- **Application Insights**
  - Application telemetry
  - Performance monitoring
  - Custom metrics

## Multi-Tenancy Strategy

### Tenant Isolation
```
Cosmos DB Strategy:
└── Database: halomap-db
    └── Container: users
        ├── Partition Key: /tenantId
        ├── Tenant A documents (tenantId = "tenant-a")
        ├── Tenant B documents (tenantId = "tenant-b")
        └── Tenant C documents (tenantId = "tenant-c")
```

**Benefits:**
- Logical isolation via partition keys
- Cost-effective (shared infrastructure)
- Easy to scale
- Query isolation enforced by partition key

## Security Architecture

### Authentication Flow
```
User → Azure AD → Function App → Managed Identity → Cosmos DB
                                                  → Key Vault
                                                  → Storage
```

### Secrets Management
- Application secrets in Key Vault
- Managed identities for service authentication
- No credentials in code
- RBAC for access control

## CI/CD Architecture

### Pipeline Flow
```
Developer
    ↓ (git push)
GitHub Repository (alz-mgmt)
    ↓ (PR trigger)
CI Workflow
    ├── Terraform Validate
    ├── Terraform Plan
    └── Post to PR Comments
    ↓ (merge to main)
CD Workflow
    ├── Terraform Plan
    ├── Manual Approval
    └── Terraform Apply
    ↓
Azure Resources Updated
```

### Authentication
- GitHub Actions → Azure via OIDC
- Managed Identity for Terraform
- No secrets in GitHub

## Cost Optimization Decisions

### What We Removed
- ❌ DDoS Protection ($3,000/month)
- ❌ Azure Firewall ($1,250/month)
- ❌ VPN Gateway ($140/month)
- ❌ ExpressRoute Gateway ($730/month)
- ❌ Azure Bastion ($140/month)
- ❌ Private DNS Resolver ($80/month)

**Savings: ~$5,300/month**

### What We Kept
- ✅ Management Groups (FREE)
- ✅ Azure Policies (FREE)
- ✅ Hub VNet (FREE)
- ✅ Private DNS Zones (~$40/month)
- ✅ Log Analytics (pay per GB)

### Pay-Per-Use Services
- Cosmos DB Serverless
- Functions Consumption Plan
- Data Factory (per pipeline run)
- Storage (per GB stored)

## Scaling Strategy

### Horizontal Scaling
- Functions auto-scale with load
- Cosmos DB scales with RU consumption
- Storage scales with data volume

### Vertical Scaling Options
- Upgrade Cosmos DB to provisioned throughput
- Move Functions to Premium plan (VNet integration)
- Add App Service Plans for web apps

## Disaster Recovery

### Current State
- All resources in Australia East
- No geo-replication configured
- Point-in-time restore available

### Future Enhancements
- Multi-region Cosmos DB
- Geo-redundant storage
- Azure Traffic Manager

## Monitoring Strategy

### Metrics Collected
- Application Insights for app telemetry
- Azure Monitor for infrastructure
- Log Analytics for centralized logs
- Cosmos DB metrics

### Alerting (To Be Configured)
- Function execution failures
- Cosmos DB throttling
- Storage account errors
- Cost anomalies

## Compliance & Governance

### Azure Policies Applied
- Require tags on resources
- Allowed locations (Australia East)
- SKU restrictions
- Network security rules

### RBAC Model
- Environment-based access control
- Least privilege principle
- Managed identities where possible

## Future Enhancements

1. **Networking**
   - Add spoke VNets when needed
   - Implement private endpoints
   - Re-enable firewall for production

2. **Security**
   - Azure Front Door for WAF
   - Azure DDoS Protection (production only)
   - Private endpoints for all services

3. **Monitoring**
   - Custom dashboards
   - Alert rules
   - Cost anomaly detection

4. **Scaling**
   - CDN for static content
   - API Management
   - Azure Redis Cache

---

**Last Updated**: October 2025
**Version**: 1.0
