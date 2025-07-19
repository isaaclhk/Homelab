# Kubernetes notes

## What is Kubernetes?

Kubernetes (often abbreviated as **K8s**) is an open-source platform designed to automate the deployment, scaling, and management of containerized applications. Originally developed by Google, it is now maintained by the Cloud Native Computing Foundation (CNCF).

### Key Features

- **Orchestration**: Automates the deployment and scaling of containers across a cluster.
- **Self-Healing**: Detects and replaces failed containers automatically.
- **Load Balancing and Service Discovery**: Routes traffic efficiently and assigns DNS/IP addresses.
- **Automated Rollouts and Rollbacks**: Updates applications safely with rollback capability.
- **Storage Orchestration**: Provisions and manages storage volumes as needed.

### Reasons to Use Kubernetes

- **Automation**: Minimizes manual intervention in managing containerized workloads.
- **Scalability and Efficiency**: Dynamically scales applications based on usage metrics.
- **High Availability**: Ensures services remain accessible through failover mechanisms.
- **Portability**: Supports deployment across diverse environments

## Cluster Architecture

A Kubernetes cluster consists of two main components:

### 1. **Control Plane (Master Components)**

Responsible for managing the overall state and operations of the cluster.

- **API Server (`kube-apiserver`)**: Entry point for all cluster operations; exposes the Kubernetes API.
- **Controller Manager (`kube-controller-manager`)**: Monitors the cluster and ensures desired states (e.g., replicating pods).
- **Scheduler (`kube-scheduler`)**: Assigns workloads (pods) to available worker nodes based on resource availability and constraints.
- **etcd**: A distributed key-value store that holds configuration data and cluster state.

### 2. **Worker Nodes**

Run the actual application workloads (containers).

- **Kubelet**: Agent that runs on each node; communicates with the control plane and ensures containers are running.
- **Kube Proxy**: Handles networking and forwards traffic to the appropriate container or service.
- **Container Runtime**: Software that runs containers (e.g., containerd, CRI-O, Docker).

### Additional Concepts

- **Pods**: The smallest deployable unit; a group of one or more containers sharing storage and network resources.
- **Services**: Abstracts access to a set of pods; enables load balancing and service discovery.
- **Namespaces**: Provide logical separation of cluster resources.


## Setup
### Download Rancher Desktop:
https://rancherdesktop.io

### configuration 
**For mac users:** \
.zshrc is the configuration file that sets up your shell environment.\
From this file, you can configure shortcuts for kubernetes commands for convenience..
```bash
vim ~./zshrc
```

```bash
### MANAGED BY RANCHER DESKTOP START (DO NOT EDIT)
export PATH="/Users/your_username/.rd/bin:$PATH"

# Initialize Zsh completion system
autoload -Uz compinit
compinit

# Alias for kubectl
alias k='kubectl'

# Enable kubectl autocompletion
source <(kubectl completion zsh)
```

## References
1. https://kubernetes.io/docs/home/
