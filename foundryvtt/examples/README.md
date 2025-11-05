# Example Configuration Files

This directory contains example configuration files for the Foundry VTT Helm chart.

## s3-config.json

Example S3 configuration file for storing Foundry assets in cloud storage.

### AWS S3

```json
{
  "endpoint": "https://s3.amazonaws.com",
  "accessKeyId": "YOUR_ACCESS_KEY_ID",
  "secretAccessKey": "YOUR_SECRET_ACCESS_KEY",
  "bucket": "your-foundry-bucket",
  "region": "us-east-1",
  "pathPrefix": "foundry-data"
}
```

### Cloudflare R2

```json
{
  "endpoint": "https://YOUR_ACCOUNT_ID.r2.cloudflarestorage.com",
  "accessKeyId": "YOUR_R2_ACCESS_KEY_ID",
  "secretAccessKey": "YOUR_R2_SECRET_ACCESS_KEY",
  "bucket": "your-foundry-bucket",
  "region": "auto"
}
```

### MinIO / Self-Hosted S3

```json
{
  "endpoint": "https://minio.example.com",
  "accessKeyId": "YOUR_MINIO_ACCESS_KEY",
  "secretAccessKey": "YOUR_MINIO_SECRET_KEY",
  "bucket": "foundry",
  "region": "us-east-1",
  "forcePathStyle": true
}
```

### DigitalOcean Spaces

```json
{
  "endpoint": "https://nyc3.digitaloceanspaces.com",
  "accessKeyId": "YOUR_SPACES_ACCESS_KEY",
  "secretAccessKey": "YOUR_SPACES_SECRET_KEY",
  "bucket": "your-space-name",
  "region": "nyc3"
}
```

### Backblaze B2

```json
{
  "endpoint": "https://s3.us-west-001.backblazeb2.com",
  "accessKeyId": "YOUR_B2_KEY_ID",
  "secretAccessKey": "YOUR_B2_APPLICATION_KEY",
  "bucket": "your-bucket-name",
  "region": "us-west-001"
}
```

## SSL/TLS Certificates

If you want Foundry to serve HTTPS directly (not recommended - use a reverse proxy instead), you need two files.

**Note:** These files will be mounted at:
- `/data/Config/certificates/cert` (certificate file)
- `/data/Config/certificates/key` (private key file)

This matches what Foundry VTT expects for SSL configuration.

### Certificate File

Your SSL certificate file (including any intermediate certificates). Can be in PEM format with any source filename (e.g., `cert.pem`, `fullchain.pem`, etc.).

```
-----BEGIN CERTIFICATE-----
MIIDXTCCAkWgAwIBAgIJAKJ...
-----END CERTIFICATE-----
```

### Private Key File

Your private key file. Can be in PEM format with any source filename (e.g., `key.pem`, `privkey.pem`, etc.).

```
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0B...
-----END PRIVATE KEY-----
```

## Usage

To use these configuration files in your Helm deployment, add them to your `values.yaml`:

```yaml
foundry:
  config:
    # S3 configuration
    s3: |
      {
        "endpoint": "https://s3.amazonaws.com",
        "accessKeyId": "YOUR_ACCESS_KEY_ID",
        "secretAccessKey": "YOUR_SECRET_ACCESS_KEY",
        "bucket": "your-foundry-bucket",
        "region": "us-east-1"
      }

    # SSL certificates
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

Or reference files:

```bash
helm install foundryvtt ./foundryvtt \
  --set-file foundry.config.s3=./s3-config.json \
  --set-file foundry.config.certificates.cert=./your-cert.pem \
  --set-file foundry.config.certificates.key=./your-key.pem
```

**Mounted paths:**
- Your `s3-config.json` → `/data/Config/s3-config.json`
- Your certificate file → `/data/Config/certificates/cert`
- Your key file → `/data/Config/certificates/key`

The source filenames can be anything (`.pem`, `.crt`, `.key`, etc.), but they will be mounted at the paths Foundry expects.

## Security Notes

1. **Never commit actual credentials to git** - these are examples only
2. S3 credentials and certificates should be treated as secrets
3. Use proper IAM permissions for S3 access (read/write to bucket only)
4. Consider using a reverse proxy (like nginx-ingress or Traefik) for SSL instead of embedding certificates in Foundry
5. Rotate credentials regularly
