# Using Reusable Workflows from Other Repositories

This guide shows how to use the reusable workflows from other repositories using absolute paths.

---

## 📍 Absolute Path Format

When calling reusable workflows from another repository, use this format:

```yaml
uses: {owner}/{repo}/.github/workflows/{workflow-file}@{ref}
```

Where:
- `{owner}` - GitHub organization or username (e.g., `your-org` or `huayuenh`)
- `{repo}` - Repository name (e.g., `workflows` or `deploy-to-cluster-sample-action-workflow`)
- `{workflow-file}` - Workflow filename (e.g., `deploy-reusable.yml`)
- `{ref}` - Git reference: branch (`main`), tag (`v1.0.0`), or commit SHA

---

## 🚀 Deployment Workflow

### Example 1: Using @main (Auto-Updates)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    uses: huayuenh/deploy-to-cluster-sample-action-workflow/.github/workflows/deploy-reusable.yml@main
    with:
      environment: production
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

**Pros:** Automatically gets updates
**Cons:** Breaking changes could affect your workflow

---

### Example 2: Using Version Tag (Stable)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    uses: huayuenh/deploy-to-cluster-sample-action-workflow/.github/workflows/deploy-reusable.yml@v1.0.0
    with:
      environment: production
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

**Pros:** Stable, predictable behavior
**Cons:** Must manually update to get new features

---

### Example 3: Using Commit SHA (Maximum Stability)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    uses: huayuenh/deploy-to-cluster-sample-action-workflow/.github/workflows/deploy-reusable.yml@abc123def456
    with:
      environment: production
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

**Pros:** Immutable, maximum stability
**Cons:** Hardest to update

---

## 🔍 PR Validation Workflow

### Example 1: Using @main

```yaml
# .github/workflows/pr-validation.yml
name: PR Validation

on:
  pull_request:
    types: [opened, reopened, synchronize]

jobs:
  validate:
    uses: huayuenh/deploy-to-cluster-sample-action-workflow/.github/workflows/pr-validation-reusable.yml@main
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

---

### Example 2: Using Version Tag

```yaml
# .github/workflows/pr-validation.yml
name: PR Validation

on:
  pull_request:
    types: [opened, reopened, synchronize]

jobs:
  validate:
    uses: huayuenh/deploy-to-cluster-sample-action-workflow/.github/workflows/pr-validation-reusable.yml@v1.0.0
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

---

## 🏢 Organization-Wide Setup (Recommended)

For organizations, create a dedicated workflows repository:

### Step 1: Create Central Repository

```
your-org/workflows/
└── .github/
    └── workflows/
        ├── deploy-reusable.yml
        └── pr-validation-reusable.yml
```

### Step 2: Consumer Repositories Use Absolute Paths

```yaml
# Any repository in your-org
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    uses: your-org/workflows/.github/workflows/deploy-reusable.yml@main
    with:
      environment: production
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

---

## 📦 Complete Example Repository

Here's what a consumer repository would look like:

### Repository Structure:
```
my-microservice/
├── .github/
│   └── workflows/
│       ├── deploy.yml          # 23 lines
│       └── pr-validation.yml   # 11 lines
├── .k8s/
│   └── deployment.yaml
├── .deploy.yaml
├── .env.template
├── Dockerfile
└── src/
```

### deploy.yml (23 lines):
```yaml
name: Deploy to Kubernetes

on:
  push:
    branches: [ main, develop ]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment'
        required: true
        type: choice
        options: [production, staging, development]

jobs:
  deploy:
    uses: your-org/workflows/.github/workflows/deploy-reusable.yml@v1.0.0
    with:
      environment: ${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

### pr-validation.yml (11 lines):
```yaml
name: PR Validation

on:
  pull_request:
    types: [opened, reopened, synchronize]

jobs:
  validate:
    uses: your-org/workflows/.github/workflows/pr-validation-reusable.yml@v1.0.0
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

---

## 🔐 Secrets Management

### Option 1: Repository Secrets (Current)
Each repository defines its own secrets:
```yaml
secrets:
  ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

### Option 2: Organization Secrets (Recommended)
Set secrets at organization level, available to all repos:
```yaml
secrets:
  ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}  # Org-level secret
```

### Option 3: Inherit Secrets (Simplest)
Use `secrets: inherit` to pass all secrets:
```yaml
jobs:
  deploy:
    uses: your-org/workflows/.github/workflows/deploy-reusable.yml@main
    with:
      environment: production
    secrets: inherit  # Passes all secrets automatically
```

---

## 🎯 Versioning Strategy

### Semantic Versioning (Recommended)

```bash
# Tag releases
git tag -a v1.0.0 -m "Initial release"
git tag -a v1.1.0 -m "Add new feature"
git tag -a v2.0.0 -m "Breaking change"
git push --tags
```

Consumer repositories reference versions:
```yaml
uses: your-org/workflows/.github/workflows/deploy-reusable.yml@v1.1.0
```

### Branch Strategy

- `@main` - Latest stable (auto-updates)
- `@develop` - Development version (testing)
- `@v1` - Major version branch (v1.x.x updates)
- `@v1.0.0` - Specific version (immutable)

---

## 📊 Migration Example

### Before (132 lines per repo):
```
50 microservices × 132 lines = 6,600 lines
```

### After (23 lines per repo):
```
50 microservices × 23 lines = 1,150 lines
1 central workflow = 181 lines
Total = 1,331 lines
```

**Savings: 80% reduction (6,600 → 1,331 lines)**

---

## 🔄 Update Process

### Updating All Repositories

**With @main:**
```bash
# Update central workflow
cd workflows
git commit -m "Improve deployment"
git push

# All repos using @main get update automatically!
```

**With version tags:**
```bash
# Update central workflow
cd workflows
git commit -m "Improve deployment"
git tag v1.1.0
git push --tags

# Repos update when they change version:
# uses: your-org/workflows/...@v1.0.0  →  @v1.1.0
```

---

## 🎓 Best Practices

### 1. Use Version Tags for Production
```yaml
# Production deployments
uses: your-org/workflows/.github/workflows/deploy-reusable.yml@v1.0.0
```

### 2. Use @main for Development
```yaml
# Development/staging
uses: your-org/workflows/.github/workflows/deploy-reusable.yml@main
```

### 3. Test Changes in Development First
```yaml
# Test new workflow version
uses: your-org/workflows/.github/workflows/deploy-reusable.yml@develop
```

### 4. Document Breaking Changes
```markdown
# CHANGELOG.md
## v2.0.0 (Breaking)
- Changed `cluster-name` to required parameter
- Migration: Add `cluster-name` input to all workflows
```

---

## 🚀 Quick Start for New Repository

### 1. Copy these 2 files:

**deploy.yml:**
```yaml
name: Deploy
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    uses: your-org/workflows/.github/workflows/deploy-reusable.yml@v1.0.0
    with:
      environment: production
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

**pr-validation.yml:**
```yaml
name: PR Validation
on:
  pull_request:
    types: [opened, reopened, synchronize]
jobs:
  validate:
    uses: your-org/workflows/.github/workflows/pr-validation-reusable.yml@v1.0.0
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

### 2. Ensure these files exist:
- `.deploy.yaml`
- `.env.template`
- `.k8s/deployment.yaml`
- `Dockerfile`

### 3. Set repository variables:
- `ICR_REGION`
- `ICR_NAMESPACE`
- `IBMCLOUD_REGION`
- `CLUSTER_NAME`

### 4. Set repository secret:
- `IBMCLOUD_API_KEY`

### 5. Done! 🎉

---

## 📞 Support

For issues with:
- **Reusable workflows:** Open issue in `your-org/workflows`
- **Your deployment:** Check your repository workflow logs
- **Actions:** Check composite action repository

---

## Summary

**Relative Path (Same Repository):**
```yaml
uses: ./.github/workflows/deploy-reusable.yml
```

**Absolute Path (Other Repository):**
```yaml
uses: your-org/workflows/.github/workflows/deploy-reusable.yml@v1.0.0
```

**Result:**
- ✅ 80% less code across organization
- ✅ Centralized maintenance
- ✅ Consistent deployments
- ✅ Easy updates
- ✅ Version control

**Start using reusable workflows today!** 🚀