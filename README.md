# CI/CD Templates

This repository contains a collection of reusable GitHub Actions workflows (`workflow_call`) tailored for platform engineering, infrastructure as code (IaC), and secure Azure deployments.

By centralizing these workflows, we enforce consistent CI/CD patterns, security standards, and governance across all Terraform projects.

---

## 🛠️ Available Workflows

### 1. `terraform-pr-gatekeeper.yml`
**Use case:** Pull Requests (Shift-Left Security)
A comprehensive gatekeeper for PRs. It runs `terraform fmt`, `validate`, performs an infrastructure security scan using **Trivy**, and injects the Terraform plan output directly as a PR comment.

```yaml
jobs:
  pr-check:
    uses: YourOrg/ci-cd-templates/.github/workflows/terraform-pr-gatekeeper.yml@main
    with:
      working-dir: "src/network"
```

### 2. `terraform-module-release.yml`
**Use case:** Terraform Module Repositories (Auto-Docs & SemVer)
Designed for reusable Terraform modules. It automatically updates the `README.md` with module inputs/outputs via `terraform-docs` and triggers a semantic release (auto-versioning like `v1.2.0`) based on Conventional Commits.

```yaml
jobs:
  release:
    uses: YourOrg/ci-cd-templates/.github/workflows/terraform-module-release.yml@main
    secrets:
      PAT_TOKEN: ${{ secrets.PAT_TOKEN }}
```

### 3. `multi-env-oidc-deploy.yml`
**Use case:** Secure Azure Deployments (Keyless & Approvals)
Deploys infrastructure using keyless OpenID Connect (OIDC) authentication with Azure Entra ID. It leverages GitHub Environments, allowing you to configure manual approval gates before applying changes to production.

```yaml
jobs:
  deploy-prod:
    uses: YourOrg/ci-cd-templates/.github/workflows/multi-env-oidc-deploy.yml@main
    with:
      environment: "production"
    secrets:
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
      AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

### 4. `firewall-policy-validator.yml`
**Use case:** Network Security & GitOps (Pre-Flight Checks)
Pulls our custom Python `pipeline-tools` to validate declarative YAML firewall and DNS rules against our IPAM (NetBox/EfficientIP) and live DNS resolvers before Terraform is even triggered. Fails fast if IPs are unregistered or DNS is unresolvable.

```yaml
jobs:
  validate-yaml:
    uses: YourOrg/ci-cd-templates/.github/workflows/firewall-policy-validator.yml@main
    with:
      rules-filepath: "firewall/azure-hub-rules.yaml"
      ipam-provider: "efficientip"
    secrets:
      IPAM_URL: ${{ secrets.IPAM_URL }}
      IPAM_USER: ${{ secrets.IPAM_USER }}
      IPAM_PASSWORD: ${{ secrets.IPAM_PASSWORD }}
      PIPELINE_TOOLS_TOKEN: ${{ secrets.PAT_TOKEN }}
```

### 5. `terraform-finops-pr-commenter.yml`
**Use case:** FinOps & Cost Transparency
Integrates Infracost to calculate the expected monthly Azure costs for the proposed infrastructure changes. Posts a clear cost breakdown directly into the Pull Request so reviewers can catch expensive misconfigurations prior to merging.

```yaml
jobs:
  finops:
    uses: YourOrg/ci-cd-templates/.github/workflows/terraform-finops-pr-commenter.yml@main
    with:
      working-dir: "src/azure-firewall"
    secrets:
      INFRACOST_API_KEY: ${{ secrets.INFRACOST_API_KEY }}
```

Markdown
### 6. `nextjs-deploy.yml`
**Use case:** Next.js Web Applications
A unified CI/CD workflow tailored for Next.js applications using React, TypeScript, and Tailwind CSS. It features Next.js build caching, strict type checking, and can dynamically deploy the application to either a Linux VM (via SSH/SCP) or an Azure Web App (via OIDC).

**Example: Deploying to a Linux VM**
```yaml
jobs:
  deploy-web:
    uses: YourOrg/ci-cd-templates/.github/workflows/nextjs-deploy.yml@main
    with:
      working-dir: "."
      node-version: "20.x"
      deploy-target: "vm"
    secrets:
      SSH_HOST: ${{ secrets.VM_HOST }}
      SSH_USER: ${{ secrets.VM_USER }}
      SSH_PRIVATE_KEY: ${{ secrets.VM_SSH_KEY }}