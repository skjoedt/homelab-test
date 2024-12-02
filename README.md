# Homelab Kubernetes Infrastructure

This repository contains Infrastructure as Code (IaC) for deploying and managing a high-availability k3s cluster. The project uses modern IaC practices with helmfile and kustomize to maintain a clean, maintainable codebase.

## Project Overview

This infrastructure stack provides:

### High Availability Control Plane
- k3s cluster with 3 nodes
- kube-vip for control plane high availability
- Automatic failover for the Kubernetes API endpoint

### Load Balancing & Ingress
- Traefik as the ingress controller
- External load balancing for services
- HTTP and HTTPS support
- Traefik dashboard for monitoring and management

### Sample Application
- Example website deployment
- High availability configuration with multiple replicas
- Automatic load balancing
- DNS integration example

## Architecture

### Network Configuration
- Control Plane VIP: 192.168.68.20
- Node IPs: 192.168.68.21-23
- LoadBalancer IP: 10.0.10.2

### Component Structure
```
.
├── cluster/
│   ├── base/                    # Base cluster configurations
│   │   ├── kube-vip/           # HA control plane
│   │   ├── traefik/            # Ingress controller
│   │   └── apps/               # Application deployments
│   └── environments/           # Environment-specific configs
└── helmfile/                   # Helm release definitions
```

## Getting Started

Please see [INSTALL.md](INSTALL.md) for detailed installation and setup instructions.

## Contributing

1. Fork the repository
2. Create a new feature branch
3. Make your changes
4. Submit a pull request

## License

MIT License - See LICENSE file for details
