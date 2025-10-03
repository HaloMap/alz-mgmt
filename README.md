# HaloMap Azure Landing Zone

Enterprise-grade Azure Landing Zone infrastructure for HaloMap B2B SaaS platform.

## 🏗️ Architecture Overview

This repository contains the Infrastructure as Code (Terraform) for HaloMap's Azure Landing Zone, providing:

- **Governance**: Management groups, Azure Policies, RBAC
- **Security**: Key Vaults, managed identities, RBAC
- **Networking**: Hub VNet architecture with Private DNS Zones
- **Monitoring**: Log Analytics, Application Insights
- **Cost Optimization**: Pay-per-use serverless architecture

## 📊 Current Infrastructure

### Landing Zone Foundation
- ✅ 11 Management Groups (governance structure)
- ✅ Azure Policy assignments (security & compliance)
- ✅ Hub Virtual Network with Private DNS Zones (~100 zones)
- ✅ Log Analytics Workspace (centralized logging)
- ✅ GitHub Actions CI/CD with OIDC authentication

### Application Resources (Dev + Prod)

#### Shared Resources (`rg-halomap-shared-aue`)
- Container Registry: `acrhalomapshared.azurecr.io`
- Shared Storage Account: `sthalomapshared`

#### Development Environment (`rg-halomap-dev-aue`)
- Key Vault: `kv-halomap-dev-aue`
- Cosmos DB (Serverless): `cosmos-halomap-dev-aue`
- Azure Functions: `func-halomap-dev-aue`
- Data Factory: `adf-halomap-dev-aue`
- Storage Account: `sthalomapdev`
- Application Insights: `func-halomap-dev-aue`

#### Production Environment (`rg-halomap-prod-aue`)
- Key Vault: `kv-halomap-prod-aue`
- Cosmos DB (Serverless): `cosmos-halomap-prod-aue`
- Azure Functions: `func-halomap-prod-aue`
- Data Factory: `adf-halomap-prod-aue`
- Storage Account: `sthalomapprod`
- Application Insights: `func-halomap-prod-aue`

## 💰 Cost Structure

**Monthly Base Cost: ~$60-100**
- Container Registry (Basic): $5/month
- Storage (minimal data): $2-5/month
- Key Vaults: <$1/month
- Private DNS Zones: ~$40/month
- Log Analytics: ~$10-50/month

**Pay-Per-Use (Serverless):**
- Cosmos DB: $0 when empty, pay per RU when used
- Functions: $0 when idle, pay per execution
- Data Factory: $0 when idle, pay per pipeline run

## 🚀 Quick Start

### Prerequisites
- Azure CLI installed
- Azure subscription with Owner permissions
- GitHub account

### Deploy Infrastructure Changes

Changes are deployed automatically via GitHub Actions:

1. Create a branch
2. Make changes to Terraform files
3. Create a Pull Request
4. CI workflow runs `terraform plan`
5. Review plan in PR comments
6. Merge PR
7. CD workflow deploys to Azure

### Manual Terraform Commands (Local Testing)

```bash
# Initialize Terraform
terraform init -backend-config=backend.tfbackend

# Plan changes
terraform plan

# Note: Apply via GitHub Actions (not locally)
```

## 📁 Repository Structure

```
alz-mgmt/
├── .github/
│   └── workflows/
│       ├── ci.yaml              # Pull request validation
│       └── cd.yaml              # Deployment to Azure
├── lib/                         # Custom policy definitions
├── modules/
│   ├── config-templating/
│   ├── management_groups/
│   └── management_resources/
├── platform-landing-zone.auto.tfvars  # Main configuration
├── terraform.tf                 # Provider & backend config
├── variables.*.tf               # Variable definitions
├── main.*.tf                    # Resource definitions
└── outputs.tf                   # Output values
```

## 🔐 Secrets Management

All secrets are stored in Azure Key Vault:

### Development Secrets (`kv-halomap-dev-aue`)
```bash
# Give yourself permission
az role assignment create \
  --role "Key Vault Secrets Officer" \
  --assignee your-email@domain.com \
  --scope /subscriptions/5d009971-18ba-4d7a-b68e-4a9b9ca8dc8f/resourceGroups/rg-halomap-dev-aue/providers/Microsoft.KeyVault/vaults/kv-halomap-dev-aue

# Store a secret
az keyvault secret set \
  --vault-name kv-halomap-dev-aue \
  --name MyApiKey \
  --value "secret-value"
```

## 🔄 CI/CD Pipeline

### CI Workflow (Pull Requests)
Triggered on: Pull requests to `main`

Steps:
1. Checkout code
2. Install Terraform
3. Initialize Terraform
4. Validate configuration
5. Run `terraform plan`
6. Post plan to PR comments

### CD Workflow (Deployments)
Triggered on: Push to `main`

Steps:
1. Checkout code
2. Install Terraform
3. Initialize Terraform
4. Run `terraform plan`
5. **Wait for manual approval**
6. Run `terraform apply`

## 📚 Documentation

- [Foundation Setup Guide](docs/FOUNDATION_SETUP.md)
- [Architecture Decisions](docs/ARCHITECTURE.md)
- [Cost Optimization](docs/COST_OPTIMIZATION.md)
- [Development Workflow](docs/DEVELOPMENT_WORKFLOW.md)

## 🎯 Key Features

### Cost Optimized
- Removed expensive enterprise networking resources
- Using serverless/consumption-based services
- Pay-per-use model for all compute and data services

### Secure by Default
- RBAC enabled on all resources
- Secrets stored in Key Vault
- Managed identities for authentication
- Private DNS zones for secure connectivity

### Developer Friendly
- Separate dev/prod environments
- Local development supported
- Fast deployment via CI/CD
- Infrastructure as Code

### Production Ready
- Enterprise governance structure
- Comprehensive monitoring
- Automated deployments
- Multi-tenant architecture ready

## 🛠️ Common Tasks

### Deploy a Function App
```bash
# In your function app directory
func azure functionapp publish func-halomap-dev-aue
```

### Push Docker Image to ACR
```bash
az acr login --name acrhalomapshared
docker build -t acrhalomapshared.azurecr.io/myapp:v1 .
docker push acrhalomapshared.azurecr.io/myapp:v1
```

### Create Cosmos DB Database
```bash
az cosmosdb sql database create \
  --account-name cosmos-halomap-dev-aue \
  --resource-group rg-halomap-dev-aue \
  --name myapp-db
```

## 🆘 Troubleshooting

### Terraform State Issues
State is stored in Azure Storage Account:
- Storage Account: `stoalzmgmaus001abha`
- Container: `mgmt-tfstate`
- Resource Group: `rg-alz-mgmt-state-australiaeast-001`

### GitHub Actions Authentication
Uses OIDC with managed identities:
- Plan Identity: `id-alz-mgmt-australiaeast-plan-001`
- Apply Identity: `id-alz-mgmt-australiaeast-apply-001`

## 📞 Support

For issues or questions:
1. Check the [docs](docs/) directory
2. Review GitHub Actions workflow logs
3. Check Azure Portal for resource status

## 📄 License

Private repository - HaloMap proprietary infrastructure

---

**Last Updated**: October 2025  
**Maintained By**: Steven Campbell  
**Azure Subscription**: Azure Sponsorship (5d009971-18ba-4d7a-b68e-4a9b9ca8dc8f)
