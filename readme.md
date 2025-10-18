# Isaac's Home Lab

## Prerequisites
- kubectl
- [helm](https://helm.sh/docs/)
- [K3s](https://docs.k3s.io/quick-start)
- [flux](https://fluxcd.io/flux/get-started/)
- [A cloudflare domain](https://www.cloudflare.com/products/registrar/)

Clone the repository:
```
git clone https://github.com/isaaclhk/Kubernetes.git
```

### K3s Setup

#### SSH
On the target machine:
```bash
# installation
sudo apt update
sudo apt install openssh-server

# enable
sudo systemctl enable ssh
sudo systemctl start ssh
```

On your remote machine:
```bash
# ssh
ssh username@<ip address>
```

#### K3s
Install K3s on the target machine:
```bash
curl -sfL https://get.k3s.io | sh -

sudo cp /etc/rancher/k3s/k3s.yaml .

exit
```

On the remote machine:
```bash
# Replace <user> and <target-ip> with your values
scp <user>@<target-ip>:/home/user/k3s.yaml .

mkdir -p ~/.kube
vim k3s.yaml # edit the IP
mv k3s.yaml .kube/config
```

## GitOps with Flux

### What is GitOps
GitOps is a way to manage Kubernetes clusters and applications using Git as the single source of truth.  
- Cluster configuration is stored in a Git repository  
- Any change to the repository (new deployment, config update, secret rotation) is automatically applied to the cluster  
- Pull requests become the deployment pipeline  

### Why Flux
[Flux](https://fluxcd.io/) is a GitOps operator for Kubernetes. It:  
- Watches your Git repository for changes  
- Applies those changes to your cluster  
- Ensures the cluster always matches what is declared in Git  

If someone manually changes the cluster, Flux will reconcile it back to match the Git state.

### How it Works
1. Install Flux into your cluster and connect it to your GitHub or GitLab repository  
2. Store Kubernetes manifests, Helm releases, or Kustomize overlays in the repository  
3. Flux continuously reconciles the cluster state with the repository  
4. Update the cluster by making changes in Git and merging pull requests  

### Repository Structure
```bash
labs
├── apps
│   ├── base
│   │   ├── audiobookshelf
│   │   ├── ghost
│   │   ├── homarr
│   │   ├── linkding
│   │   └── mealie
│   ├── ebf
│   │   └── ghost
│   └── home
│       ├── audiobookshelf
│       ├── homarr
│       ├── linkding
│       └── mealie
├── clusters
│   └── flux-system
├── infrastructure
│   ├── renovate
│   ├── longhorn
│   └── velero
└── monitoring
    ├── configs
    └── controllers
```
## Exposing Apps to the Internet via Cloudflare

### Overview
Cloudflare is a cloud-based service that provides security, performance, and reliability for websites and applications.  It acts as a reverse proxy, sitting between users and your server, to protect and accelerate traffic. 

A guide on how to expose a Kubernetes service to the public Internet using a remotely managed Cloudflare Tunnel is available [here](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/deployment-guides/kubernetes/).

### Key Services
- **Content Delivery Network (CDN)**  
  Delivers cached copies of your site from servers around the world for faster load times.

- **DDoS Protection**  
  Shields your site from distributed denial-of-service attacks.

- **Web Application Firewall (WAF)**  
  Filters malicious requests and blocks common attacks such as SQL injection or cross-site scripting.

- **DNS Management**  
  Provides fast and reliable domain name resolution.

- **SSL/TLS Encryption**  
  Automatically manages certificates to enable secure HTTPS connections.

- **Zero Trust and Access Control**  
  Protects internal applications and services without the need for a traditional VPN.

### How It Works
1. A visitor tries to access your website.  
2. The request is routed through Cloudflare’s global network.  
3. Cloudflare checks for threats, serves cached content if possible, and forwards safe traffic to your server.  
4. Responses from your server are optimized and cached before being sent back to the visitor.  

### Benefits
- Faster performance through global caching and optimization  
- Stronger security with built-in protection against attacks  
- Increased reliability with traffic routed around server outages  
- Simplified operations with managed DNS and automatic SSL  

## Security
### Encrypting and Decrypting Secrets
In this lab, secrets are encrypted using age. \
Read the [docs](https://fluxcd.io/flux/guides/mozilla-sops/) for more details. In particular, pay attention to the following sections:
- Configure the Git directory for encryption 
- Encrypting secrets using age 

Note: 
1. The `.sops.yaml` is usually placed at the root directory for consistency. Add specific `.sops.yaml` files in subdirectories only if those apps need different encryption keys or rules.
2. ⚠️ Remember to encrypt the secret files before pushing to repository.

### Self-signed TLS Certificate

A [self-signed TLS certificate](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs) is a digital certificate that is generated and signed by the same entity, rather than being issued by a trusted Certificate Authority (CA).  
It contains the same components as a CA-signed certificate (such as domain, public key, and validity period), but since it is not verified by a third party, browsers and clients will typically display a warning.

### Benefits

- **Encryption**: Enables HTTPS, ensuring that data exchanged between client and server is encrypted.  
- **Practical for Non-Production**: Useful in development, testing, and internal environments where trust warnings are acceptable.  
- **Cost-Effective**: Can be created quickly and free of charge without relying on an external CA.  

> **Note**: For production systems or public-facing services, a CA-signed certificate (e.g., from Let’s Encrypt) is strongly recommended to establish trust and prevent browser security warnings.


## Monitoring
### [Prometheus-Grafana Monitoring Stack](https://github.com/prometheus-community/helm-charts/tree/main)

A collection of Helm charts maintained by the Prometheus community, used to deploy Prometheus, Grafana, Alertmanager, and related exporters/tools in Kubernetes environments.  

#### Key Features
- Charts maintained by the Prometheus community on GitHub  
- Supports installation via Helm (`prometheus-community` repo)  
- Provides OCI artifacts for all charts  
- Cluster monitoring, alerting, and exporting metrics

To use Grafana:  
- Import dashboards by uploading the `dashboard.json` file.  
- Default login: **admin / password**  
- For security, change the default password immediately after your first login.
- Grafana is configured using ingress to only be accessible from the local area network.

#### Accessing Grafana
##### Option 1: Private DNS via Cloudflare (Recommended)
From your cloudflare dashboard -> click on your domain -> DNS -> Records \
Add a DNS A record with your desired name and IPv4 address. Set it to DNS only.


##### Option 2: Static Host Mapping
To make the hostname resolve to your cluster node IP, add an entry to your `/etc/hosts` file:

```bash
sudo vim /etc/hosts
```
Then append a line with your cluster node’s IP and chosen hostname, for example:
```vim
192.168.1.50 grafana.mydomain.com
```
After saving, you can access Grafana in your browser at:
```text
http://grafana.mydomain.com
```
## Infrastructure

### Distributed Block Storage via [Longhorn](https://longhorn.io/)

Longhorn is a cloud-native distributed block storage system for Kubernetes. It provides persistent volumes for stateful applications with built-in replication, snapshots, and backups.

In this lab, Longhorn is configured as the default StorageClass with a single replica (optimized for single-node homelab setups). All application PersistentVolumeClaims automatically use Longhorn for storage provisioning.

Key features:
- **Default StorageClass** - Automatic volume provisioning for new PVCs
- **Web UI** - Visual management of volumes, nodes, and backups
- **Snapshot Support** - Point-in-time volume snapshots for data protection

To access the Longhorn UI:
```bash
kubectl port-forward -n longhorn-system svc/longhorn-frontend 8080:80
# Open http://localhost:8080
```

### Disaster Recovery via [Velero](https://velero.io/)

Velero is a backup and disaster recovery tool for Kubernetes clusters. It backs up cluster resources and persistent volumes to object storage (S3-compatible), enabling full cluster restoration in case of data loss or cluster failure.

In this lab, Velero runs weekly backups to an S3 bucket with a 7-day retention policy (keeping only 1 backup at a time for cost optimization).

Configuration highlights:
- **Backup Schedule** – Weekly on Mondays at 2:00 AM
- **Retention** – 7 days (168 hours)
- **Storage Backend** – AWS S3-compatible bucket 
- **Backup Scope** – All namespaces and persistent volumes
- **Recovery Window** – Single backup retained for hardware failure scenarios

Common operations:
```bash
# List all backups
velero backup get

# Manually create a backup
velero backup create manual-backup-$(date +%Y%m%d-%H%M%S) --default-volumes-to-fs-backup --wait

# Restore from backup
velero restore create --from-backup <backup-name>
```

> **Important - Recovery Timing**: With a 7-day TTL and weekly schedule, only 1 backup exists at any time. This means:
> - **Between backup runs**: You may have 0 backups visible (normal behavior)
> - **After a crash lasting 6+ days**: The backup may be deleted immediately when Velero starts due to TTL expiration
>
> **If restoring after a multi-day outage**:
> 1. Scale down Velero **before** it reconciles: `kubectl scale deployment velero -n velero --replicas=0`
> 2. Restore your data: `velero restore create --from-backup <backup-name>`
> 3. Scale Velero back up: `kubectl scale deployment velero -n velero --replicas=1`
>
> **To avoid this issue entirely**: Consider increasing TTL to 14+ days for a safety buffer.

### Automatic Image Updates via [Renovate](https://docs.renovatebot.com/examples/self-hosting/)

Renovate is an automation tool designed to manage and update dependencies, including Docker images, Helm charts, and Kubernetes manifests. Rather than relying on the non-deterministic :latest tag, Renovate continuously monitors registries for new versions and generates merge or pull requests with proposed updates.

By integrating Renovate into your workflow, you gain:
- **Reproducibility** – Builds are consistent and traceable across environments
- **Transparency** – Updates are surfaced as reviewable requests before merging
- **Governance** – Policies, schedules, and automerge rules ensure controlled updates

## Other Apps
- [Linkding](https://github.com/sissbruecker/linkding)
- [Mealie](https://mealie.io/)
- [Homarr](https://homarr.dev/docs/getting-started/installation/helm/)
- [Audiobookshelf](https://www.audiobookshelf.org/)
- [Ghost](https://ghost.org/)
