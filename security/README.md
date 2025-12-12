# Security Configurations

Security hardening configurations and templates for Linux servers.

## Directory Contents

| File | Purpose |
|------|---------|
| `ufw/` | UFW (Uncomplicated Firewall) rules |
| `fail2ban/` | Fail2ban jail configurations |
| `apparmor/` | AppArmor profiles |
| `auditd/` | Audit daemon rules |

## Quick Security Checklist

### Immediate Actions (New Server)

- [ ] Create non-root user with sudo
- [ ] Configure SSH key authentication
- [ ] Disable SSH password authentication
- [ ] Disable SSH root login
- [ ] Enable firewall (UFW)
- [ ] Enable automatic security updates
- [ ] Install and configure fail2ban

### Recommended Hardening

- [ ] Configure sysctl security parameters
- [ ] Set up audit logging
- [ ] Configure log rotation
- [ ] Implement file integrity monitoring
- [ ] Set proper file permissions
- [ ] Remove unnecessary packages
- [ ] Disable unused services

## Firewall (UFW)

### Basic Setup

```bash
# Install UFW
sudo apt install ufw

# Set defaults
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (do this before enabling!)
sudo ufw allow ssh

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose
```

### Common Rules

```bash
# Web server
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow to specific port from IP
sudo ufw allow from 192.168.1.100 to any port 22

# Rate limiting SSH
sudo ufw limit ssh

# Delete a rule
sudo ufw delete allow 80/tcp
```

## Fail2ban

### Installation

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
```

### Basic Configuration

```bash
# /etc/fail2ban/jail.local
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5
ignoreip = 127.0.0.1/8 ::1

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 24h
```

### Management Commands

```bash
# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Unban an IP
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Check banned IPs
sudo fail2ban-client get sshd banned
```

## Automatic Security Updates

### Unattended Upgrades

```bash
# Install
sudo apt install unattended-upgrades

# Configure
sudo dpkg-reconfigure -plow unattended-upgrades

# Or manually enable
sudo tee /etc/apt/apt.conf.d/20auto-upgrades << 'EOF'
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";
EOF
```

### Configuration

```bash
# /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}";
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
```

## User Security

### Password Policy

```bash
# /etc/security/pwquality.conf
minlen = 12
minclass = 3
maxrepeat = 3
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
```

### Login Limits

```bash
# /etc/security/limits.conf
* soft nproc 65535
* hard nproc 65535
* soft nofile 65535
* hard nofile 65535
```

## Audit Logging

### Install and Enable

```bash
sudo apt install auditd
sudo systemctl enable auditd
sudo systemctl start auditd
```

### Basic Audit Rules

```bash
# /etc/audit/rules.d/audit.rules

# Log all commands run by root
-a exit,always -F arch=b64 -F euid=0 -S execve -k root_commands

# Monitor /etc/passwd changes
-w /etc/passwd -p wa -k passwd_changes

# Monitor SSH configuration
-w /etc/ssh/sshd_config -p wa -k sshd_config

# Monitor sudo usage
-w /var/log/sudo.log -p wa -k sudo_log
```

## File Permissions

### Sensitive File Permissions

```bash
# SSH
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_*
chmod 644 ~/.ssh/*.pub

# System files
chmod 644 /etc/passwd
chmod 640 /etc/shadow
chmod 644 /etc/group
chmod 640 /etc/gshadow

# SSH daemon
chmod 600 /etc/ssh/*_key
chmod 644 /etc/ssh/*.pub
chmod 644 /etc/ssh/sshd_config
```

### Find World-Writable Files

```bash
# Find world-writable files
find / -type f -perm -0002 -ls 2>/dev/null

# Find world-writable directories
find / -type d -perm -0002 -ls 2>/dev/null

# Find SUID/SGID files
find / -type f \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null
```

## Service Hardening

### List Enabled Services

```bash
systemctl list-unit-files --type=service --state=enabled
```

### Disable Unnecessary Services

```bash
# Common services to disable if not needed
sudo systemctl disable cups
sudo systemctl disable avahi-daemon
sudo systemctl disable bluetooth
```

## Security Scanning Tools

```bash
# Lynis - Security auditing tool
sudo apt install lynis
sudo lynis audit system

# Chkrootkit - Rootkit scanner
sudo apt install chkrootkit
sudo chkrootkit

# RKHunter - Rootkit hunter
sudo apt install rkhunter
sudo rkhunter --check
```

## Resources

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [NIST Security Guidelines](https://www.nist.gov/cyberframework)
- [Ubuntu Security Guide](https://ubuntu.com/security/certifications/docs/usg)
- [Debian Security Manual](https://www.debian.org/doc/manuals/securing-debian-manual/)
