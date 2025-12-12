# SSH Configuration

SSH server and client configuration templates for secure remote access.

## Files

| File | Purpose |
|------|---------|
| `sshd_config` | SSH server configuration (daemon) |
| `ssh_config` | SSH client configuration |
| `banner.txt` | Login banner template |

## SSH Server Hardening

### Recommended sshd_config Settings

```bash
# =============================================================================
# /etc/ssh/sshd_config - Hardened SSH Server Configuration
# =============================================================================

# Protocol and listening
Port 22
Protocol 2
AddressFamily inet

# Authentication
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no

# Security
StrictModes yes
MaxAuthTries 3
MaxSessions 10
LoginGraceTime 60

# Disable unused authentication methods
KerberosAuthentication no
GSSAPIAuthentication no
HostbasedAuthentication no
IgnoreRhosts yes

# Logging
SyslogFacility AUTH
LogLevel VERBOSE

# Session settings
ClientAliveInterval 300
ClientAliveCountMax 2
TCPKeepAlive yes

# Disable forwarding (enable as needed)
AllowTcpForwarding no
X11Forwarding no
AllowAgentForwarding no
PermitTunnel no

# Restrict users (uncomment and customize)
# AllowUsers deploy admin
# AllowGroups ssh-users

# Banner
Banner /etc/ssh/banner.txt

# Subsystems
Subsystem sftp /usr/lib/openssh/sftp-server
```

## Applying Changes

```bash
# Backup existing config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Copy new config
sudo cp sshd_config /etc/ssh/sshd_config

# Validate configuration
sudo sshd -t

# Restart SSH service
sudo systemctl restart sshd
```

> **Warning**: Always keep an existing session open while testing SSH changes. If the config is invalid, you could lock yourself out.

## SSH Key Management

### Generate Secure Keys

```bash
# Ed25519 (recommended - modern, fast, secure)
ssh-keygen -t ed25519 -C "your_email@example.com"

# RSA (if Ed25519 not supported - use 4096 bits minimum)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### Deploy Keys

```bash
# Copy public key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server

# Or manually add to authorized_keys
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

### Key Permissions

```bash
# On local machine
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# On server
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## SSH Client Configuration

### ~/.ssh/config

```bash
# =============================================================================
# SSH Client Configuration
# =============================================================================

# Default settings for all hosts
Host *
    AddKeysToAgent yes
    IdentitiesOnly yes
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Example: Production server
Host prod
    HostName server.example.com
    User deploy
    Port 22
    IdentityFile ~/.ssh/id_ed25519_prod

# Example: Jump host / bastion
Host bastion
    HostName bastion.example.com
    User admin
    IdentityFile ~/.ssh/id_ed25519_bastion

# Example: Access through jump host
Host internal-server
    HostName 10.0.0.50
    User deploy
    ProxyJump bastion
```

## Security Recommendations

### Do
- Use Ed25519 or RSA-4096 keys
- Disable password authentication
- Disable root login
- Use AllowUsers/AllowGroups to restrict access
- Enable fail2ban for brute force protection
- Use non-standard ports if exposed to internet
- Keep SSH updated

### Don't
- Use DSA or RSA < 2048 bits
- Allow password authentication on public servers
- Permit root login
- Disable StrictModes
- Ignore SSH security warnings

## Fail2ban Integration

```bash
# Install fail2ban
sudo apt install fail2ban

# Create SSH jail configuration
sudo tee /etc/fail2ban/jail.d/sshd.conf << 'EOF'
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
EOF

# Restart fail2ban
sudo systemctl restart fail2ban

# Check status
sudo fail2ban-client status sshd
```

## Two-Factor Authentication (Optional)

```bash
# Install Google Authenticator
sudo apt install libpam-google-authenticator

# Configure for user
google-authenticator

# Add to PAM configuration
# /etc/pam.d/sshd:
# auth required pam_google_authenticator.so

# Enable in sshd_config:
# ChallengeResponseAuthentication yes
# AuthenticationMethods publickey,keyboard-interactive
```

## Troubleshooting

```bash
# Test configuration
sudo sshd -t

# Verbose connection (client-side)
ssh -vvv user@server

# Check auth log
sudo tail -f /var/log/auth.log

# Check SSH service status
sudo systemctl status sshd
```

## Resources

- [OpenSSH Documentation](https://www.openssh.com/manual.html)
- [SSH.com Security Best Practices](https://www.ssh.com/academy/ssh/security)
- [Mozilla SSH Guidelines](https://infosec.mozilla.org/guidelines/openssh)
