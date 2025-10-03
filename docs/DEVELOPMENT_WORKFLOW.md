# Development Workflow

## Getting Started

### Local Development Setup

1. **Install Prerequisites**
```bash
# Azure CLI
brew install azure-cli

# Terraform
brew install terraform

# Azure Functions Core Tools
brew install azure-functions-core-tools@4

# Node.js (if using Functions)
brew install node@20
```

2. **Authenticate with Azure**
```bash
az login
az account set --subscription 5d009971-18ba-4d7a-b68e-4a9b9ca8dc8f
```

3. **Clone Repository**
```bash
git clone https://github.com/HaloMap/alz-mgmt.git
cd alz-mgmt
```

## Working with Infrastructure

### Making Infrastructure Changes

1. **Create a feature branch**
```bash
git checkout -b feature/add-new-resource
```

2. **Make changes to Terraform files**
```bash
# Edit platform-landing-zone.auto.tfvars or other .tf files
code platform-landing-zone.auto.tfvars
```

3. **Test locally (optional)**
```bash
terraform init -backend-config=backend.tfbackend
terraform plan
```

4. **Commit and push**
```bash
git add .
git commit -m "Add new resource configuration"
git push origin feature/add-new-resource
```

5. **Create Pull Request**
```bash
gh pr create --title "Add new resource" --body "Description of changes"
```

6. **Review Terraform Plan**
- CI workflow will automatically run
- Check PR comments for terraform plan output
- Review what will be created/changed/destroyed

7. **Merge PR**
```bash
gh pr merge --squash
```

8. **Monitor Deployment**
```bash
gh run watch
```

## Working with Applications

### Deploying Azure Functions

#### Local Development
```bash
# Create a new function project
func init my-function-app --worker-runtime node --language typescript
cd my-function-app

# Create a new function
func new --name HttpTrigger --template "HTTP trigger"

# Run locally
func start
```

#### Get Secrets from Key Vault
```bash
# Give yourself access
az role assignment create \
  --role "Key Vault Secrets Officer" \
  --assignee admin@stevenhalomap.onmicrosoft.com \
  --scope /subscriptions/5d009971-18ba-4d7a-b68e-4a9b9ca8dc8f/resourceGroups/rg-halomap-dev-aue/providers/Microsoft.KeyVault/vaults/kv-halomap-dev-aue

# Get Cosmos DB connection string
az keyvault secret show \
  --vault-name kv-halomap-dev-aue \
  --name CosmosDB-ConnectionString \
  --query value -o tsv
```

#### Deploy to Azure
```bash
# Deploy to dev
func azure functionapp publish func-halomap-dev-aue

# Deploy to prod
func azure functionapp publish func-halomap-prod-aue
```

### Working with Cosmos DB

#### Create Database and Container
```bash
# Create database
az cosmosdb sql database create \
  --account-name cosmos-halomap-dev-aue \
  --resource-group rg-halomap-dev-aue \
  --name halomap-db

# Create container with partition key
az cosmosdb sql container create \
  --account-name cosmos-halomap-dev-aue \
  --resource-group rg-halomap-dev-aue \
  --database-name halomap-db \
  --name users \
  --partition-key-path "/tenantId" \
  --throughput 400
```

#### Query Data
```bash
# Install Cosmos DB extension
az extension add --name cosmosdb-preview

# Query documents
az cosmosdb sql container query \
  --account-name cosmos-halomap-dev-aue \
  --resource-group rg-halomap-dev-aue \
  --database-name halomap-db \
  --container-name users \
  --query-text "SELECT * FROM c WHERE c.tenantId = 'tenant-a'"
```

### Working with Container Registry

#### Build and Push Images
```bash
# Login to ACR
az acr login --name acrhalomapshared

# Build image
docker build -t acrhalomapshared.azurecr.io/myapp:v1.0.0 .

# Push image
docker push acrhalomapshared.azurecr.io/myapp:v1.0.0

# List images
az acr repository list --name acrhalomapshared --output table
```

### Working with Storage

#### Upload Files
```bash
# Get storage account key
STORAGE_KEY=$(az storage account keys list \
  --account-name sthalomapdev \
  --resource-group rg-halomap-dev-aue \
  --query "[0].value" -o tsv)

# Create container
az storage container create \
  --name mydata \
  --account-name sthalomapdev \
  --account-key $STORAGE_KEY

# Upload file
az storage blob upload \
  --container-name mydata \
  --name myfile.txt \
  --file ./local-file.txt \
  --account-name sthalomapdev \
  --account-key $STORAGE_KEY
```

## Development Environments

### Dev Environment
- **Purpose**: Active development and testing
- **Data**: Test/dummy data only
- **Access**: All developers have Contributor access
- **Cost**: Minimal, serverless resources idle

### Prod Environment
- **Purpose**: Customer-facing production
- **Data**: Real customer data
- **Access**: Limited to ops team
- **Cost**: Pay-per-use based on actual usage

## Best Practices

### Infrastructure as Code
- ✅ Always make changes via Terraform
- ✅ Never make manual changes in Azure Portal
- ✅ Test in dev before deploying to prod
- ✅ Use descriptive commit messages
- ✅ Review terraform plans carefully

### Secrets Management
- ✅ Store all secrets in Key Vault
- ✅ Use managed identities for authentication
- ✅ Never commit secrets to Git
- ✅ Rotate secrets regularly

### Cost Management
- ✅ Delete unused resources
- ✅ Use serverless/consumption plans
- ✅ Monitor spending with Azure Cost Management
- ✅ Set up billing alerts

### Testing
- ✅ Test functions locally before deploying
- ✅ Use dev environment for integration testing
- ✅ Validate changes in dev before promoting to prod

## Troubleshooting

### Function App Not Starting
```bash
# Check logs
func azure functionapp logstream func-halomap-dev-aue

# Check app settings
az functionapp config appsettings list \
  --name func-halomap-dev-aue \
  --resource-group rg-halomap-dev-aue
```

### Cosmos DB Connection Issues
```bash
# Test connectivity
az cosmosdb show \
  --name cosmos-halomap-dev-aue \
  --resource-group rg-halomap-dev-aue

# Get connection string
az cosmosdb keys list \
  --name cosmos-halomap-dev-aue \
  --resource-group rg-halomap-dev-aue \
  --type connection-strings
```

### Terraform State Issues
```bash
# If state is locked
az storage blob lease break \
  --container-name mgmt-tfstate \
  --blob-name terraform.tfstate \
  --account-name stoalzmgmaus001abha

# View state
terraform show
```

## Useful Commands

### Quick Access to Resources
```bash
# Open Azure Portal to resource group
open "https://portal.azure.com/#@stevenhalomap.onmicrosoft.com/resource/subscriptions/5d009971-18ba-4d7a-b68e-4a9b9ca8dc8f/resourceGroups/rg-halomap-dev-aue"

# Open Function App
open "https://func-halomap-dev-aue.azurewebsites.net"

# Open Cosmos DB Data Explorer
open "https://portal.azure.com/#@stevenhalomap.onmicrosoft.com/resource/subscriptions/5d009971-18ba-4d7a-b68e-4a9b9ca8dc8f/resourceGroups/rg-halomap-dev-aue/providers/Microsoft.DocumentDB/databaseAccounts/cosmos-halomap-dev-aue/dataExplorer"
```

### Monitor Costs
```bash
# View current month costs
az consumption usage list \
  --start-date $(date -u -v-30d +%Y-%m-%d) \
  --end-date $(date -u +%Y-%m-%d) \
  --query "[].{Service:instanceName, Cost:pretaxCost}" \
  --output table
```

---

**Last Updated**: October 2025
