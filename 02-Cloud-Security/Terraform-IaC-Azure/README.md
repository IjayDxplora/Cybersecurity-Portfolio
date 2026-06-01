# 🏗️ Terraform IaC — Azure Secure Resource Provisioning

**Category:** Infrastructure as Code | Cloud Security
**Environment:** Microsoft Azure
**Status:** 🔄 In Progress

---

## 🎯 Objective

Build reusable, security-hardened Terraform templates to provision and govern Azure resources consistently across dev, staging, and production environments — eliminating manual configuration drift and enforcing security standards at deployment time.

---

## 📋 Templates Planned

| Template | Description | Status |
|----------|-------------|--------|
| `azure-resource-group/` | Secure resource group with RBAC, tags, and locks | 🔄 Coming soon |
| `azure-vnet-nsg/` | Virtual network with hardened NSG rules | 🔄 Coming soon |
| `azure-keyvault/` | Key Vault with RBAC, soft delete, purge protection | 🔄 Coming soon |
| `azure-storage-secure/` | Storage account with encryption, HTTPS-only, no public access | 🔄 Coming soon |
| `entra-id-app-registration/` | App registration with minimal permissions | 🔄 Coming soon |

---

## 🔒 Security Principles Applied
- Least privilege RBAC on all resources
- No public access by default — explicit opt-in required
- Encryption at rest and in transit enforced as default
- Resource locks on production environments
- Tags enforced for cost tracking and ownership

---

*This folder will be populated with working Terraform configurations — check back for updates.*
