# helloworld - Argo CD Application (STAGING)

This repository contains the Argo CD Application manifest for the **STAGING** environment of the helloworld essesseff™ app.  

It is ***not necessary*** to be an essesseff™ subscriber in order to make use of the standardized pattern and automation offered in this and corresponding code and config repositories for configuring and managing your Spring Boot application to follow said standardized pattern of development, build, deployment and promotion through DEV -> QA -> STAGING -> PROD environments, although it should not surprise you that it will be much easier for essesseff™ subscribers to do so.

*Please Note:*

*essesseff™ is an independent DevOps ALM PaaS-as-SaaS and is in no way affiliated with, endorsed by, sponsored by, or otherwise connected to GitHub® or The Linux Foundation®.* 

*essesseff™ is a trademark of essesseff LLC.*

*GITHUB®, the GITHUB® logo design and the INVERTOCAT logo design are trademarks of GitHub, Inc., registered in the United States and other countries.*

*Argo®, Helm®, Kubernetes® and K8s® are registered trademarks of The Linux Foundation.*

## See Also

- [essesseff Documentation](https://essesseff.com/docs) - essesseff platform documentation
- [Argo CD Documentation](https://argo-cd.readthedocs.io/) - Argo CD documentation
- [Helm Documentation](https://helm.sh/docs) - Helm documentation
- [Kubernetes Documentation](https://kubernetes.io/docs/home/) - Kubernetes (K8s) documentation
- [GitHub Documentation](https://docs.github.com/en) - GitHub documentation

## Repository Structure

```
helloworld-argocd-staging/
├── app-of-apps.yaml.template                      # Root Application template
├── argocd/
│   └── helloworld-staging-application.yaml.template # STAGING environment Application manifest (auto-synced) template
├── argocd-repository-secret.yaml.template            # Argo CD repository secrets template
├── ghcr-credentials-secret.yaml.template             # GHCR credentials (set once per K8s cluster for organization) template
├── notifications-configmap.yaml.template             # Argo CD notifications configuration template
├── offboarding/
│   └── offboard-essesseff-helloworld-springboot-templat.sh # script for offboarding the essesseff-helloworld-springboot-templat namespace from K8s
│   └── offboard-helloworld-staging.sh               # script for offboarding the helloworld staging app 1) from essesseff only or 2) from Argo CD and K8s entirely
├── setup-argocd-cluster.sh           # Argo CD K8s setup script 
├── setup-argocd.sh                   # Argo CD helloworld-staging essesseff app setup script 
└── README.md                         # This file
```

## Architecture

- **Deployment Model**: Trunk-based development (single `main` branch)
- **Deploy**: Promote, re-deploy and rollback (via essesseff UX).  For non-essesseff subscribers and for otherwise strictly GitOps deployments, STAGING deployments and code promotions can be accomplished through commit(s) to the config-staging repo, in particular the Helm Chart.yaml and values.yaml.
- **GitOps**: Managed by Argo CD with automated sync

## Quick Start

### (if not done already) Deploy/Configure Argo CD on the Environment-specific Kubernetes Cluster 

1. **Run Argo CD cluster setup script**:
```bash
   chmod 744 setup-argocd-cluster.sh
   ./setup-argocd-cluster.sh
   ```
   
### Configure helloworld-staging essesseff App to Argo CD and deploy to K8s

1. **Git Clone This Repository**:
   ```bash
   git clone git@github.com:essesseff-helloworld-springboot-templat/helloworld-argocd-staging.git
   ```
   
2. **Configure Environment Variables in .env File**:
   ```bash
   cp env.example .env
   ```
   Then set the environment variables in the .env which will be used for generating the following from the templates in this repository:

      a. ***Configuration of Argo CD repository access***:
   
      argocd-repository-secret.yaml with your GitHub Argo CD machine username and token
   
      This creates secrets for Argo CD to access:
      - `helloworld-argocd-staging` repository (to read Application manifests)
      - `helloworld-config-staging` repository (to read Helm charts and values)

      b. ***Configuration of Argo CD access to GitHub Container Registry (GHCR)***:
   
      ghcr-credentials-secret.yaml with your GitHub Argo CD machine username, token, email, and base64 credentials

      c. ***Configuration of helloworld-staging Deployment in Argo CD***:

      helloworld-staging-application.yaml is used to configure the helloworld-staging deployment

      d. ***Configuration of helloworld-argocd-staging App-of-Apps Deployment in Argo CD***:

      app-of-apps.yaml is used to configure the helloworld-argocd-staging app-of-apps deployment

      e. ***Configuration of helloworld-staging Argo CD Notifications to essesseff***:

      If notifications-secret.yaml is downloaded from essesseff for helloworld-staging, notifications-configmap.yaml will be used to configure Argo CD notifications to essesseff.
   
      **Note**: This secret can be set once for the entire GitHub organization / K8s namespace and will be used by Argo CD to pull container images from GHCR for all environments. You do not need to create separate secrets for each environment repository but should set the ghcr-credentials secret at least once per K8s namespace in each relevant K8s cluster.  *If the ghcr-credentials-secret.yaml.template file is not present, the setup-argocd.sh script will assume that the ghcr-credentials secret is already set for the given K8s namespace on the env-specific K8s cluster and move on.*

3. **(if an essesseff-subscribed app) Configure Argo CD Notifications Secrets**:

   Request the notifications-secret.yaml file contents from the essesseff UX for helloworld here:
   https://www.essesseff.com/home/[YOUR_essesseff_TEAM_ACCOUNT]/apps/helloworld/settings

   Copy the downloaded file to ./notifications-secret.yaml 

4. **Run the setup-argocd.sh Script**:
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

6. **Access the Deployed Application**:
   ```bash
   kubectl port-forward service/helloworld-staging 8081:80 -n essesseff-helloworld-springboot-templat
   # Access: http://localhost:8081
   ```
### How to Offboard helloworld-staging Deployment from Argo CD and K8s

1. **Execute the offboarding script**:
   ```bash
   cd offboarding
   chmod 744 offboard-helloworld-staging.sh
   ./offboard-helloworld-staging.sh
   ```

### How to Offboard essesseff-helloworld-springboot-templat K8s Namespace and All of its Resources

1. **Execute the offboarding script**:
   ```bash
   cd offboarding
   chmod 744 offboard-essesseff-helloworld-springboot-templat.sh
   ./offboard-essesseff-helloworld-springboot-templat.sh
   ```

## Application Details

- **Name**: `helloworld-staging`
- **Namespace**: `argocd`
- **Source Repository**: `helloworld-config-staging`
- **Destination Namespace**: `essesseff-helloworld-springboot-templat`
- **Sync Policy**: Automated with prune and self-heal enabled

## Deployment Process

### STAGING Code Promotion Deployment (essesseff-Subscribed App)

1. QA or DevOps Engineer declares QA deployment STABLE in essesseff UX
2. Release Engineer deploys STABLE release to STAGING in essesseff UX
3. essesseff GitHub App automation triggers essesseff to auto-update Helm `helloworld-config-staging/Chart.yaml` and `helloworld-config-staging/values.yaml` with the image tag of the newly built image
4. Argo CD syncs STAGING Application automatically on K8s

### Manual GitOps Deployment

1. Push update(s) to the `main` branch in `helloworld-config-staging` repository, typically to Helm Chart.yaml and/or values.yaml
2. Argo CD syncs STAGING Application automatically on K8s

## Repository URLs

- **Source**: `https://github.com/essesseff-helloworld-springboot-templat/helloworld`
- **Config STAGING**: `https://github.com/essesseff-helloworld-springboot-templat/helloworld-config-staging`
- **Argo CD STAGING**: `https://github.com/essesseff-helloworld-springboot-templat/helloworld-argocd-staging` (this repo)

## essesseff Integration

This setup requires the essesseff platform for automated deployment orchestration:

- **Decision-driven promotions**: STAGING deployment executed by Release Engineer of STABLE release via essesseff UX
- **RBAC enforcement**: Role-based access control for code and config development, build, deployment, promotion, etc.
- **Audit trail**: Complete history of all builds, deployments, promotions, etc.

## Argo CD Configuration

### Reduce Git Polling Interval (Optional)

By default, Argo CD polls Git repositories every ~3 minutes (120-180 seconds). To reduce this to, for example, 30 seconds for faster change detection:

```bash
kubectl patch configmap argocd-cm -n argocd --type merge -p '{"data":{"timeout.reconciliation":"30s","timeout.reconciliation.jitter":"10s"}}'
```

This will:
- Set base polling interval to 30 seconds
- Add up to 10 seconds of jitter (total: 30-40 seconds)
- Allow Argo CD to detect changes in `argocd/helloworld-staging-application.yaml` more quickly

## How It Works

1. **essesseff manages** image lifecycle and promotion decisions
2. **essesseff updates** Helm `Chart.yaml` and `values.yaml` files in config repos with approved image tags
3. **Argo CD detects** changes via Git polling (default: ~3 minutes, configurable to 30 seconds, as in the example above, or to the interval of your choosing)
4. **Argo CD syncs** Application automatically (auto-sync enabled)
5. **Kubernetes resources** are updated with new image versions and/or configuration settings as per Helm chart and overrides i.e. values.yaml settings

## See Also

- [essesseff Documentation](https://essesseff.com/docs) - essesseff platform documentation
- [Argo CD Documentation](https://argo-cd.readthedocs.io/) - Argo CD documentation
- [Helm Documentation](https://helm.sh/docs) - Helm documentation
- [Kubernetes Documentation](https://kubernetes.io/docs/home/) - Kubernetes (K8s) documentation
- [GitHub Documentation](https://docs.github.com/en) - GitHub documentation
  
