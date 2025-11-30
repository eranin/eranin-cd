# Helm Deploy to ArgoCD

## Overview
This workflow automates continuous delivery into the ArgoCD GitOps repository.  
It updates Helm values with the latest Docker image (repository + tag), commits the change, and pushes it to the infrastructure repository—triggering ArgoCD to deploy automatically.

## Features
- GitOps-based delivery to ArgoCD  
- Automatic `values.yaml` patching using `yq`  
- Hermetic & deterministic environment  
- Safe commit logic (commits only when changes are detected)  
- Fully reusable via `workflow_call`

## Repository Structure
```
.github/
└── workflows/
    └── deploy_helm_argocd.yml
```

## Usage: Triggering the Deployment Workflow

In your application repository:

```yaml
name: Deploy Service

on:
  workflow_dispatch:

jobs:
  deploy:
    uses: eranin/eranin-ci/.github/workflows/deploy_helm_argocd.yml@main
    with:
      app_name: "emoney-be"
      values_path: "apps/emoney/values.yaml"
      image_name: "emoney-be"
      image_tag: ${{ needs.build.outputs.image_tag }}
      helm_repo: "eranin/eranin-infra"
    secrets:
      INFRA_REPO_TOKEN: ${{ secrets.INFRA_REPO_TOKEN }}
      IMAGE_REGISTRY: ${{ secrets.REGISTRY_URL }}
```

## Inputs

| Name | Required | Description |
|------|----------|-------------|
| `app_name` | Yes | Identifier used in commit messages |
| `values_path` | Yes | Path to Helm `values.yaml` inside infra repo |
| `image_name` | Yes | Docker image name without registry |
| `image_tag` | Yes | Tag produced by CI pipeline |
| `helm_repo` | No | Infra repo containing Helm manifests (default: `eranin/eranin-infra`) |

## Secrets

| Secret | Description |
|--------|-------------|
| `INFRA_REPO_TOKEN` | Token with write access to infra repository |
| `IMAGE_REGISTRY` | Registry host name (e.g., `103.162.20.16:5000`) |

## Workflow Steps

### 1. Checkout Infra Repository
Pulls the GitOps infrastructure repository that ArgoCD watches.

### 2. Install YQ
Uses a deterministic download method to avoid version drift.

### 3. Patch Helm Values
Updates:
- `.deployment.image.repository`
- `.deployment.image.tag`

This ensures the application uses the latest image.

### 4. Commit and Push Changes
- Configures bot identity  
- Commits only when actual file changes are detected  
- Pushes to the infra repo to trigger ArgoCD sync  

### 5. GitHub Summary
Shows the updated image reference in the Actions UI.

## Best Practices
- Pair this workflow after your Docker Build CI workflow  
- Keep infra repo protected with PR rules if needed  
- Use strongly scoped PAT tokens for CD operations  
- Apply ArgoCD sync windows in production environments  

## Troubleshooting

### 1. File Not Updated
Check path correctness:
```
values_path: "apps/service/values.yaml"
```

### 2. ArgoCD Not Deploying
- Ensure ArgoCD is monitoring the correct branch  
- Verify repository permissions  

### 3. “No Changes to Commit”
The workflow detected no modification—ensure the Helm chart uses the correct keys:

```
deployment.image.repository
deployment.image.tag
```

## Contact
For support, contact **Eranin DevOps Team**.
