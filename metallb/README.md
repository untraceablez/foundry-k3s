# MetalLB Configuration for K3s

This directory contains configuration files for MetalLB, which provides LoadBalancer capabilities for bare-metal Kubernetes clusters.

## Installation

### Install MetalLB

```bash
# Install MetalLB using the official manifest
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.3/config/manifests/metallb-native.yaml
```

Wait for MetalLB to be ready:

```bash
kubectl wait --namespace metallb-system \
  --for=condition=ready pod \
  --selector=app=metallb \
  --timeout=90s
```

### Configure MetalLB

After MetalLB is installed, apply the IP address pool and L2 advertisement configuration:

```bash
kubectl apply -f ipaddresspool.yaml
kubectl apply -f l2advertisement.yaml
```

## Configuration Files

### ipaddresspool.yaml

Defines the range of IP addresses that MetalLB can assign to LoadBalancer services. You must edit this file to match your LAN's IP address range.

### l2advertisement.yaml

Configures MetalLB to advertise the LoadBalancer IPs using Layer 2 mode (ARP/NDP).

## Customization

Before deploying, edit `ipaddresspool.yaml` to set your IP address range:

```yaml
spec:
  addresses:
  - 192.168.1.100-192.168.1.110  # Change this to match your network
```

Make sure the IP range:
1. Is within your LAN subnet
2. Does not conflict with your DHCP server's range
3. Has enough addresses for your services

## Verification

Check that MetalLB is running:

```bash
kubectl get pods -n metallb-system
```

Verify the IP address pool:

```bash
kubectl get ipaddresspool -n metallb-system
```

Verify L2 advertisement:

```bash
kubectl get l2advertisement -n metallb-system
```

## Usage with Foundry VTT

Once MetalLB is configured, the Foundry VTT Helm chart will automatically request a LoadBalancer IP. You can specify a particular IP by setting:

```yaml
service:
  loadBalancerIP: "192.168.1.100"  # Must be within the IP pool range
```

Or use MetalLB annotations to request a specific IP:

```yaml
service:
  annotations:
    metallb.universe.tf/loadBalancerIPs: "192.168.1.100"
```
