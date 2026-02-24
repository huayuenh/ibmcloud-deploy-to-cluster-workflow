# Reusable Workflow Guide

This repository now includes a **reusable workflow** that dramatically simplifies deployment configuration.

## 📊 Comparison

### Before (Composite Action)
```yaml
# 132 lines of workflow configuration
# 30+ parameters to configure
# Complex error handling
# Manual rollback logic
```

### After (Reusable Workflow)
```yaml
# 23 lines of workflow configuration
# 2-3 parameters to configure
# Built-in error handling
# Automatic rollback
```

**Result: 83% reduction in workflow code!**

---

## 🚀 Quick Start

### 1. Simple Deployment

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    uses: ./.github/workflows/deploy-reusable.yml
    with:
      environment: production
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

That's it! The reusable workflow handles:
- ✅ Building Docker image
- ✅ Pushing to IBM Cloud Container Registry
- ✅ Deploying to Kubernetes cluster
- ✅ Running acceptance tests
- ✅ Automatic rollback on failure

---

## 📝 Configuration

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `environment` | Deployment environment | `production`, `staging`, `development` |

### Required Secrets

| Secret | Description |
|--------|-------------|
| `ibmcloud-apikey` | IBM Cloud API key for authentication |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `image-tag` | Docker image tag | `github.sha` |
| `cluster-name` | Kubernetes cluster name | `vars.CLUSTER_NAME` |
| `cluster-region` | Cluster region | `us-south` |
| `run-acceptance-tests` | Run tests after deployment | `true` |
| `auto-rollback` | Auto-rollback on failure | `true` |

---

## 📚 Usage Examples

### Example 1: Production Deployment

```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    uses: ./.github/workflows/deploy-reusable.yml
    with:
      environment: production
      run-acceptance-tests: true
      auto-rollback: true
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

### Example 2: Multi-Environment

```yaml
name: Deploy

on:
  push:
    branches: [ main, develop ]

jobs:
  deploy:
    uses: ./.github/workflows/deploy-reusable.yml
    with:
      environment: ${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

### Example 3: Manual Deployment with Custom Tag

```yaml
name: Manual Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment'
        required: true
        type: choice
        options: [production, staging, development]
      image-tag:
        description: 'Image tag'
        required: false
        type: string

jobs:
  deploy:
    uses: ./.github/workflows/deploy-reusable.yml
    with:
      environment: ${{ inputs.environment }}
      image-tag: ${{ inputs.image-tag }}
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

### Example 4: Deploy Without Tests

```yaml
name: Quick Deploy

on:
  workflow_dispatch:

jobs:
  deploy:
    uses: ./.github/workflows/deploy-reusable.yml
    with:
      environment: development
      run-acceptance-tests: false
      auto-rollback: false
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

---

## 🔄 Workflow Jobs

The reusable workflow includes 4 jobs:

### 1. Build
- Builds Docker image
- Pushes to IBM Cloud Container Registry
- Scans for vulnerabilities

### 2. Deploy
- Deploys to Kubernetes cluster
- Configures ingress
- Runs health checks
- Outputs application URL

### 3. Test (Optional)
- Runs acceptance tests
- Uses deployed application URL
- Can be skipped with `run-acceptance-tests: false`

### 4. Rollback (Conditional)
- Triggers on deployment or test failure
- Automatically rolls back to previous version
- Can be disabled with `auto-rollback: false`

---

## 📤 Outputs

The reusable workflow provides outputs:

```yaml
jobs:
  deploy:
    uses: ./.github/workflows/deploy-reusable.yml
    with:
      environment: production
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
  
  notify:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Send notification
        run: |
          echo "Deployed to: ${{ needs.deploy.outputs.app-url }}"
          echo "Image: ${{ needs.deploy.outputs.image }}"
          echo "Status: ${{ needs.deploy.outputs.deployment-status }}"
```

---

## 🔧 Required Repository Setup

### 1. Repository Variables

Set these in **Settings → Secrets and variables → Actions → Variables**:

- `ICR_REGION` - IBM Cloud Container Registry region (e.g., `us.icr.io`)
- `ICR_NAMESPACE` - ICR namespace
- `IBMCLOUD_REGION` - IBM Cloud region (e.g., `us-south`)
- `CLUSTER_NAME` - Kubernetes cluster name

### 2. Repository Secrets

Set these in **Settings → Secrets and variables → Actions → Secrets**:

- `IBMCLOUD_API_KEY` - IBM Cloud API key

### 3. Environments

Create environments in **Settings → Environments**:

- `production`
- `staging`
- `development`

Configure protection rules as needed (e.g., require approval for production).

### 4. Required Files

Ensure these files exist in your repository:

- `.deploy.yaml` - Deployment configuration
- `.env.template` - Environment variable template
- `.k8s/deployment.yaml` - Kubernetes manifest template
- `Dockerfile` - Docker build instructions

---

## 🎯 Benefits

### For Individual Repositories

- ✅ **83% less code** (132 → 23 lines)
- ✅ **Simpler configuration** (2-3 parameters vs 30+)
- ✅ **Automatic updates** (update once, all repos benefit)
- ✅ **Built-in best practices** (rollback, health checks, etc.)

### For Organizations

- ✅ **Centralized maintenance** (update 1 workflow, not 50)
- ✅ **Consistent deployments** (same process everywhere)
- ✅ **Faster onboarding** (copy 23 lines, done!)
- ✅ **Better compliance** (audit 1 workflow, not 50)

---

## 🔄 Migration from Composite Action

### Step 1: Keep Old Workflow
Rename `ci-pipeline.yml` to `ci-pipeline-old.yml` (backup)

### Step 2: Create New Workflow
Create `deploy.yml` using reusable workflow (23 lines)

### Step 3: Test
Deploy to staging/development first

### Step 4: Switch
Once validated, use new workflow for production

### Step 5: Clean Up
Delete old workflow after successful migration

---

## 📖 Advanced Usage

### Using from Another Repository

If you publish the reusable workflow to a central repository:

```yaml
jobs:
  deploy:
    uses: your-org/workflows/.github/workflows/deploy-reusable.yml@main
    with:
      environment: production
    secrets:
      ibmcloud-apikey: ${{ secrets.IBMCLOUD_API_KEY }}
```

### Version Pinning

Pin to specific version for stability:

```yaml
uses: your-org/workflows/.github/workflows/deploy-reusable.yml@v1.0.0
```

Or use `@main` for automatic updates:

```yaml
uses: your-org/workflows/.github/workflows/deploy-reusable.yml@main
```

---

## 🆘 Troubleshooting

### Deployment Fails

Check the workflow run logs:
1. Build job - Image build issues
2. Deploy job - Cluster access or manifest issues
3. Test job - Application not responding

### Rollback Not Working

Ensure:
- `auto-rollback: true` is set
- Deployment has previous revision
- Cluster access is configured

### Tests Failing

- Check `APP_URL` environment variable
- Verify application is accessible
- Review test logs in Test job

---

## 📞 Support

For issues or questions:
1. Check workflow run logs
2. Review this documentation
3. Check composite action documentation
4. Open an issue in the repository

---

## 🎉 Summary

The reusable workflow provides:

- **83% less code** per repository
- **Automatic rollback** on failure
- **Built-in testing** with acceptance tests
- **Centralized maintenance** for easy updates
- **Consistent deployments** across all repositories

**Start using it today and simplify your deployments!** 🚀