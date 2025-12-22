# helloworld - Argo CD Application (STAGING)

This repository contains the Argo CD Application manifest for the **STAGING** environment of the helloworld essesseff app.

## See Also

- [essesseff Documentation](https://essesseff.com/docs) - essesseff platform documentation
- [Argo CD Documentation](https://argo-cd.readthedocs.io/) - Argo CD documentation
- [Helm Documentation](https://helm.sh/docs) - Helm documentation
- [Kubernetes Documentation](https://kubernetes.io/docs/home/) - Kubernetes (K8s) documentation
- [GitHub Documentation](https://docs.github.com/en) - GitHub documentation

## Repository Structure

```
helloworld-argocd-staging/
├── app-of-apps.yaml                  # Root Application
├── argocd/
│   └── helloworld-staging-application.yaml  # STAGING environment Application manifest (auto-synced)
├── argocd-repository-secret.yaml     # Argo CD repository secrets
├── ghcr-credentials-secret.yaml      # GHCR credentials (set once per K8s cluster for organization)
├── notifications-configmap.yaml      # Argo CD notifications configuration
├── setup-argocd-cluster.sh           # Argo CD K8s setup script 
├── setup-argocd.sh                   # Argo CD helloworld-staging essesseff app setup script 
└── README.md                          # This file
```

## Architecture

- **Deployment Model**: Trunk-based development (single `main` branch)
- **Manual Deploy**: Enabled (via essesseff UI with RBAC)
- **GitOps**: Managed by Argo CD with automated sync

## Quick Start

### Deploy/Configure Argo CD on the Environment-specific Kubernetes Cluster (if not done already)

1. **Run Argo CD cluster setup script**:
```bash
   chmod 744 setup-argocd-cluster.sh
   ./setup-argocd-cluster.sh
   ```
   
### Deploy helloworld-staging essesseff App to Argo CD

1. **Configure Argo CD repository access** (if not already done):
   
   Edit argocd-repository-secret.yaml with your GitHub Argo CD machine username and token
   
   This creates secrets for Argo CD to access:
   - `helloworld-argocd-staging` repository (to read Application manifests)
   - `helloworld-config-staging` repository (to read Helm charts and values)

2. **Configure Argo CD access to GitHub Container Registry (GHCR)**:
   
   Edit ghcr-credentials-secret.yaml with your GitHub Argo CD machine username, token, email, and base64 credentials
   
   **Note**: This secret can be set once for the entire GitHub organization and will be used by Argo CD to pull container images from GHCR for all environments. You do not need to create separate secrets for each environment repository.

3. **Configure Argo CD notifications secrets**:

   Request the notifications-secret.yaml file contents from the essesseff UX for helloworld here:
   https://www.essesseff.com/home/YOUR_essesseff_TEAM_ACCOUNT/apps/helloworld/settings

   Save the contents to ./notifications-secret.yaml 

4. **Run the setup-argocd.sh script**:
   ```bash
   chmod 744 setup-argocd.sh
   ./setup-argocd.sh
   ```

   This script applies all secrets, configmaps, Argo CD application definitions, etc. for helloworld STAGING.

5. **Verify in Argo CD UI**:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   # Access: https://localhost:8080
   ```
   
   You should see:
   - `helloworld-argocd-staging` - Root Application (watches this repository)
   - `helloworld-staging` - Environment Application (auto-synced by root Application)

6. **Access the deployed application**:
   ```bash
   kubectl port-forward service/helloworld-staging 8081:80 -n essesseff-helloworld-springboot-templat
   # Access: http://localhost:8081
   ```

## Application Details

- **Name**: `helloworld-staging`
- **Namespace**: `argocd`
- **Source Repository**: `helloworld-config-staging`
- **Destination Namespace**: `essesseff-helloworld-springboot-templat`
- **Sync Policy**: Automated with prune and self-heal enabled

## Deployment Process

### Manual Deployment

1. **QA Engineer** marks image as Stable
2. **Release Engineer** deploys to STAGING (optional)
3. Argo CD syncs STAGING Application automatically

## Repository URLs

- **Source**: `https://github.com/essesseff-helloworld-springboot-templat/helloworld`
- **Config STAGING**: `https://github.com/essesseff-helloworld-springboot-templat/helloworld-config-staging`
- **Argo CD STAGING**: `https://github.com/essesseff-helloworld-springboot-templat/helloworld-argocd-staging` (this repo)

## essesseff Integration

This setup requires the essesseff platform for deployment orchestration:

- **RBAC enforcement**: Role-based access control for deployments
- **Approval workflows**: Manual approvals for STAGING deployments
- **Deployment policies**: Enforced promotion paths (Stable → STAGING)
- **Audit trail**: Complete history of all deployments and approvals

## Argo CD Configuration

### Reduce Git Polling Interval (Optional)

By default, Argo CD polls Git repositories every ~3 minutes (120-180 seconds). To reduce this to 60 seconds for faster change detection:

```bash
kubectl patch configmap argocd-cm -n argocd --type merge -p '{"data":{"timeout.reconciliation":"60s","timeout.reconciliation.jitter":"10s"}}'
```

This will:
- Set base polling interval to 60 seconds
- Add up to 10 seconds of jitter (total: 60-70 seconds)
- Allow Argo CD to detect changes in `argocd/helloworld-staging-application.yaml` more quickly

## How It Works

1. **essesseff manages** image lifecycle and promotion decisions
2. **essesseff updates** `Chart.yaml` and `values.yaml` files in config repos with approved image tags
3. **Argo CD detects** changes via Git polling (default: ~3 minutes, configurable to 60 seconds)
4. **Argo CD syncs** Application automatically (auto-sync enabled)
5. **Kubernetes resources** are updated with new image versions

## See Also

- [essesseff Documentation](https://essesseff.com/docs) - essesseff platform documentation
- [Argo CD Documentation](https://argo-cd.readthedocs.io/) - Argo CD documentation
- [Helm Documentation](https://helm.sh/docs) - Helm documentation
- [Kubernetes Documentation](https://kubernetes.io/docs/home/) - Kubernetes (K8s) documentation
- [GitHub Documentation](https://docs.github.com/en) - GitHub documentation


