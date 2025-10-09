# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a GitOps-managed Kubernetes homelab repository using Flux CD. The cluster automatically syncs its state from the `main` branch of https://github.com/isaaclhk/Homelab.git.

## Repository Structure

The repository follows a structured GitOps layout:

```
labs/
├── clusters/           # Flux Kustomizations pointing to apps/infrastructure/monitoring
│   ├── flux-system/    # Flux bootstrap configuration (interval: 10m)
│   ├── home.yaml       # Points to apps/home with SOPS decryption enabled
│   ├── ebf.yaml        # Points to apps/ebf with SOPS decryption enabled
│   ├── infrastructure.yaml
│   └── monitoring.yaml
├── apps/
│   ├── base/           # Base manifests (namespace, deployment, service, storage)
│   │   ├── linkding/
│   │   ├── mealie/
│   │   ├── homarr/
│   │   ├── audiobookshelf/
│   │   └── ghost/
│   ├── home/           # Environment-specific overlays (Cloudflare tunnels, secrets)
│   └── ebf/
├── infrastructure/
│   └── renovate/       # Automatic dependency updates (CronJob runs @daily)
└── monitoring/
    ├── controllers/    # Prometheus-Grafana Helm release
    └── configs/        # Grafana TLS certificates
```

### Application Architecture Pattern

Each application follows a consistent two-layer pattern:

**Base Layer** (`apps/base/<app>/`):
- `namespace.yaml` - Application namespace
- `deployment.yaml` or `<app>-deployment.yaml` - Core application deployment with pinned image versions
- `service.yaml` - Kubernetes service (typically ClusterIP)
- `storage.yaml` - PersistentVolumeClaims for stateful data
- `kustomization.yaml` - References all base resources
- Additional resources as needed (e.g., `mysql.yaml`, `mysql-config.yaml` for Ghost)

**Environment Layer** (`apps/home/<app>/` or `apps/ebf/<app>/`):
- `tunnel.yaml` - Cloudflare tunnel deployment for external access (image: `cloudflare/cloudflared:latest`)
- `tunnel-token.yaml` - Encrypted SOPS secret containing tunnel token
- `kustomization.yaml` - References base resources and environment-specific overlays
- Additional secrets as needed (e.g., `db-secret.yaml` for Homarr)

## Key Technologies

- **GitOps**: Flux CD (reconciles cluster state every 10 minutes)
- **Secrets Management**: SOPS with age encryption (see `.sops.yaml`)
- **External Access**: Cloudflare Tunnels (all public-facing apps use remotely managed tunnels)
- **Monitoring**: Prometheus-Grafana stack (Helm chart, LAN-only access via Traefik ingress)
- **Dependency Management**: Renovate (self-hosted CronJob, runs daily)

## Common Commands

### Flux Management
```bash
# Check Flux system status
flux check

# View all Flux resources
flux get all

# Force reconciliation of a Kustomization
flux reconcile kustomization apps-home --with-source

# View Flux logs
flux logs --all-namespaces --follow

# Suspend/resume reconciliation
flux suspend kustomization apps-home
flux resume kustomization apps-home
```

### Secrets Management with SOPS

All secrets in `*.yaml` files are encrypted using SOPS with age encryption. The age public key is defined in `.sops.yaml`.

```bash
# Encrypt a secret file (REQUIRED before git push)
sops --encrypt --in-place labs/apps/home/<app>/tunnel-token.yaml

# Decrypt a secret file for viewing/editing
sops --decrypt labs/apps/home/<app>/tunnel-token.yaml

# Edit an encrypted file in-place
sops labs/apps/home/<app>/tunnel-token.yaml
```

**Important**: Always encrypt secret files before committing to the repository. Flux automatically decrypts them using the `sops-age` secret in the cluster.

### Application Deployment

```bash
# Apply changes manually (for testing before git push)
kubectl apply -k labs/apps/base/<app>/
kubectl apply -k labs/apps/home/<app>/

# View application status
kubectl get pods -n <app-namespace>
kubectl logs -n <app-namespace> <pod-name>
kubectl describe pod -n <app-namespace> <pod-name>

# Check Cloudflare tunnel health
kubectl logs -n <app-namespace> -l pod=cloudflared
```

### Monitoring Access

Grafana is accessible only from the local network at `http://grafana.lam-lab.cc` (requires DNS record or `/etc/hosts` entry pointing to cluster IP).

- **Default credentials**: admin / password (change immediately after first login)
- **Import dashboards**: Upload `dashboard.json` files via Grafana UI

## Adding a New Application

1. **Create base resources** in `labs/apps/base/<app>/`:
   - Create namespace, deployment, service, storage, and kustomization.yaml
   - Pin specific image versions (do NOT use `:latest` tags for determinism)

2. **Create environment overlay** in `labs/apps/home/<app>/`:
   - Create `tunnel.yaml` for Cloudflare tunnel deployment
   - Create `tunnel-token.yaml` with encrypted token (use SOPS)
   - Create `kustomization.yaml` referencing base and overlays

3. **Update environment kustomization**:
   - Add new app to `labs/apps/home/kustomization.yaml` resources list

4. **Encrypt secrets and commit**:
   ```bash
   sops --encrypt --in-place labs/apps/home/<app>/tunnel-token.yaml
   git add labs/apps/home/<app>/ labs/apps/base/<app>/
   git commit -m "feat: add <app> application"
   git push
   ```

5. **Flux will automatically deploy** within 10 minutes (or force reconciliation with `flux reconcile kustomization apps-home`)

## Security Considerations

- **Never commit unencrypted secrets** - Always run `sops --encrypt --in-place` before git push
- **Image pinning** - Use specific versions (e.g., `sissbruecker/linkding:1.44.0`) instead of `:latest`
- **Renovate automation** - Dependency updates are proposed automatically via pull requests
- **Cloudflare tunnels** - All external access goes through Cloudflare for DDoS protection and zero-trust access
- **Grafana access** - Monitoring is intentionally LAN-only with self-signed TLS certificates

## Troubleshooting

### Flux not syncing
```bash
# Check Flux system health
flux check

# View Flux events
kubectl get events -n flux-system --sort-by='.lastTimestamp'

# Check Git repository connection
flux get sources git

# Force reconciliation
flux reconcile source git flux-system
```

### SOPS decryption failures
- Ensure the `sops-age` secret exists in the cluster and contains the correct age private key
- Verify `.sops.yaml` matches the encryption pattern and age public key
- Check Flux logs for decryption errors: `flux logs --level=error`

### Cloudflare tunnel issues
```bash
# Check tunnel deployment
kubectl get pods -n <app-namespace> -l pod=cloudflared

# View tunnel logs
kubectl logs -n <app-namespace> -l pod=cloudflared

# Verify tunnel token secret
kubectl get secret tunnel-token -n <app-namespace>
```

### Application not accessible
1. Check pod status: `kubectl get pods -n <app-namespace>`
2. Check service: `kubectl get svc -n <app-namespace>`
3. Check Cloudflare tunnel logs
4. Verify DNS records in Cloudflare dashboard point to the tunnel
