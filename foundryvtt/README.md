# Foundry VTT Helm Chart

A Helm chart for deploying [Foundry Virtual Tabletop](https://foundryvtt.com) on Kubernetes using the [felddy/foundryvtt-docker](https://github.com/felddy/foundryvtt-docker) container image.

## Version

- Chart Version: 0.1.0
- Foundry VTT Version: 13.350

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- MetalLB or another LoadBalancer provider installed in your cluster
- A valid Foundry VTT license
- Persistent volume provisioner support in the cluster

## Installing MetalLB

This chart assumes MetalLB is already installed. If you haven't installed it yet, see the [MetalLB configuration guide](../metallb/README.md) in this repository.

## Getting Your Foundry Release URL

Before installing this chart, you **MUST** obtain a presigned release URL from Foundry VTT:

1. Log in to your account at [https://foundryvtt.com](https://foundryvtt.com)
2. Navigate to your profile: [https://foundryvtt.com/me/licenses](https://foundryvtt.com/me/licenses)
3. Find your purchased license
4. Click on your operating system (Linux/NodeJS)
5. Right-click the "Timed URL" button and copy the link address
6. This is your `FOUNDRY_RELEASE_URL` - it will look like:
   ```
   https://foundryvtt.s3.amazonaws.com/releases/13.350/FoundryVTT-13.350.zip?AWSAccessKeyId=...
   ```

**Important:** This URL is time-limited and must be obtained fresh when installing or updating.

## Quick Start

### 1. Create a values override file

Create a file named `my-values.yaml`:

```yaml
# REQUIRED: Your presigned Foundry release URL
foundry:
  releaseUrl: "https://foundryvtt.s3.amazonaws.com/releases/13.350/FoundryVTT-13.350.zip?AWSAccessKeyId=..."

  # Set a strong admin key
  admin:
    key: "your-secure-admin-key-here"

# Configure your LAN IP address
service:
  annotations:
    metallb.universe.tf/loadBalancerIPs: "192.168.1.100"
  # OR use:
  # loadBalancerIP: "192.168.1.100"
```

### 2. Install the chart

```bash
# Install the chart
helm install foundryvtt ./foundryvtt -f my-values.yaml

# Or if you want to specify a namespace
helm install foundryvtt ./foundryvtt -f my-values.yaml --namespace gaming --create-namespace
```

### 3. Check the deployment

```bash
# Check if the pods are running
kubectl get pods

# Check the service and get the external IP
kubectl get svc

# Check logs if needed
kubectl logs -l app.kubernetes.io/name=foundryvtt
```

### 4. Access Foundry VTT

Once deployed, access Foundry VTT at:

```
http://<your-configured-ip>:30000
```

For example: `http://192.168.1.100:30000`

## Configuration

### Required Configuration

| Parameter | Description | Example |
|-----------|-------------|---------|
| `foundry.releaseUrl` | Presigned URL from Foundry profile | See "Getting Your Foundry Release URL" above |
| `foundry.admin.key` | Admin password for Foundry VTT | `"my-secure-password"` |

### Service Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Kubernetes service type | `LoadBalancer` |
| `service.port` | Service port | `30000` |
| `service.annotations` | Service annotations for MetalLB | `{}` |
| `service.loadBalancerIP` | Specific IP to request | `""` |

**Setting a specific IP address:**

Method 1 - Using annotations (recommended):
```yaml
service:
  annotations:
    metallb.universe.tf/loadBalancerIPs: "192.168.1.100"
```

Method 2 - Using loadBalancerIP:
```yaml
service:
  loadBalancerIP: "192.168.1.100"
```

### Foundry Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `foundry.version` | Foundry version to install | `"13.350"` |
| `foundry.world` | Specific world to launch | `""` |
| `foundry.hostname` | Custom hostname for invite links | `""` |
| `foundry.licenseKey` | Specific license key | `""` |
| `foundry.preserveConfig` | Preserve GUI config changes | `false` |

### Admin Password Management

The admin password behavior depends on your configuration:

**Default Configuration (Recommended):**
```yaml
foundry:
  preserveConfig: true
  admin:
    key: "your-initial-admin-key"
    setAdminKey: false  # Don't reset password on restart
```

With this setup:
- ✅ Initial admin key is set on first deployment
- ✅ You can change the password via the Foundry web interface
- ✅ Password changes persist across pod restarts
- ✅ Configuration changes made in the GUI are preserved

**Alternative: Reset Password on Every Restart**
```yaml
foundry:
  preserveConfig: false
  admin:
    key: "your-admin-key"
    setAdminKey: true  # Force reset on every restart
```

With this setup:
- ⚠️ Admin key is reset on every pod restart
- ⚠️ GUI configuration changes are lost on restart
- ⚠️ You cannot change the password via the web interface

| Parameter | Description | Default |
|-----------|-------------|---------|
| `foundry.admin.key` | Admin password for Foundry | `"changeme-admin-key"` |
| `foundry.admin.setAdminKey` | Force admin key on restart | `false` |
| `foundry.preserveConfig` | Preserve GUI changes | `true` |

**Troubleshooting Password Issues:**

If you can't change your password via the web interface:
1. Ensure `preserveConfig: true` in your values.yaml
2. Ensure `setAdminKey: false` in your values.yaml
3. Upgrade the deployment: `helm upgrade foundryvtt ./foundryvtt`
4. The pod will restart without forcing the admin key
5. You can now change the password in Foundry's web interface

### Alternative Authentication

Instead of using `releaseUrl`, you can use your Foundry account credentials:

```yaml
foundry:
  auth:
    username: "your-foundryvtt-username"
    password: "your-foundryvtt-password"
  version: "13.350"
```

### Proxy Configuration

If running behind a reverse proxy:

```yaml
foundry:
  proxy:
    ssl: true
    port: "443"
```

### Configuration Files (S3 & Certificates)

You can optionally configure S3 storage for assets and/or SSL certificates. These files will be mounted at the paths where Foundry expects them:
- S3 config: `/data/Config/s3-config.json`
- SSL certificate: `/data/Config/certificates/cert`
- SSL private key: `/data/Config/certificates/key`

#### S3 Configuration

To use S3-compatible storage for Foundry assets:

```yaml
foundry:
  config:
    s3: |
      {
        "endpoint": "https://s3.amazonaws.com",
        "accessKeyId": "YOUR_ACCESS_KEY_ID",
        "secretAccessKey": "YOUR_SECRET_ACCESS_KEY",
        "bucket": "your-foundry-bucket",
        "region": "us-east-1"
      }
```

Supported S3-compatible services:
- AWS S3
- Cloudflare R2
- DigitalOcean Spaces
- Backblaze B2
- MinIO
- Any S3-compatible storage

See [examples/README.md](examples/README.md) for provider-specific examples.

#### SSL/TLS Certificates

To enable HTTPS directly in Foundry (not recommended - use a reverse proxy instead):

```yaml
foundry:
  config:
    certificates:
      cert: |
        -----BEGIN CERTIFICATE-----
        MIIDXTCCAkWgAwIBAgIJAKJ...
        -----END CERTIFICATE-----
      key: |
        -----BEGIN PRIVATE KEY-----
        MIIEvQIBADANBgkqhkiG9w0B...
        -----END PRIVATE KEY-----
```

Or load from files:

```bash
helm install foundryvtt ./foundryvtt \
  --set-file foundry.config.s3=./s3-config.json \
  --set-file foundry.config.certificates.cert=./your-cert.pem \
  --set-file foundry.config.certificates.key=./your-key.pem
```

**Note:** The certificate and key files will be mounted to `/data/Config/certificates/cert` and `/data/Config/certificates/key` respectively (without file extensions), which is what Foundry expects.

### Persistence Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.size` | PVC size | `10Gi` |
| `persistence.storageClass` | Storage class | `""` (default) |
| `persistence.accessMode` | Access mode | `ReadWriteOnce` |
| `persistence.existingClaim` | Use existing PVC | `""` |

### Resource Configuration

```yaml
resources:
  limits:
    cpu: 2000m
    memory: 2Gi
  requests:
    cpu: 500m
    memory: 512Mi
```

### Advanced Environment Variables

Add custom environment variables:

```yaml
extraEnv:
  - name: FOUNDRY_MINIFY_STATIC_FILES
    value: "true"
  - name: CONTAINER_VERBOSE
    value: "true"
  - name: TIMEZONE
    value: "America/New_York"
```

## Complete Example

Here's a complete `my-values.yaml` example:

```yaml
# Image configuration
image:
  repository: felddy/foundryvtt
  tag: "13.350"
  pullPolicy: IfNotPresent

# Foundry configuration
foundry:
  # REQUIRED: Get this from https://foundryvtt.com/me/licenses
  releaseUrl: "https://foundryvtt.s3.amazonaws.com/releases/13.350/FoundryVTT-13.350.zip?AWSAccessKeyId=..."

  # Admin credentials
  admin:
    key: "my-super-secure-admin-password"

  # Version
  version: "13.350"

  # Optional: Auto-launch a specific world
  world: "my-campaign-world"

  # Optional: Custom hostname for invitation links
  hostname: "foundry.mylan.local"

  # Preserve config changes made in the GUI
  preserveConfig: true

# Service configuration - expose on LAN
service:
  type: LoadBalancer
  port: 30000
  annotations:
    metallb.universe.tf/loadBalancerIPs: "192.168.1.100"

# Persistent storage
persistence:
  enabled: true
  size: 20Gi
  # Optional: Use a specific storage class
  # storageClass: "local-path"

# Resource limits
resources:
  limits:
    cpu: 2000m
    memory: 2Gi
  requests:
    cpu: 500m
    memory: 512Mi

# Additional environment variables
extraEnv:
  - name: FOUNDRY_MINIFY_STATIC_FILES
    value: "true"
  - name: TIMEZONE
    value: "America/New_York"
```

## Upgrading

To upgrade the chart:

```bash
# Get a fresh release URL from https://foundryvtt.com/me/licenses
# Update your my-values.yaml with the new URL and version

helm upgrade foundryvtt ./foundryvtt -f my-values.yaml
```

## Uninstalling

```bash
helm uninstall foundryvtt

# If you want to delete the PVC as well:
kubectl delete pvc foundryvtt
```

## Troubleshooting

### Pod won't start

Check the logs:
```bash
kubectl logs -l app.kubernetes.io/name=foundryvtt
```

Common issues:
1. **Missing or expired `releaseUrl`**: Get a fresh URL from your Foundry profile
2. **PVC not binding**: Check your storage provisioner is working
3. **MetalLB not assigning IP**: Verify MetalLB is installed and configured

### Can't access Foundry

1. Check the service has an external IP:
   ```bash
   kubectl get svc foundryvtt
   ```

2. Verify the IP is in your MetalLB pool range

3. Check firewall rules on your k3s nodes

4. Verify the pod is running:
   ```bash
   kubectl get pods -l app.kubernetes.io/name=foundryvtt
   ```

### Configuration not persisting

Set `foundry.preserveConfig: true` in your values to preserve changes made in the Foundry GUI between container restarts.

### Performance issues

Increase resource limits:
```yaml
resources:
  limits:
    cpu: 4000m
    memory: 4Gi
  requests:
    cpu: 1000m
    memory: 1Gi
```

## Security Considerations

1. **Store secrets securely**: Consider using external secret management (e.g., sealed-secrets, external-secrets-operator)
2. **Use strong admin keys**: Don't use default or weak passwords
3. **Network security**: Consider using NetworkPolicies to restrict access
4. **Update regularly**: Keep the Foundry version up to date with security patches

## Contributing

Contributions are welcome! Please submit issues and pull requests to the repository.

## References

- [Foundry VTT Official Site](https://foundryvtt.com)
- [felddy/foundryvtt-docker GitHub](https://github.com/felddy/foundryvtt-docker)
- [MetalLB Documentation](https://metallb.universe.tf/)
- [Helm Documentation](https://helm.sh/docs/)

## License

This Helm chart is provided as-is. Foundry Virtual Tabletop is a licensed product - you must own a valid license to use it.
