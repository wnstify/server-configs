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
| `standard-template.yaml` | Balanced security and performance | General purpose servers |
| `performance-template.yaml` | Aggressive performance tuning | High-traffic web/app servers |

## Template Comparison

| Feature | Standard | Performance |
|---------|----------|-------------|
| SSH Hardening | Yes | Yes |
| Automatic Updates | Yes | Yes |
| Dynamic Swap | Yes | Yes |
| SSD Optimizations | Yes | Yes |
| Weekly Reboot | Yes | Yes |
| TCP BBR | No | Yes |
| Network Buffer Tuning | No | Yes |
| File Descriptor Limits | No | Yes |
| Connection Backlog Tuning | No | Yes |
| TCP Keepalive Tuning | No | Yes |

### When to Use Each Template

**Standard Template** - Best for:
- Personal servers and VPS
- Development environments
- Low to medium traffic websites
- When you want minimal system changes

**Performance Template** - Best for:
- High-traffic web servers (Nginx, Apache)
- Application servers (Node.js, Python, Go, Java)
- Database servers (PostgreSQL, MySQL, Redis)
- Load balancers and reverse proxies
- Any server handling many concurrent connections

## Usage

### 1. Choose a Template

Select the template that best matches your workload.

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

## Performance Template Features

### TCP BBR Congestion Control

BBR (Bottleneck Bandwidth and Round-trip propagation time) is Google's congestion control algorithm that significantly improves network throughput, especially on high-latency connections.

```bash
# Verify BBR is active
sysctl net.ipv4.tcp_congestion_control
# Output: net.ipv4.tcp_congestion_control = bbr
```

**Benefits:**
- Higher throughput on long-distance connections
- Better performance on lossy networks
- Reduced latency under load

### Network Buffer Tuning

The performance template configures larger TCP buffers for high-throughput scenarios:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `net.core.rmem_max` | 16MB | Max receive buffer |
| `net.core.wmem_max` | 16MB | Max send buffer |
| `net.ipv4.tcp_rmem` | 4KB-16MB | TCP receive buffer range |
| `net.ipv4.tcp_wmem` | 4KB-16MB | TCP send buffer range |

### Connection Backlog

Increased backlog settings prevent connection drops under high load:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `net.core.somaxconn` | 65535 | Max socket connections queued |
| `net.core.netdev_max_backlog` | 65535 | Max packets queued |
| `net.ipv4.tcp_max_syn_backlog` | 65535 | Max SYN requests queued |

### File Descriptor Limits

High-concurrency servers need many open file descriptors:

| Setting | Value | Purpose |
|---------|-------|---------|
| `fs.file-max` | 2,097,152 | System-wide max FDs |
| `fs.nr_open` | 2,097,152 | Per-process max FDs |
| `nofile` (ulimit) | 1,048,576 | Soft/hard limit for processes |

### TCP Keepalive

Faster detection of dead connections:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `tcp_keepalive_time` | 600s | Time before sending keepalive |
| `tcp_keepalive_intvl` | 60s | Interval between probes |
| `tcp_keepalive_probes` | 3 | Number of probes before dropping |

### Network Security

Both templates include network security hardening:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `accept_source_route` | 0 | Reject source-routed packets |
| `accept_redirects` | 0 | Ignore ICMP redirects |
| `tcp_syncookies` | 1 | SYN flood protection |

## Swap Sizing Logic

Both templates use dynamic swap allocation:

| RAM | Swap Size |
|-----|-----------|
| ≤ 2GB | Equal to RAM |
| 2-8GB | Half of RAM |
| > 8GB | 4GB (capped) |

## Creating Custom Templates

Start with either template and customize:

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

## Verifying Performance Settings

After deployment, verify the performance settings are active:

```bash
# Check TCP BBR
sysctl net.ipv4.tcp_congestion_control

# Check file descriptor limits
cat /proc/sys/fs/file-max
ulimit -n

# Check network buffers
sysctl net.core.rmem_max
sysctl net.core.wmem_max

# Check connection backlog
sysctl net.core.somaxconn
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
- [TCP BBR Paper](https://research.google/pubs/pub45646/)
- [Linux Kernel Sysctl Documentation](https://www.kernel.org/doc/Documentation/sysctl/)
