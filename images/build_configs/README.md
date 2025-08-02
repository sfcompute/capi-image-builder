# Build Configurations

This directory contains JSON configuration files for customizing CAPI image builds. These files allow you to override default settings and customize images for specific use cases without modifying the core Packer templates.

## Overview

The Image Builder tool uses Packer and Ansible to build images for Kubernetes CAPI providers. Each provider has its own format (e.g., AWS uses AMIs, vSphere uses OVAs). This directory contains configuration files that let you customize these builds.

For comprehensive documentation, see the [official Image Builder documentation](https://image-builder.sigs.k8s.io/capi/capi).

## Prerequisites

- Packer version >= 1.6.0
- Goss plugin for Packer version >= 1.2.0
- Ansible version >= 2.10.0

If needed binaries are not present, install them to `images/capi/.bin` with:

```bash
make deps
```

## How to Use Build Configuration Files

### Basic Usage

Use the `PACKER_VAR_FILES` environment variable to specify your configuration file:

```bash
PACKER_VAR_FILES=../build_configs/your-config.json make build-ami-ubuntu-2404
```

### Multiple Configuration Files

You can pass multiple configuration files, with later files taking precedence:

```bash
PACKER_VAR_FILES="../build_configs/base.json ../build_configs/custom.json" make build-ami-ubuntu-2404
```

### Example Build Commands

```bash
# Build Ubuntu 24.04 ARM64 AMI with custom configuration
PACKER_VAR_FILES=../build_configs/ubuntu-2404-arm64.json make build-ami-ubuntu-2404

# Build with multiple configuration files
PACKER_VAR_FILES="../build_configs/base.json ../build_configs/encrypted.json" make build-ami-ubuntu-2404

# Build for specific regions only
PACKER_VAR_FILES=../build_configs/limited-regions.json make build-ami-ubuntu-2404
```

## Available Configuration Files

| File                     | Description                      | Use Case                         |
| ------------------------ | -------------------------------- | -------------------------------- |
| `ubuntu-2404-arm64.json` | Ubuntu 24.04 ARM64 configuration | ARM64 builds, private AMIs       |
| `base.json`              | Base configuration template      | Starting point for custom builds |
| `encrypted.json`         | Encrypted volume configuration   | Security-focused builds          |
| `limited-regions.json`   | Limited region distribution      | Cost optimization                |

## Common Configuration Options

### AMI Sharing and Access

```json
{
  "ami_groups": "",
  "ami_users": "123456789012,987654321098",
  "snapshot_groups": "",
  "snapshot_users": "123456789012,987654321098"
}
```

- `ami_groups: ""` - Private AMI (default: `"all"` for public)
- `ami_users` - Specific AWS account IDs that can access the AMI
- `snapshot_groups` - Groups that can create volumes from snapshots
- `snapshot_users` - Users that can create volumes from snapshots

### Architecture and Components

```json
{
  "arch": "arm64",
  "containerd_arch": "arm64",
  "containerd_version": "1.7.28",
  "kubernetes_semver": "v1.33.1",
  "ecr_credential_provider": "true",
  "ecr_credential_provider_arch": "arm64"
}
```

### Region Distribution

```json
{
  "ami_regions": "us-east-1,us-west-2,eu-west-1"
}
```

### Security and Encryption

```json
{
  "encrypted": "true",
  "kms_key_id": "arn:aws:kms:region:account:key/key-id"
}
```

## Creating Custom Configurations

### Step 1: Start with a Base Configuration

Copy an existing configuration file or create a new one:

```json
{
  "arch": "amd64",
  "build_name": "my-custom-build",
  "kubernetes_semver": "v1.33.1",
  "ami_groups": "",
  "ami_regions": "us-east-1,us-west-2"
}
```

### Step 2: Add Custom Variables

Reference the [official documentation](https://image-builder.sigs.k8s.io/capi/capi#customization) for available variables:

```json
{
  "extra_rpms": "\"nfs-utils net-tools\"",
  "http_proxy": "http://proxy.corp.com",
  "additional_registry_images": "true",
  "additional_registry_images_list": "my-registry/my-image:v1.0"
}
```

### Step 3: Build with Your Configuration

```bash
PACKER_VAR_FILES=../build_configs/my-custom.json make build-ami-ubuntu-2404
```

## Troubleshooting

### Common Issues

1. **Block Public Access Error**

   ```
   Error modify AMI attributes: OperationNotPermitted: You can't publicly share this image because block public access for AMIs is enabled
   ```

   **Solution**: Set `"ami_groups": ""` in your configuration for private AMIs.

2. **Architecture Mismatch**

   ```
   Error: architecture mismatch
   ```

   **Solution**: Ensure `arch`, `containerd_arch`, and other architecture-related variables are consistent.

3. **Permission Errors**
   ```
   Error: Access Denied
   ```
   **Solution**: Verify AWS credentials and IAM permissions for the required operations.

### Debugging

Enable verbose output:

```bash
PACKER_LOG=1 PACKER_VAR_FILES=../build_configs/your-config.json make build-ami-ubuntu-2404
```

## Reference Documentation

- [Official Image Builder Documentation](https://image-builder.sigs.k8s.io/capi/capi)
- [Packer Amazon AMI Builder](https://www.packer.io/docs/builders/amazon.html)
- [CAPI Image Builder GitHub Repository](https://github.com/kubernetes-sigs/image-builder)

## Examples

### Private AMI with Custom Packages

```json
{
  "ami_groups": "",
  "snapshot_groups": "",
  "extra_rpms": "\"nfs-utils net-tools tcpdump\"",
  "kubernetes_semver": "v1.33.1",
  "ami_regions": "us-east-1,us-west-2"
}
```

### Encrypted AMI for Multiple Accounts

```json
{
  "encrypted": "true",
  "ami_users": "123456789012,987654321098",
  "snapshot_users": "123456789012,987654321098",
  "ami_groups": "",
  "snapshot_groups": ""
}
```

### ARM64 Build with ECR Credential Provider

```json
{
  "arch": "arm64",
  "containerd_arch": "arm64",
  "containerd_version": "1.7.28",
  "ecr_credential_provider": "true",
  "ecr_credential_provider_arch": "arm64",
  "ecr_credential_provider_version": "v1.31.1",
  "ecr_credential_provider_os": "linux"
}
```
