# Server Configs

Production-ready server configurations including cloud-init templates, security hardening, and performance tuning for Debian/Ubuntu servers.

## Overview

This repository contains battle-tested configurations and templates for deploying and maintaining Linux servers. Whether you're spinning up a single VPS or managing a fleet of servers, these configs provide a solid foundation with security and performance in mind.

## Repository Structure

```
server-configs/
├── cloud-init/           # Cloud-init templates for various use cases
│   ├── standard.yaml     # Balanced config for general use
│   ├── performance.yaml  # Optimized for high-performance workloads
│   └── standard-template.yaml  # Well-commented template with placeholders
├── scripts/              # Setup and maintenance scripts
├── sysctl/               # Kernel parameter tuning
├── ssh/                  # SSH server configuration templates
├── security/             # Security hardening configs
└── docs/                 # Additional documentation
```

## Quick Start

### Cloud-Init (New Server Deployment)

1. Copy the appropriate template from `cloud-init/`
2. Replace placeholder values:
   - `<YOUR_USERNAME>` - Your desired username
   - `<YOUR_SSH_PUBLIC_KEY>` - Your SSH public key
3. Paste into your cloud provider's "User Data" or "Cloud-Init" field
4. Launch your server

```bash
# Example: Copy and customize the standard template
cp cloud-init/standard-template.yaml my-server.yaml
# Edit my-server.yaml with your values
```

## Features

### Security Hardening
- **SSH hardening**: Key-only authentication, root login disabled
- **Automatic security updates**: Via unattended-upgrades
- **Strict file permissions**: Secure defaults for sensitive files
- **Firewall templates**: UFW configurations included

### Performance Optimizations
- **Dynamic swap**: Automatically sized based on available RAM
- **SSD/NVMe tuning**: Optimized I/O scheduler, noatime mount option
- **Kernel parameters**: Tuned sysctl settings for modern workloads
- **Memory management**: Optimized swappiness and dirty ratios

### Maintenance
- **Scheduled reboots**: Optional weekly maintenance window
- **Log rotation**: Sensible defaults for log management
- **Cleanup scripts**: Automated housekeeping tasks

## Cloud-Init Templates

| Template | Use Case | Features |
|----------|----------|----------|
| `standard.yaml` | General purpose servers | Balanced security + performance |
| `performance.yaml` | High-traffic applications | Aggressive optimizations |
| `standard-template.yaml` | Starting point | All placeholders, well-commented |

## Supported Platforms

### Operating Systems
- Ubuntu 20.04 LTS, 22.04 LTS, 24.04 LTS
- Debian 11 (Bullseye), 12 (Bookworm)

### Cloud Providers
Tested and compatible with:
- Hetzner Cloud
- DigitalOcean
- Linode/Akamai
- Vultr
- AWS EC2
- Google Cloud Platform
- Azure
- Any provider supporting cloud-init

## Configuration Details

### SSH Configuration
```
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
StrictModes yes
```

### Swap Sizing Logic
| RAM | Swap Size |
|-----|-----------|
| ≤ 2GB | Equal to RAM |
| 2-8GB | Half of RAM |
| > 8GB | 4GB (capped) |

### SSD Optimizations
- `vm.swappiness=10` - Prefer RAM over swap
- `vm.dirty_ratio=10` - Flush writes at 10% memory
- `vm.dirty_background_ratio=5` - Background flush at 5%
- `noatime` mount option - Reduce unnecessary writes

## Usage Examples

### Basic Server Setup
```yaml
# In your cloud provider's user-data field:
#cloud-config
users:
  - name: deploy
    ssh_authorized_keys:
      - "ssh-ed25519 AAAA... your-key"
    sudo: ALL=(ALL) NOPASSWD:ALL
```

### Apply Sysctl Settings Manually
```bash
# Copy sysctl configuration
sudo cp sysctl/performance.conf /etc/sysctl.d/99-performance.conf
sudo sysctl -p /etc/sysctl.d/99-performance.conf
```

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Areas for Contribution
- Additional cloud-init templates
- Distribution-specific configs (RHEL, Alpine, etc.)
- Monitoring and alerting integrations
- Container/Kubernetes configurations
- Documentation improvements

## Security

For security concerns, please review our [Security Policy](SECURITY.md).

**Important**: Never commit sensitive data like private keys, passwords, or API tokens to this repository.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [cloud-init documentation](https://cloudinit.readthedocs.io/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Debian Administrator's Handbook](https://debian-handbook.info/)

---

**Disclaimer**: These configurations are provided as-is. Always review and test configurations in a non-production environment before deploying to production servers.
