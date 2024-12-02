# Installation Guide

[Previous sections remain unchanged...]

### 5. Configure DNS

You have several options for DNS configuration:

#### Option 1: Using dnsmasq (Recommended for Production)

Add to your dnsmasq configuration (/etc/dnsmasq.conf or appropriate config directory):
```
# Option A: Single domain
address=/test.fanen.dk/10.0.10.2

# Option B: Wildcard for all *.fanen.dk subdomains
address=/.fanen.dk/10.0.10.2
```

Restart dnsmasq after making changes:
```bash
sudo systemctl restart dnsmasq
```

#### Option 2: Using nip.io for Quick Testing

If you don't want to configure DNS, you can use nip.io for immediate testing. Update your ingress host to:
```yaml
host: test-10-0-10-2.nip.io
```

This will automatically resolve to 10.0.10.2 without any DNS configuration.

#### Option 3: Local hosts file

Add to /etc/hosts:
```
10.0.10.2 test.fanen.dk
```

### 6. Update Application Ingress

Depending on your chosen DNS option, update the IngressRoute in `cluster/base/apps/test-website/ingress.yaml`:

```yaml
# For dnsmasq or /etc/hosts
spec:
  routes:
    - match: Host(`test.fanen.dk`)

# OR for nip.io
spec:
  routes:
    - match: Host(`test-10-0-10-2.nip.io`)
```

Apply the changes:
```bash
kubectl apply -f cluster/base/apps/test-website/ingress.yaml
```

[Rest of the document remains unchanged...]

## Testing DNS Resolution

Test your DNS configuration:

```bash
# For dnsmasq setup
dig test.fanen.dk

# For nip.io
dig test-10-0-10-2.nip.io

# Test web access
curl -I http://test.fanen.dk  # or your chosen domain
```

[Previous troubleshooting section remains...]