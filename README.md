# Homelab Test Infrastructure

This repository contains the Infrastructure as Code (IaC) for a k3s cluster deployment with the following components:

- kube-vip for control plane high availability
- Traefik as the ingress controller
- Test website deployment

## Prerequisites

- k3s cluster with 3 nodes
- helmfile
- kubectl
- kustomize

## Network Configuration

- Node IPs: 192.168.68.21-23
- Control Plane VIP: 192.168.68.20
- LoadBalancer IP: 10.0.10.2

## Getting Started

1. Clone this repository
2. Install prerequisites
3. Apply the configurations using helmfile

```bash
helmfile sync
```