# Foundry VTT K3s Deployment

This repository contains Helm charts and configuration for deploying [Foundry Virtual Tabletop](https://www.foundryvtt.com/) on K3s using the [felddy/foundryvtt-docker](https://github.com/felddy/foundryvtt-docker) container image.

## Repository Structure

```
foundry-k3s/
├── foundryvtt/        # Helm chart for Foundry VTT (v0.1.0)
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── templates/
│   └── README.md      # Detailed installation and configuration guide
└── metallb/           # MetalLB LoadBalancer configuration
    ├── ipaddresspool.yaml
    ├── l2advertisement.yaml
    └── README.md      # MetalLB setup guide
```

## Quick Start

### 1. Install MetalLB (if not already installed)

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.3/config/manifests/metallb-native.yaml

# Wait for MetalLB to be ready
kubectl wait --namespace metallb-system \
  --for=condition=ready pod \
  --selector=app=metallb \
  --timeout=90s

# Configure MetalLB with your IP range (edit metallb/ipaddresspool.yaml first!)
kubectl apply -f metallb/ipaddresspool.yaml
kubectl apply -f metallb/l2advertisement.yaml
```

See [metallb/README.md](metallb/README.md) for detailed MetalLB configuration.

### 2. Get Your Foundry Release URL

1. Log in to [https://foundryvtt.com/me/licenses](https://foundryvtt.com/me/licenses)
2. Find your license and click your operating system (Linux/NodeJS)
3. Right-click "Timed URL" and copy the link address

### 3. Deploy Foundry VTT

Create `my-values.yaml`:

```yaml
foundry:
  # REQUIRED: Your presigned release URL from step 2
  releaseUrl: "https://foundryvtt.s3.amazonaws.com/releases/13.350/FoundryVTT-13.350.zip?AWSAccessKeyId=..."

  admin:
    key: "your-secure-admin-password"

service:
  annotations:
    metallb.universe.tf/loadBalancerIPs: "192.168.1.100"  # Your LAN IP
```

Install the chart:

```bash
helm install foundryvtt ./foundryvtt -f my-values.yaml
```

### 4. Access Foundry

Once deployed, access Foundry at: `http://<your-lan-ip>:30000`

## Documentation

- **[Foundry VTT Helm Chart Documentation](foundryvtt/README.md)** - Complete installation and configuration guide
- **[MetalLB Configuration Guide](metallb/README.md)** - LoadBalancer setup for K3s

## Features

- Deploy Foundry VTT on K3s with a single Helm command
- Exposes Foundry on your LAN using MetalLB LoadBalancer
- Persistent storage for Foundry data, worlds, and modules
- Kubernetes secrets for secure credential management
- Configurable via Helm values for easy customization
- Support for all felddy/foundryvtt-docker environment variables

## Requirements

- Kubernetes 1.19+ (K3s recommended)
- Helm 3.0+
- MetalLB or another LoadBalancer provider
- Valid Foundry VTT license
- Persistent volume support in your cluster

## Version Information

- **Chart Version**: 0.1.0
- **Default Foundry Version**: 13.350
- **Container Image**: felddy/foundryvtt

## Contributing

This repository is designed to eventually include:
- CI/CD workflows for chart validation
- Automated testing against upstream image updates
- Chart versioning and releases

Contributions are welcome!

## License

This Helm chart is provided as-is. Foundry Virtual Tabletop is a licensed product - you must own a valid license to use it.

## References

- [Foundry VTT Official Site](https://foundryvtt.com)
- [felddy/foundryvtt-docker](https://github.com/felddy/foundryvtt-docker)
- [MetalLB Documentation](https://metallb.universe.tf/)
- [K3s Documentation](https://docs.k3s.io/)


