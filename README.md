# H&M ML Platform - GitOps Repository

This repository contains the GitOps configuration for deploying and managing ML components on Kubernetes using ArgoCD and Argo Workflows.

## 📁 Repository Structure

```
my-gitops/
├── .github/
│   └── workflows/
│       └── validate-gitops.yaml    # CI pipeline để kiểm tra repo này
│
├── argocd-root/                      # <--- THƯ MỤC CỦA "ỨNG DỤNG MẸ"
│   └── root-application.yaml       #     (File duy nhất bạn apply thủ công)
│
├── apps/                             # Nơi định nghĩa CÁCH CHẠY ứng dụng
│   ├── ml-recommendation-inference/
│   │   ├── base/
│   │   └── overlays/
│   │       ├── production/
│   │       └── staging/
│   └── ml-training-workflow-circle/
│       ├── base/
│       └── overlays/
│
├── bootstrap/                        # <--- Nơi định nghĩa "Bảng Phân Công Nhiệm Vụ"
│   ├── apps/                         #     cho các ứng dụng con gồm các applicationset trỏ đến từng apps từng môi trường
│   │   ├── ml-recommendation-prod-inference.yaml
│   │   ├── ml-recommendation-staging-inference.yaml
│   │   ├── ml-training-workflow-prod-circle.yaml
│   │   └── ml-training-workflow-staging-circle.yaml
│   └── platform/                     #     (Tùy chọn) Nhiệm vụ cho các thành phần platform
│       ├── monitoring-app.yaml
│       └── logging-app.yaml
│
└── manifests/                        # Nơi định nghĩa các thành phần platform
    ├── monitoring/
    └── logging/
```

## 🚀 Quick Start

### Prerequisites

- Kubernetes cluster with ArgoCD installed
- Argo Workflows controller installed
- AWS ECR access configured
- kubectl configured to access your cluster

### Initial Setup

1. **Update Repository URL**: Edit `argocd-root/root-application.yaml` and replace `https://github.com/your-org/hm-mlops-gitops.git` with your actual repository URL.

2. **Update Image Registry**: Edit the overlays in `apps/*/overlays/*/kustomization.yaml` to use your ECR registry URLs.

3. **Apply Root Application**:
   ```bash
   kubectl apply -f argocd-root/root-application.yaml
   ```

4. **Verify**: Check ArgoCD UI to see all applications being created and synced.

## 📋 Components

### ML Recommendation Inference

- **Base**: Common configuration for inference service
- **Production Overlay**: Production-specific settings (5 replicas, higher resources)
- **Staging Overlay**: Staging-specific settings (2 replicas, lower resources)

### ML Training Workflow Circle

- **Base**: Argo Workflow template defining the training pipeline
  - Data Ingestion → Data Processing → Data EDA → Model Training
- **Production Overlay**: Production training settings
- **Staging Overlay**: Staging training settings

### Platform Components

- **Monitoring**: Prometheus, Grafana stack (placeholder)
- **Logging**: ELK stack or similar (placeholder)

## 🔄 Workflow

1. **CI/CD Pipeline** (from `h&m_deeplearning` repo):
   - Builds Docker images and pushes to ECR with immutable tags
   - Tags: `main-{sha}`, `develop-{sha}`, etc.

2. **GitOps Sync** (this repo):
   - ArgoCD monitors this repository
   - Automatically syncs changes to Kubernetes cluster
   - Applications use Kustomize overlays to select appropriate image tags

3. **Image Updates**:
   - Update image tags in overlay `kustomization.yaml` files
   - Commit and push → ArgoCD syncs automatically
   - Or use ArgoCD Image Updater for automated updates

## 🛠️ Development

### Validate Locally

```bash
# Validate YAML syntax
yamllint apps/ bootstrap/ argocd-root/

# Validate Kustomize builds
kustomize build apps/ml-recommendation-inference/overlays/production

# Validate ArgoCD Applications
kubectl apply --dry-run=client -f argocd-root/root-application.yaml
```

### CI Pipeline

The `.github/workflows/validate-gitops.yaml` workflow automatically:
- Validates YAML syntax
- Validates ArgoCD Application schemas
- Validates Kustomize builds
- Validates repository structure

## 📝 Best Practices

1. **Never edit resources directly in cluster** - Always update Git and let ArgoCD sync
2. **Use Kustomize overlays** - Separate base from environment-specific configs
3. **Immutable image tags** - Use SHA-based tags, not `latest`
4. **Sync waves** - Use `argocd.argoproj.io/sync-wave` annotations for ordering
5. **ApplicationSets** - Use for managing multiple similar applications

## 🔒 Security

- ServiceAccounts use AWS IRSA (IAM Roles for Service Accounts)
- IAM roles are environment-specific
- Secrets should be managed via Sealed Secrets or External Secrets Operator
- No hardcoded credentials in manifests

## 📚 References

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Argo Workflows Documentation](https://argoproj.github.io/workflows/)
- [Kustomize Documentation](https://kustomize.io/)

