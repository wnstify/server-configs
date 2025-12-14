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

## Security Hardening Explained

Both templates implement defense-in-depth security. This section explains each security measure and why it represents the safest configuration.

### SSH Security

SSH is the primary attack vector for Linux servers. These settings follow security best practices recommended by [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks), [NIST](https://www.nist.gov/), and [Mozilla's OpenSSH Guidelines](https://infosec.mozilla.org/guidelines/openssh).

| Setting | Value | Security Rationale |
|---------|-------|-------------------|
| `PermitRootLogin` | `no` | **Critical.** Root has unlimited privileges. If compromised, attacker has full control. Using a non-root user with sudo creates an audit trail and requires two-step privilege escalation. |
| `PubkeyAuthentication` | `yes` | SSH keys use 256-bit (Ed25519) or 2048-4096-bit (RSA) cryptographic keys. Virtually impossible to brute force compared to passwords. |
| `PasswordAuthentication` | `no` | **Critical.** Passwords are vulnerable to brute force attacks, credential stuffing, keyloggers, and social engineering. SSH keys eliminate all these attack vectors. |
| `StrictModes` | `yes` | SSH refuses to start if key files have insecure permissions (e.g., world-readable). Prevents accidental exposure of private keys. |

#### Why Key-Only Authentication is Safest

**Password vulnerabilities:**
- Brute force attacks: Automated tools try thousands of passwords per second
- Credential stuffing: Attackers use leaked passwords from data breaches
- Dictionary attacks: Common passwords are tried first
- Keyloggers: Malware can capture typed passwords
- Shoulder surfing: Passwords can be observed

**SSH key advantages:**
- Ed25519 keys have 128-bit security (equivalent to ~3000-bit RSA)
- Private key never leaves your machine
- No secret transmitted over the network
- Can't be guessed or brute-forced in any practical timeframe
- Optional passphrase adds second factor

### User Account Security

```yaml
users:
  - name: <YOUR_USERNAME>
    lock_passwd: true      # Disable password login entirely
    sudo: ALL=(ALL) NOPASSWD:ALL
```

| Setting | Security Rationale |
|---------|-------------------|
| `lock_passwd: true` | Completely disables password authentication for the user. Even if an attacker gains shell access through another vulnerability, they cannot use `su` to switch users with a password. |
| Non-root user | **Principle of least privilege.** Daily operations don't require root. Sudo provides temporary elevation with full audit logging in `/var/log/auth.log`. |
| Root `lock_passwd: true` | Ensures root account cannot be accessed directly, even via console. All privileged access must go through sudo. |

### Automatic Security Updates

```bash
apt install unattended-upgrades -y
dpkg-reconfigure -f noninteractive unattended-upgrades
```

**Why automatic updates are critical:**

| Threat | Mitigation |
|--------|-----------|
| Zero-day exploits | Patches applied within hours of release, not days/weeks |
| Known vulnerabilities (CVEs) | Automatic patching closes the window of exposure |
| Human error | No reliance on manual intervention |
| Compliance | Many standards (PCI-DSS, HIPAA, SOC2) require timely patching |

**Ubuntu/Debian unattended-upgrades:**
- Only applies security updates by default (not feature updates)
- Runs daily via systemd timer
- Can be configured to auto-reboot if needed
- Logs all actions to `/var/log/unattended-upgrades/`

### Network Security Settings

The performance template includes additional network-level protections:

| Parameter | Value | Attack Prevented | Explanation |
|-----------|-------|-----------------|-------------|
| `net.ipv4.conf.all.accept_source_route` | `0` | IP Spoofing | Source routing lets attackers specify the route packets take, bypassing firewalls. Disabling it forces packets through normal routing. |
| `net.ipv4.conf.all.accept_redirects` | `0` | MITM Attacks | ICMP redirects can be used to reroute traffic through an attacker's machine. Disabling prevents route table manipulation. |
| `net.ipv4.tcp_syncookies` | `1` | SYN Flood DDoS | During SYN floods, the kernel uses cryptographic cookies instead of allocating memory for half-open connections. Allows legitimate connections during attacks. |

#### Additional Recommended Network Hardening

These are included in the performance template and recommended for all production servers:

```bash
# Prevent IP spoofing
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.default.rp_filter=1

# Ignore ICMP broadcast requests (Smurf attack prevention)
net.ipv4.icmp_echo_ignore_broadcasts=1

# Ignore bogus ICMP errors
net.ipv4.icmp_ignore_bogus_error_responses=1

# Disable IPv6 if not used (reduces attack surface)
net.ipv6.conf.all.disable_ipv6=1
```

### File System Security

#### Swap File Permissions

```bash
chmod 600 /swapfile
```

**Why this matters:** Swap space may contain sensitive data from memory, including:
- Encryption keys
- Passwords in memory
- Session tokens
- Database contents

Permission `600` (owner read/write only) ensures only root can access swap contents.

#### Mount Options

```bash
noatime
```

While primarily a performance optimization, `noatime` also:
- Reduces disk writes (extends SSD life)
- Marginally improves security by not updating access times that could leak information about file access patterns

### Weekly Maintenance Reboot

```bash
0 4 * * sun /sbin/shutdown -r now
```

**Security benefits:**

| Benefit | Explanation |
|---------|-------------|
| Kernel updates applied | Many security patches require reboot to take effect. Weekly reboot ensures kernel patches are applied within 7 days. |
| Memory cleared | Long-running processes can accumulate sensitive data in memory. Reboot ensures clean state. |
| Stuck processes terminated | Zombie or hung processes that might have been compromised are terminated. |
| Services restarted with new configs | Ensures services pick up security configuration changes. |

### Verifying Security Settings

After deployment, verify your security configuration:

```bash
# Check SSH settings
sudo sshd -T | grep -E "(permitrootlogin|passwordauthentication|pubkeyauthentication)"

# Verify password is locked
sudo passwd -S root
sudo passwd -S <YOUR_USERNAME>
# Should show "L" (locked) or "NP" (no password)

# Check unattended-upgrades is enabled
systemctl status unattended-upgrades

# Verify network security settings
sysctl net.ipv4.conf.all.accept_source_route
sysctl net.ipv4.conf.all.accept_redirects
sysctl net.ipv4.tcp_syncookies

# Check swap permissions
ls -la /swapfile
# Should show: -rw------- 1 root root
```

### Security Checklist

After deployment, ensure:

- [ ] Can SSH with key only (password rejected)
- [ ] Cannot SSH as root directly
- [ ] `sudo` works for your user
- [ ] Unattended-upgrades is active
- [ ] Swap file has 600 permissions
- [ ] Network security sysctls are set

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

### Cloud-Init
- [Cloud-init Documentation](https://cloudinit.readthedocs.io/)
- [Cloud-init Examples](https://cloudinit.readthedocs.io/en/latest/topics/examples.html)
- [Module Reference](https://cloudinit.readthedocs.io/en/latest/topics/modules.html)

### Performance
- [TCP BBR Paper](https://research.google/pubs/pub45646/)
- [Linux Kernel Sysctl Documentation](https://www.kernel.org/doc/Documentation/sysctl/)

### Security
- [CIS Benchmarks for Ubuntu/Debian](https://www.cisecurity.org/cis-benchmarks)
- [Mozilla OpenSSH Guidelines](https://infosec.mozilla.org/guidelines/openssh)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [SSH Key Best Practices](https://www.ssh.com/academy/ssh/keygen)
