# Cloud Architecture Extension Roadmap

This document extends the original AdventureWorks data-engineering project toward an Azure Cloud Architect case study.

## Target architecture

```text
GitHub/API
   |
Azure Data Factory
   |
ADLS Gen2: landing -> bronze -> silver -> gold
   |
Azure Databricks / Delta Lake
   |
Azure SQL or Synapse serverless SQL
   |
Power BI
```

Cross-cutting services:

- Microsoft Entra ID and managed identities
- Azure Key Vault
- Azure Monitor and Log Analytics
- Private Endpoints and Private DNS
- Azure Policy and resource locks
- GitHub Actions for CI/CD

## Implementation phases

### Phase 1: Baseline data platform

1. Create a resource group for the learning environment.
2. Create an ADLS Gen2 storage account with hierarchical namespace enabled.
3. Create `source`, `bronze`, `silver`, and `gold` containers.
4. Upload the AdventureWorks CSV files to `source`.
5. Build an ADF Copy Activity from `source` to `bronze`.
6. Run the Databricks notebooks for Silver and Gold transformations.
7. Connect Power BI to the Gold output.

### Phase 2: Production data-engineering practices

1. Parameterize the ADF pipeline by file name and processing date.
2. Add incremental loading instead of copying every file on every run.
3. Store Delta tables instead of only CSV outputs.
4. Add schema validation and reject invalid records to a quarantine path.
5. Add row-count, null-rate, duplicate and referential-integrity checks.
6. Add a pipeline run ID and ingestion timestamp to every layer.
7. Configure retry, timeout and failure notification policies.

### Phase 3: Security and governance

1. Use a managed identity instead of storage keys where possible.
2. Store any remaining secrets in Azure Key Vault.
3. Assign least-privilege RBAC roles for ADF, Databricks and analysts.
4. Enable diagnostic logs for Storage, ADF and Databricks.
5. Add Microsoft Purview or a documented data catalogue.
6. Define data classification and retention rules.
7. Document who may read Bronze, Silver and Gold data.

### Phase 4: Network architecture

1. Design a hub-and-spoke virtual network.
2. Place private endpoints for Storage, Key Vault and SQL/Synapse.
3. Configure Private DNS zones.
4. Restrict public network access after private connectivity is verified.
5. Document the on-premises connectivity option using VPN or ExpressRoute.

### Phase 5: Infrastructure as Code and CI/CD

1. Add Terraform modules for resource group, storage, Key Vault and ADF.
2. Separate `dev`, `test` and `prod` variables.
3. Run `terraform fmt` and `terraform validate` in GitHub Actions.
4. Require a manual approval before production deployment.
5. Keep secrets in GitHub/Azure secret stores, never in the repository.
6. Export and version ADF pipeline definitions.

### Phase 6: Architecture documentation

Document the following decisions:

- Why ADF orchestrates while Databricks transforms;
- Why ADLS is used as the data lake;
- Why Delta Lake is used for reliable table storage;
- Why managed identity is preferred over access keys;
- How the solution would migrate from on-premises SQL Server;
- How cost is controlled by job clusters, schedules and auto-termination;
- How the design changes for multiple customers or business units.

## Acceptance criteria

- A new CSV can be ingested without editing the pipeline.
- A failed file is isolated and does not corrupt Silver or Gold.
- A rerun does not create duplicate records.
- Every dataset has an owner, classification and retention policy.
- No storage account keys are committed to Git.
- A developer can deploy the infrastructure from a clean environment.
- Power BI reads only business-ready Gold data.

## Important safety rule

Do not run `terraform apply` until the subscription, region, budget and estimated monthly cost have been reviewed. Use `terraform plan` first and destroy temporary resources after practice.
