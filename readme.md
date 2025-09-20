# Isaac's Home Lab

## Prerequisites
- kubectl
- helm
- K3s
- flux

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

## Homarr
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

## Mealie
```bash
# navigate to the mealie directory
cd mealie

# apply all configuration files
kubectl apply -f namespace.yaml # create namespace
kubectl apply -f storage.yaml # persistent volume
kubectl apply -f deployment.yaml # deploy
kubectl apply -f service.yaml # service config
```

## Grafana-Prometheus Monitoring Stack
```bash
# navigate to monitoring directory
cd monitoring

# add the Helm chart repository and update it
helm repo add prometheus-community \
https://prometheus-community.github.io/helm-charts
helm repo update

# create a namespace for the monitoring stack
kubectl create namespace monitoring

# install
helm install prometheus-stack \
prometheus-community/kube-prometheus-stack \
-n monitoring

# upgrade with the custom values file
helm upgrade \
-f values.yaml \
prometheus-stack prometheus-community/kube-prometheus-stack

# apply the load balancer configuration
kubectl apply -f loadbalancer.yaml
```

Import the dashboard by uploading the `dashboard.json`.