# IBM Cloud Kubernetes CI Pipeline Example

A complete GitHub Actions workflow example demonstrating automated continuous integration and deployment to IBM Cloud Kubernetes Service (IKS) using reusable custom actions.

## Overview
test
This repository showcases production-ready CI pipelines for containerized applications on IBM Cloud. It demonstrates:

- **Pull Request CI** - Automated build, test, and scan for pull requests
- **Continuous Deployment** - Automated deployment on merge to main branch
- **Dynamic Image Naming** - Automatic image naming based on repository or custom inputs
- **Multi-Environment Deployment** - Branch-based namespace selection (production, staging, development)
- **Security Scanning** - Automated vulnerability scanning with IBM Cloud Container Registry
- **Pull Request Validation** - Isolated PR builds with automatic cleanup
- **GitHub Integration** - Commit status updates and deployment summaries

## GitHub Actions Workflows

### Continuous Deployment Pipeline

**File:** `.github/workflows/ci-pipeline.yml`

**Purpose:** Continuous deployment pipeline that builds, tests, scans, and deploys to Kubernetes on every push to main branch.

**Triggers:**
- Push to `main` branch
- Manual workflow dispatch with optional inputs

**Workflow Steps:**

1. **Clone Repository** - Checkout source code
2. **Run Unit Tests** - Execute Jest test suite
3. **Build Docker Image** - Build with dynamic naming and metadata
   - Uses `huayuenh/docker-container-build-action`
   - Automatically names image based on repository name or custom input
   - Adds build args and OCI labels
4. **Push to IBM Cloud Container Registry** - Push and scan image
   - Uses `huayuenh/container-registry-service-action`
   - Performs vulnerability scanning
   - Stores in IBM Cloud Container Registry
5. **Deploy to Kubernetes** - Deploy with full configuration
   - Uses `huayuenh/cluster-service-action`
   - Creates/updates deployment with 3 replicas
   - Configures ClusterIP service
   - Sets up automatic ingress with TLS
   - Performs health checks
   - Configures resource limits and requests
6. **Display Summary** - Show deployment details in GitHub Actions UI

**Key Features:**

```yaml
# Dynamic image naming - no hardcoded values
app-name: ${{ github.event.inputs.app-name }}  # Falls back to repo name

# Automatic namespace selection based on branch
namespace: production  # main branch
namespace: staging     # develop branch
namespace: development # other branches

# Automatic ingress with TLS
auto-ingress: true
ingress-tls: true

# Health check validation
health-check-path: /health
health-check-timeout: 300

# Resource management
resource-limits-cpu: 500m
resource-limits-memory: 512Mi
resource-requests-cpu: 250m
resource-requests-memory: 256Mi
```

**Workflow Inputs (Manual Dispatch):**

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `app-name` | Custom application name | ❌ | Repository name |
| `namespace` | Kubernetes namespace | ❌ | Branch-based |

**Outputs:**
- Image name and ID
- Registry location
- Deployment status
- Application URL
- Service IP
- Health check results

### Pull Request CI Pipeline

**File:** `.github/workflows/pr-pipeline.yml`

**Purpose:** Continuous integration for pull requests - builds, tests, and scans without deploying. Automatically cleans up PR images after validation.

**Triggers:**
- Pull request opened or reopened
- Manual workflow dispatch

**Workflow Steps:**

1. **Clone Repository** - Checkout PR branch
2. **Set Commit Status** - Mark PR as pending
3. **Run Unit Tests** - Execute Jest test suite
4. **Build Docker Image** - Build with PR-specific tag
   - Tag format: `pr-{number}-{sha}`
   - Example: `pr-42-abc123def`
5. **Push to Registry** - Push and scan image
6. **Cleanup** - Delete PR image from registry (runs always, even on failure)
7. **Display Summary** - Show build information
8. **Update Commit Status** - Mark PR as success or failure

**Key Features:**

```yaml
# PR-specific tagging
tag: pr-${{ github.event.pull_request.number }}-${{ github.sha }}

# Automatic cleanup - always runs
- name: Delete PR image from registry
  if: always()
  uses: huayuenh/container-registry-service-action@main
  with:
    action: delete

# GitHub commit status integration
- uses: huayuenh/set-commit-status-common-action@main
  with:
    state: "success"  # or "failure"
    context: "PR Pipeline"
```

**Benefits:**
- No deployment to production
- Isolated PR testing
- Automatic resource cleanup
- Clear PR status indicators
- No leftover images in registry

## Custom GitHub Actions Used

This example demonstrates the following reusable actions:

### 1. Docker Container Build Action
**Repository:** `huayuenh/docker-container-build-action`

**Purpose:** Build Docker images with smart defaults and dynamic naming

**Key Features:**
- Automatic image naming from repository name
- Optional custom `app-name` and `tag` inputs
- Build args and labels support
- Multi-platform builds
- Outputs image name, tag, and ID

### 2. Container Registry Service Action
**Repository:** `huayuenh/container-registry-service-action`

**Purpose:** Interact with IBM Cloud Container Registry

**Key Features:**
- Push/pull/delete operations
- Automatic vulnerability scanning
- Multi-region support
- Authentication handling

### 3. Cluster Service Action
**Repository:** `huayuenh/cluster-service-action`

**Purpose:** Deploy to IBM Cloud Kubernetes/OpenShift clusters

**Key Features:**
- Kubernetes and OpenShift support
- Automatic deployment generation
- Service and ingress creation
- Health check validation
- Resource configuration
- Environment variable injection

### 4. Set Commit Status Action
**Repository:** `huayuenh/set-commit-status-common-action`

**Purpose:** Update GitHub commit status

**Key Features:**
- Set pending/success/failure states
- Custom context and descriptions
- Link to workflow runs

## Configuration

### Required Secrets

Configure in GitHub repository settings → Secrets and variables → Actions:

| Secret | Description | Example |
|--------|-------------|---------|
| `IBMCLOUD_API_KEY` | IBM Cloud API key with Container Registry and Kubernetes permissions | `abc123...` |

### Required Variables

Configure in GitHub repository settings → Secrets and variables → Actions → Variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `IBMCLOUD_REGION` | IBM Cloud region | `us-south` |
| `ICR_NAMESPACE` | Container Registry namespace | `my-namespace` |
| `ICR_REGION` | Container Registry endpoint | `us.icr.io` |
| `CLUSTER_NAME` | Kubernetes cluster name | `my-iks-cluster` |
| `IKS_DEPLOYMENT_NAME` | Deployment name | `hello-app` |
| `RESOURCE_GROUP` | IBM Cloud resource group | `default` |

### Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `IMAGE_NAME` | Custom image name | Repository name |

## Quick Start

### 1. Fork or Clone This Repository

```bash
git clone https://github.com/yourusername/ghe-kube.git
cd ghe-kube
```

### 2. Configure Secrets and Variables

Add the required secrets and variables in your GitHub repository settings.

### 3. Push to Main Branch

```bash
git add .
git commit -m "Initial commit"
git push origin main
```

The continuous deployment pipeline will automatically:
- Run unit tests
- Build Docker image
- Push to IBM Cloud Container Registry
- Scan for vulnerabilities
- Deploy to Kubernetes
- Display deployment summary

### 4. Create a Pull Request

```bash
git checkout -b feature/my-feature
# Make changes
git commit -am "Add new feature"
git push origin feature/my-feature
```

Create a PR on GitHub. The PR CI pipeline will:
- Run unit tests
- Build and scan image
- Update commit status
- Clean up PR image

## Deployment Architecture

**Image Registry:**
- IBM Cloud Container Registry
- Automatic vulnerability scanning
- Multi-region support
- PR images automatically cleaned up

**Kubernetes Deployment:**
- 3 replicas for high availability
- ClusterIP service
- Automatic ingress with TLS
- Health check endpoint validation
- Resource limits and requests
- Environment variables injected

**Namespaces:**
- `production` - main branch deployments
- `staging` - develop branch deployments
- `development` - feature branch deployments

## Workflow Customization

### Change Image Name

**Option 1:** Use workflow input (manual dispatch)
```yaml
app-name: my-custom-app
```

**Option 2:** Set repository variable
```
IMAGE_NAME=my-custom-app
```

### Change Deployment Namespace

**Option 1:** Use workflow input (manual dispatch)
```yaml
namespace: my-namespace
```

**Option 2:** Modify branch-based logic in workflow
```yaml
DEPLOY_NAMESPACE: ${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}
```

### Adjust Resource Limits

Edit the deploy step in `ci-pipeline.yml`:
```yaml
resource-limits-cpu: 1000m
resource-limits-memory: 1Gi
resource-requests-cpu: 500m
resource-requests-memory: 512Mi
```

### Add Environment Variables

Edit the deploy step in `ci-pipeline.yml`:
```yaml
env-vars: |
  VERSION=${{ steps.image-tag.outputs.tag }}
  ENVIRONMENT=${{ env.DEPLOY_NAMESPACE }}
  BUILD_SHA=${{ github.sha }}
  MY_CUSTOM_VAR=value
```

## Sample Application

This repository includes a simple Node.js Express application for demonstration purposes.

**Technology:** Node.js 20, Express.js 4.x  
**Base Image:** Red Hat UBI 8  
**Endpoint:** `GET /` - Returns welcome message  
**Port:** 8080

### Local Development

```bash
# Install dependencies
npm install

# Run tests
npm run test-unit

# Start application
npm run start-server
```

### Docker

```bash
# Build
docker build -t hello-containers .

# Run
docker run -p 8080:8080 hello-containers
```

## Troubleshooting

### Workflow Fails at Build Step
- Check Dockerfile syntax
- Verify base image is accessible
- Review build logs in GitHub Actions

### Workflow Fails at Push Step
- Verify `IBMCLOUD_API_KEY` secret is set
- Check Container Registry namespace exists
- Ensure API key has Container Registry permissions

### Workflow Fails at Deploy Step
- Verify cluster name and region are correct
- Check API key has Kubernetes permissions
- Review cluster connectivity
- Check namespace exists or can be created

### Image Not Found
- Verify image was pushed successfully
- Check registry hostname and namespace
- Ensure image name matches between steps

## Best Practices Demonstrated

✅ **Pull Request CI** - Validate changes before merge
✅ **Continuous Deployment** - Automatic deployment on merge
✅ **Dynamic Naming** - No hardcoded image names
✅ **Reusable Actions** - Modular, composable workflows
✅ **Security Scanning** - Automated vulnerability detection
✅ **Resource Management** - CPU and memory limits
✅ **Health Checks** - Deployment validation
✅ **Clean Architecture** - Separation of concerns
✅ **Automatic Cleanup** - No orphaned resources
✅ **Status Updates** - Clear PR feedback
✅ **Deployment Summaries** - Actionable information

## License

Licensed under the Apache License, Version 2.0. See LICENSE for details.

## Contributing

This is an example repository demonstrating CI patterns with continuous deployment. Feel free to fork and adapt for your own projects.
