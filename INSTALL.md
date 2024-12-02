# Installation Guide

## Prerequisites

### Hardware Requirements
- 3 nodes with at least:
  - 2 CPU cores per node
  - 2GB RAM per node
  - 20GB storage per node

### Software Requirements
- Linux OS on all nodes (tested with Ubuntu Server 22.04)
- Network interface `enp5s0` available on all nodes
- Root or sudo access

### Network Requirements
- Static IPs configured for all nodes:
  - Node 1: 192.168.68.21
  - Node 2: 192.168.68.22
  - Node 3: 192.168.68.23
- Reserved IPs:
  - 192.168.68.20 (control plane VIP)
  - 10.0.10.2 (load balancer IP)

## Installation Steps

### 1. Install Required Tools

```bash
# Install helmfile
curl -Lo ./helmfile https://github.com/helmfile/helmfile/releases/latest/download/helmfile_linux_amd64
chmod +x ./helmfile
sudo mv ./helmfile /usr/local/bin/

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/

# Install kustomize
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
sudo mv ./kustomize /usr/local/bin/
```

### 2. Install k3s

On the first node (192.168.68.21):
```bash
curl -sfL https://get.k3s.io | sh -s - server \
  --disable traefik \
  --disable servicelb \
  --tls-san 192.168.68.20 \
  --node-ip 192.168.68.21 \
  --advertise-address 192.168.68.21
```

Get the node token:
```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

On the other nodes (192.168.68.22 and 192.168.68.23):
```bash
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.68.21:6443 K3S_TOKEN=<NODE_TOKEN> sh -s - server \
  --disable traefik \
  --disable servicelb \
  --node-ip <NODE_IP> \
  --advertise-address <NODE_IP>
```

### 3. Deploy kube-vip

```bash
# Apply kube-vip configuration
git clone https://github.com/yourusername/homelab-test.git
cd homelab-test
kubectl apply -f cluster/base/kube-vip/
```

### 4. Deploy Remaining Components

```bash
# Deploy with helmfile
helmfile sync
```

### 5. Configure DNS

Add to your dnsmasq configuration:
```
address=/test.fanen.home/10.0.10.2
```

Or add to /etc/hosts:
```
10.0.10.2 test.fanen.home
```

## Verification

### 1. Check Cluster Status
```bash
kubectl get nodes
kubectl get pods --all-namespaces
```

### 2. Verify High Availability
```bash
# Test API server through VIP
kubectl --server https://192.168.68.20:6443 get nodes
```

### 3. Verify Web Access
```bash
# Check website accessibility
curl -H "Host: test.fanen.home" http://10.0.10.2
```

## Troubleshooting

### Common Issues

1. Control plane VIP not accessible:
   - Verify kube-vip pods are running
   - Check network interface name matches configuration
   - Ensure IP is not used by another device

2. Website not accessible:
   - Verify Traefik pods are running
   - Check LoadBalancer IP assignment
   - Verify DNS resolution

### Logs
```bash
# Check kube-vip logs
kubectl logs -n kube-system -l app=kube-vip

# Check Traefik logs
kubectl logs -n traefik -l app.kubernetes.io/name=traefik
```
