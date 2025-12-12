# Sysctl Configurations

Kernel parameter tuning configurations for different workloads.

## What is Sysctl?

`sysctl` is used to modify kernel parameters at runtime. These parameters control various aspects of the Linux kernel including networking, memory management, and security.

## Configuration Files

| File | Purpose | Use Case |
|------|---------|----------|
| `standard.conf` | Balanced defaults | General purpose servers |
| `performance.conf` | Aggressive tuning | High-traffic applications |
| `security.conf` | Security hardening | Exposed/public servers |
| `network.conf` | Network optimizations | Load balancers, proxies |

## Usage

### Apply Configuration

```bash
# Copy to sysctl.d
sudo cp performance.conf /etc/sysctl.d/99-performance.conf

# Apply immediately
sudo sysctl -p /etc/sysctl.d/99-performance.conf

# Or apply all configurations
sudo sysctl --system
```

### Verify Settings

```bash
# Check specific value
sysctl vm.swappiness

# Check all custom settings
sysctl -a | grep -E "(swappiness|dirty_ratio)"
```

## Common Parameters

### Memory Management

```bash
# Reduce swappiness (prefer RAM over swap)
vm.swappiness = 10

# Control dirty page flushing
vm.dirty_ratio = 10
vm.dirty_background_ratio = 5

# Overcommit settings
vm.overcommit_memory = 0
vm.overcommit_ratio = 50
```

### Network Tuning

```bash
# Increase connection backlog
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535

# TCP buffer sizes
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216

# Enable TCP BBR congestion control
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

# TCP keepalive
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 60
net.ipv4.tcp_keepalive_probes = 3
```

### Security Hardening

```bash
# Disable IP forwarding (unless routing)
net.ipv4.ip_forward = 0

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Enable SYN cookies (SYN flood protection)
net.ipv4.tcp_syncookies = 1

# Log martian packets
net.ipv4.conf.all.log_martians = 1

# Disable source routing
net.ipv4.conf.all.accept_source_route = 0
```

### File Descriptors

```bash
# Increase file descriptor limits
fs.file-max = 2097152
fs.nr_open = 2097152
```

## Workload-Specific Tuning

### Web Servers (Nginx, Apache)

```bash
net.core.somaxconn = 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15
```

### Database Servers (PostgreSQL, MySQL)

```bash
vm.swappiness = 1
vm.dirty_ratio = 40
vm.dirty_background_ratio = 10
```

### Container Hosts (Docker, Kubernetes)

```bash
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
fs.inotify.max_user_watches = 524288
```

## Persistence

Settings in `/etc/sysctl.d/*.conf` persist across reboots. Files are loaded in alphabetical order, so use numbered prefixes:

```
/etc/sysctl.d/
├── 10-network.conf      # Base network settings
├── 20-security.conf     # Security hardening
└── 99-custom.conf       # Custom overrides (loaded last)
```

## Troubleshooting

```bash
# Check for errors
sudo sysctl --system 2>&1 | grep -i error

# View current value
sysctl -n vm.swappiness

# Temporarily change (lost on reboot)
sudo sysctl -w vm.swappiness=10
```

## Resources

- [Linux Kernel Documentation](https://www.kernel.org/doc/Documentation/)
- [Red Hat Performance Tuning Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/monitoring_and_managing_system_status_and_performance/)
- [Brendan Gregg's Linux Performance](https://www.brendangregg.com/linuxperf.html)
