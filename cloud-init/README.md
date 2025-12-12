# Cloud-Init Templates

This directory contains cloud-init configuration templates for automated server provisioning.

## What is Cloud-Init?

[Cloud-init](https://cloudinit.readthedocs.io/) is the industry standard for cross-platform cloud instance initialization. It runs during initial boot and configures:

- Users and SSH keys
- Package installation
- File creation
- Custom scripts
- And much more

## Templates

| File | Description | Use Case |
|------|-------------|----------|
| `standard-template.yaml` | Well-commented template with placeholders | Starting point for customization |
| `standard.yaml` | Balanced security and performance | General purpose servers |
| `performance.yaml` | Optimized for high-load workloads | Web servers, databases |
| `minimal.yaml` | Bare essentials only | Containers, ephemeral instances |

## Usage

### 1. Choose a Template

Select the template that best matches your needs.

### 2. Customize

Replace placeholder values:

```yaml
# Replace these:
<YOUR_USERNAME>        # Your desired username
<YOUR_SSH_PUBLIC_KEY>  # Your public SSH key (ssh-ed25519 AAAA... or ssh-rsa AAAA...)
```

### 3. Deploy

**Hetzner Cloud:**
```
Server Creation → Cloud config → Paste YAML
```

**DigitalOcean:**
```
Create Droplet → Advanced Options → User data → Paste YAML
```

**AWS EC2:**
```
Launch Instance → Advanced Details → User data → Paste YAML
```

**Vultr:**
```
Deploy Server → Server Settings → Cloud-Init User-Data → Paste YAML
```

### 4. Verify

After the server boots, verify cloud-init completed:

```bash
# Check status
cloud-init status

# View logs
sudo cat /var/log/cloud-init-output.log

# Check for errors
sudo cloud-init analyze show
```

## Creating Custom Templates

Start with `standard-template.yaml` and customize:

```yaml
#cloud-config

# Users
users:
  - name: your-username
    ssh_authorized_keys:
      - "ssh-ed25519 AAAA..."
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash

# Packages to install
packages:
  - htop
  - vim
  - curl

# Commands to run
runcmd:
  - echo "Hello from cloud-init!"
```

## Common Modules

### Users and Groups

```yaml
users:
  - name: deploy
    groups: [sudo, docker]
    ssh_authorized_keys:
      - "ssh-ed25519 AAAA..."
    sudo: ALL=(ALL) NOPASSWD:ALL
```

### Package Installation

```yaml
packages:
  - nginx
  - postgresql
  - redis-server

package_update: true
package_upgrade: true
```

### File Creation

```yaml
write_files:
  - path: /etc/motd
    content: |
      Welcome to the server!
    permissions: '0644'
```

### Running Commands

```yaml
runcmd:
  - systemctl enable nginx
  - systemctl start nginx
```

## Debugging

If cloud-init doesn't work as expected:

```bash
# Full logs
sudo cat /var/log/cloud-init.log

# Output log
sudo cat /var/log/cloud-init-output.log

# Re-run cloud-init (testing only)
sudo cloud-init clean
sudo cloud-init init
```

## Resources

- [Cloud-init Documentation](https://cloudinit.readthedocs.io/)
- [Cloud-init Examples](https://cloudinit.readthedocs.io/en/latest/topics/examples.html)
- [Module Reference](https://cloudinit.readthedocs.io/en/latest/topics/modules.html)
