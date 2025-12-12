# Security Policy

## Supported Versions

We actively maintain security updates for configurations targeting the following platforms:

| Platform | Version | Supported |
|----------|---------|-----------|
| Ubuntu | 24.04 LTS | :white_check_mark: |
| Ubuntu | 22.04 LTS | :white_check_mark: |
| Ubuntu | 20.04 LTS | :white_check_mark: |
| Debian | 12 (Bookworm) | :white_check_mark: |
| Debian | 11 (Bullseye) | :white_check_mark: |
| Debian | 10 (Buster) | :x: |
| Ubuntu | 18.04 LTS | :x: |

## Reporting a Vulnerability

If you discover a security vulnerability in any of the configurations, please report it responsibly.

### How to Report

1. **Do NOT open a public issue** for security vulnerabilities
2. Send details to the repository maintainer via private communication
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

### What to Expect

- Acknowledgment within 48 hours
- Assessment and response within 7 days
- Credit in the fix commit (unless you prefer anonymity)

## Security Best Practices

When using configurations from this repository:

### Before Deployment

1. **Review all configurations** before applying to production systems
2. **Replace all placeholder values** (`<YOUR_USERNAME>`, `<YOUR_SSH_PUBLIC_KEY>`, etc.)
3. **Test in a staging environment** first
4. **Backup existing configurations** before applying changes

### Sensitive Data

**Never commit to this repository:**
- Private SSH keys
- Passwords or passphrases
- API keys or tokens
- Personal identification information
- Internal IP addresses or hostnames
- Any credentials or secrets

### SSH Key Security

- Use Ed25519 keys (preferred) or RSA 4096-bit minimum
- Protect private keys with a strong passphrase
- Use separate keys for different purposes
- Rotate keys periodically

```bash
# Generate a secure Ed25519 key
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### Configuration Security

When customizing templates:

1. **Principle of least privilege**: Only grant necessary permissions
2. **Avoid NOPASSWD sudo** in production when possible
3. **Enable firewall** (ufw/iptables) with minimal open ports
4. **Keep systems updated**: Automated security updates are enabled by default
5. **Monitor logs**: Set up log aggregation and alerting

## Security Features

### Included by Default

- **SSH hardening**
  - Password authentication disabled
  - Root login disabled
  - Public key authentication enforced
  - Strict modes enabled

- **Automatic updates**
  - Unattended security updates enabled
  - Configurable reboot schedule

- **File permissions**
  - Swap file: 600
  - SSH config: 644
  - Private keys: 600

### Recommended Additions

Consider adding these for production:

```bash
# Fail2ban for brute force protection
apt install fail2ban

# UFW firewall
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw enable

# Audit logging
apt install auditd
```

## Compliance Considerations

These configurations provide a foundation for various compliance requirements, but additional hardening may be needed:

- **CIS Benchmarks**: Partial compliance; review CIS Ubuntu/Debian benchmarks
- **PCI-DSS**: Additional controls required
- **HIPAA**: Additional controls and documentation required
- **SOC 2**: Foundational controls included

## Security Audit Checklist

Before deploying to production:

- [ ] All placeholder values replaced
- [ ] SSH keys are unique per environment
- [ ] Firewall rules configured
- [ ] Unnecessary services disabled
- [ ] Log monitoring configured
- [ ] Backup strategy in place
- [ ] Incident response plan documented
- [ ] Access controls documented
- [ ] Configuration tested in staging

## Resources

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [Ubuntu Security Guide](https://ubuntu.com/security)
- [Debian Security Information](https://www.debian.org/security/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)

---

Security is a continuous process, not a destination. Regularly review and update your configurations.
