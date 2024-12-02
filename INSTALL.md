# Installation Guide

## Prerequisites

### Local Machine Requirements (MacBook)
- macOS with Terminal access
- SSH access to all k3s nodes

### Remote Node Requirements
- 3 nodes with at least:
  - 2 CPU cores per node
  - 2GB RAM per node
  - 20GB storage per node
- Ubuntu Server 22.04 (recommended)
- Network interface `enp5s0` available on all nodes
- SSH enabled with sudo access
- Static IPs configured:
  - Node 1: 192.168.68.21
  - Node 2: 192.168.68.22
  - Node 3: 192.168.68.23
- Reserved IPs:
  - 192.168.68.20 (control plane VIP)
  - 10.0.10.2 (load balancer IP)

## Installation Steps

### 1. Install Required Tools on MacBook

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install required tools
brew install helm
brew install helmfile
brew install kubectl
brew install kustomize

# Install k3sup
brew install k3sup

# Verify installations
helmfile --version
kubectl version --client
kustomize version
k3sup version
```

### 2. Install k3s Cluster Using k3sup

```bash
# Create a directory for your kubeconfig
mkdir -p ~/.kube

# Install first master node
k3sup install \
  --ip 192.168.68.21 \
  --user ubuntu \
  --k3s-extra-args "--disable traefik --disable servicelb --node-ip 192.168.68.21 --advertise-address 192.168.68.21 --tls-san 192.168.68.20" \
  --k3s-version v1.27.7+k3s2 \
  --context homelab \
  --merge \
  --local-path ~/.kube/config

# Join second master node
k3sup join \
  --ip 192.168.68.22 \
  --user ubuntu \
  --server-ip 192.168.68.21 \
  --server-user ubuntu \
  --k3s-extra-args "--disable traefik --disable servicelb --node-ip 192.168.68.22 --advertise-address 192.168.68.22" \
  --k3s-version v1.27.7+k3s2 \
  --server

# Join third master node
k3sup join \
  --ip 192.168.68.23 \
  --user ubuntu \
  --server-ip 192.168.68.21 \
  --server-user ubuntu \
  --k3s-extra-args "--disable traefik --disable servicelb --node-ip 192.168.68.23 --advertise-address 192.168.68.23" \
  --k3s-version v1.27.7+k3s2 \
  --server
```

### 3. Set Up Local Repository

```bash
# Clone the repository
git clone https://github.com/yourusername/homelab-test.git
cd homelab-test

# Deploy kube-vip
kubectl apply -f cluster/base/kube-vip/

# Wait for kube-vip to be ready
kubectl -n kube-system wait pod -l name=kube-vip --for condition=ready --timeout=90s
```

### 4. Deploy Remaining Components

```bash
# Initialize helm repositories
helmfile repos

# Deploy with helmfile
helmfile sync
```

### 5. Configure Local DNS Resolution

You have several options for DNS configuration on your MacBook:

#### Option 1: Using dnsmasq via Homebrew

```bash
# Install dnsmasq
brew install dnsmasq

# Create config directory if it doesn't exist
mkdir -p $(brew --prefix)/etc/dnsmasq.d

# Add configuration for fanen.dk domain
echo 'address=/.fanen.dk/10.0.10.2' >> $(brew --prefix)/etc/dnsmasq.d/fanen.dk.conf

# Start dnsmasq service
sudo brew services start dnsmasq

# Configure macOS to use dnsmasq for .dk domains
sudo mkdir -p /etc/resolver
echo 'nameserver 127.0.0.1' | sudo tee /etc/resolver/dk
```

#### Option 2: Using nip.io for Quick Testing

No configuration needed, just update your ingress host in `cluster/base/apps/test-website/ingress.yaml` to:
```yaml
host: test-10-0-10-2.nip.io
```

#### Option 3: Local /etc/hosts file

```bash
echo '10.0.10.2 test.fanen.dk' | sudo tee -a /etc/hosts
```

### 6. Update Application Ingress

```bash
# Apply the updated ingress configuration
kubectl apply -f cluster/base/apps/test-website/ingress.yaml
```

## Verification

### 1. Check Cluster Status
```bash
# Verify nodes are ready
kubectl get nodes

# Check all pods
kubectl get pods -A
```

### 2. Verify High Availability
```bash
# Test API server through VIP
KUBECONFIG=~/.kube/config kubectl --server https://192.168.68.20:6443 get nodes
```

### 3. Test DNS Resolution

```bash
# For dnsmasq setup
dig test.fanen.dk @127.0.0.1

# For nip.io
dig test-10-0-10-2.nip.io

# Test web access
curl -I http://test.fanen.dk  # or your chosen domain
```

## Troubleshooting

### Common Issues

1. SSH Connection Issues:
   - Verify SSH key is properly set up
   - Check if you can SSH manually to each node
   - Ensure correct username is used

2. Control plane VIP not accessible:
   - Verify kube-vip pods are running
   - Check network interface name matches configuration
   - Ensure IP is not used by another device

3. DNS resolution not working:
   - For dnsmasq: check if service is running `brew services list`
   - Verify resolver configuration: `cat /etc/resolver/dk`
   - Try flushing DNS cache: `sudo killall -HUP mDNSResponder`

4. Website not accessible:
   - Verify Traefik pods are running
   - Check LoadBalancer IP assignment
   - Verify DNS resolution

### Logs and Debugging
```bash
# Check kube-vip logs
kubectl logs -n kube-system -l app=kube-vip

# Check Traefik logs
kubectl logs -n traefik -l app.kubernetes.io/name=traefik

# Test DNS resolution
dig test.fanen.dk @127.0.0.1
scutil --dns

# Check dnsmasq configuration
cat $(brew --prefix)/etc/dnsmasq.d/fanen.dk.conf
```