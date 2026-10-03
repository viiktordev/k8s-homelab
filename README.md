# Kubernetes Homelab

A bare-metal Kubernetes cluster built from scratch for hands-on experimentation with Kubernetes administration, networking, storage, observability, security, and workload orchestration.

The cluster is bootstrapped with `kubeadm` and runs across multiple physical machines in my homelab.

This repository contains the declarative configuration used to manage the cluster and serves as both a learning environment and a practical platform for self-hosted workloads.

## Architecture

```text id="q1mzfv"
                         Internet
                            |
                     Router / NAT
                            |
                      Local Network
                            |
            +---------------+---------------+
            |                               |
     Control Plane                     Worker Nodes
            |                               |
            +--------- Kubernetes ----------+
                            |
                         Cilium
                            |
                    Cluster Workloads
```

The cluster currently consists of:

- 1 control-plane node
- 2 worker nodes
- Bare-metal infrastructure
- Debian Linux
- Kubernetes bootstrapped with `kubeadm`
- `containerd` as the container runtime
- Cilium as the CNI
- CoreDNS for cluster DNS

## Repository Structure

```text id="e29rkl"
k8s-homelab/
├── apps/
│   └── teamspeak/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── pv.yaml
│       └── pvc.yaml
│
├── namespaces/
│   ├── production.yaml
│   └── sandbox.yaml
│
└── README.md
```

### `apps/`

Contains Kubernetes manifests for workloads deployed to the cluster.

Each application is isolated in its own directory and can contain its own documentation and Kubernetes resources.

### `namespaces/`

Contains the namespace definitions used to logically organize workloads.

Currently:

- `production` — stable workloads
- `sandbox` — experiments and temporary workloads

## Networking

The cluster uses **Cilium** as its Container Network Interface (CNI).

Cilium provides pod networking and connectivity between workloads running across the physical nodes.

Cluster networking has been validated using the Cilium connectivity test suite.

```bash id="4sr0q9"
cilium connectivity test
```

External services are currently exposed through Kubernetes Services and router-level NAT/port forwarding.

Future improvements will include dedicated load balancing and ingress infrastructure.

## Storage

Because the cluster runs on bare-metal infrastructure, there is no cloud-provided storage layer.

Persistent workloads currently use Kubernetes:

```text id="6k70sm"
Pod
 |
 v
PersistentVolumeClaim
 |
 v
PersistentVolume
 |
 v
Physical Storage
```

Local PersistentVolumes can use node affinity to ensure workloads are scheduled on the physical node containing their data.

This provides basic persistence while keeping the storage architecture simple and transparent.

A future iteration of the cluster may introduce dynamic provisioning and shared or distributed storage.

## Namespaces

Workloads are separated using Kubernetes namespaces.

```text id="t41s5g"
Cluster
├── production
│   └── Stable workloads
│
└── sandbox
    └── Experiments and testing
```

Additional namespaces will be introduced as infrastructure components such as monitoring and observability are added.

## Applications

Applications deployed to the cluster are stored under:

```text id="ljc2m4"
apps/
```

Each application is responsible for defining its own Kubernetes resources and documentation.

Currently deployed:

- TeamSpeak

Application-specific architecture and deployment instructions should be documented inside the corresponding application directory.

## Cluster Operations

Some useful commands for operating the cluster:

Check node status:

```bash id="z9g7mo"
kubectl get nodes -o wide
```

Check workloads across the cluster:

```bash id="s68v4v"
kubectl get pods -A -o wide
```

Check services:

```bash id="20fcdw"
kubectl get svc -A
```

Check persistent storage:

```bash id="9hiyxs"
kubectl get pv
kubectl get pvc -A
```

Check Cilium:

```bash id="sncjsl"
cilium status
```

Check cluster events:

```bash id="uxufmm"
kubectl get events -A --sort-by='.lastTimestamp'
```

## Current Stack

| Layer | Technology |
|---|---|
| Operating System | Debian Linux |
| Kubernetes | kubeadm |
| Container Runtime | containerd |
| Networking / CNI | Cilium |
| Cluster DNS | CoreDNS |
| Storage | Local PersistentVolumes |
| Workload Management | Kubernetes Deployments |
| Configuration | Kubernetes manifests |

## Roadmap

The cluster will evolve as new Kubernetes concepts and infrastructure components are introduced.

Planned improvements include:

- Dynamic storage provisioning
- Shared or distributed persistent storage
- MetalLB
- Ingress Controller
- TLS certificate management
- Prometheus
- Grafana
- Centralized logging
- Resource requests and limits
- Liveness and readiness probes
- Network Policies
- Secrets management
- Backup and disaster recovery
- GitOps
- Cluster security improvements

The repository structure will evolve alongside the cluster, with infrastructure components eventually separated from application workloads.

## Goals

The primary goal of this homelab is to gain practical experience operating Kubernetes outside of a managed cloud environment.

Building the cluster from the ground up provides hands-on experience with:

- Cluster bootstrapping
- Node management
- Container runtimes
- Pod networking
- DNS
- Scheduling
- Persistent storage
- Service exposure
- Troubleshooting
- Cluster operations

The environment is also used for hands-on preparation for the **Certified Kubernetes Administrator (CKA)** certification.

## Philosophy

Instead of abstracting infrastructure behind managed services, this homelab intentionally exposes the underlying Kubernetes components.

The goal is not only to deploy applications, but to understand how the cluster behaves when nodes fail, Pods are recreated, storage moves, networking breaks, and infrastructure changes.
