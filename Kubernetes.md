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
![alt text](images/kubernetes-cluster-architecture.svg)
**Figure 1.** Architecture of a Kubernetes cluster

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

### Configuration 
**For mac users:** \
.zshrc is the configuration file that sets up your shell environment.\
From this file, you can configure shortcuts for kubernetes commands for convenience.
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

## Kubectl

`kubectl` is the primary command-line tool for interacting with Kubernetes clusters. It allows you to deploy applications, inspect and manage cluster resources, and view logs. With `kubectl`, you can:

- Create, update, and delete resources (e.g., pods, deployments, services)
- View the status of cluster components (`kubectl get pods`, `kubectl get nodes`)
- Debug issues using commands like `kubectl describe` and `kubectl logs`
- Apply configuration files (`kubectl apply -f <file.yaml>`)
- Execute commands inside containers (`kubectl exec -it <pod> -- <command>`)
- Manage namespaces, contexts, and cluster authentication

## Minikube

Minikube is a tool that lets you run Kubernetes locally. It creates a single-node cluster inside a virtual machine (VM) or container on your laptop. Minikube is ideal for:

- Learning Kubernetes concepts
- Developing and testing applications before deploying to a production cluster
- Experimenting with Kubernetes features without cloud resources

Key features of Minikube:

- Supports most Kubernetes features
- Easy installation and startup (`minikube start`)
- Provides a local Docker registry for building and testing images
- Integrates with kubectl for cluster management
- Allows you to enable Kubernetes add-ons (e.g., dashboard, ingress)

To get started, install Minikube and run:

```bash
minikube start
kubectl get nodes
```
## Infrastructure as code
Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure using machine-readable configuration files, rather than manual processes. In Kubernetes, IaC enables you to define cluster resources (pods, services, deployments, etc.) declaratively using YAML or JSON manifests.

### Benefits of IaC in Kubernetes

- **Consistency**: Ensures environments are reproducible and standardized.
- **Version Control**: Store infrastructure definitions in Git for tracking changes and collaboration.
- **Automation**: Integrate with CI/CD pipelines for automated deployments and updates.
- **Scalability**: Easily scale resources by updating configuration files.

Apply with:

```bash
kubectl apply -f deployment.yaml
```

## References
1. https://kubernetes.io/docs/home/
