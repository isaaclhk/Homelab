# Isaac's Home Lab

## Prerequisites
- kubectl
- helm
- K3s

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
```

On the remote machine:
```bash
# Create kube directory if it doesn’t exist
mkdir -p ~/.kube

# Copy k3s.yaml from target machine to your local ~/.kube/config
# Replace <user> and <target-ip> with your values
scp <user>@<target-ip>:/etc/rancher/k3s/k3s.yaml ~/.kube/config

# Fix ownership and permissions
chown "$USER":"$USER" ~/.kube/config
chmod 600 ~/.kube/config
```

In `~/.kube/config`, update::
```vim
clusters:
- cluster:
    server: https://<target-ip>:6443
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