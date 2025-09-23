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
kube/
├── clusters/
│   └── prod/
│       └── kustomization.yaml
├── apps/
│   ├── nginx/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── redis/
│       └── helmrelease.yaml
└── infrastructure/
    └── monitoring/
        └── kustomization.yaml
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

## Managing Secrets
In this lab, secrets are encrypted using age. \
Read the [docs](https://fluxcd.io/flux/guides/mozilla-sops/) for more details. In particular, pay attention to the following sections:
- Configure the Git directory for encryption 
- Encrypting secrets using age 

Note: 
1. The `.sops.yaml` is usually placed at the root directory for consistency. Add specific `.sops.yaml` files in subdirectories only if those apps need different encryption keys or rules.
2. ⚠️ Remember to encrypt the secret files before pushing to repository.

## Apps
### [Linkding](https://github.com/sissbruecker/linkding)
A simple, self-hosted bookmark manager. 
#### Key Features
- Save and organize bookmarks with tags  
- Full-text search across saved bookmarks  
- Minimal and fast web interface  
- Import and export bookmarks (Netscape HTML format)  
- REST API for integrations and automation  
- Designed to be lightweight and easy to self-host (Docker support included) 

### [Mealie](https://mealie.io/)

Mealie is an open-source, self-hosted recipe manager and meal planner.  
It lets you save recipes, plan meals, and generate shopping lists - all in one place.  

- Import or create recipes  
- Plan meals for the week  
- Auto-generate grocery lists  
- Multi-user and smart home integrations  

### Homarr
```bash
# navigate to homarr directory
cd homarr

# add the Helm chart repository and update it
helm repo add homarr-labs https://homarr-labs.github.io/charts/
helm repo update

# create a namespace for homarr
kubectl create namespace homarr 

# install Homarr in the homarr namespace
helm install homarr homarr-labs/homarr -n homarr

# apply persistent volume configuration
kubectl apply -f storage.yaml

# apply load balancer configuration
kubectl apply -f loadbalancer.yaml

# upgrade Homarr with the custom values file
helm upgrade -f values.yaml \
homarr homarr-labs/homarr \
-n homarr
```


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
