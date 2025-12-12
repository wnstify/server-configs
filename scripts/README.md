# Scripts

This directory contains setup and maintenance scripts for server administration.

## Directory Structure

```
scripts/
├── setup/          # Initial server setup scripts
├── maintenance/    # Ongoing maintenance tasks
├── security/       # Security-related scripts
└── utils/          # Utility scripts
```

## Available Scripts

| Script | Purpose | Usage |
|--------|---------|-------|
| *Coming soon* | | |

## Script Standards

All scripts in this repository follow these standards:

### Header Template

```bash
#!/bin/bash
# =============================================================================
# Script: script-name.sh
# Description: Brief description of what this script does
# Author: Your Name
# Usage: ./script-name.sh [options]
#
# Options:
#   -h, --help     Show this help message
#   -v, --verbose  Enable verbose output
#
# Examples:
#   ./script-name.sh
#   ./script-name.sh --verbose
#
# Requirements:
#   - Ubuntu 22.04+ or Debian 12+
#   - Root/sudo access
# =============================================================================
```

### Best Practices

1. **Use strict mode:**
   ```bash
   set -euo pipefail
   ```

2. **Check for root when needed:**
   ```bash
   if [[ $EUID -ne 0 ]]; then
       echo "This script must be run as root"
       exit 1
   fi
   ```

3. **Use functions:**
   ```bash
   log_info() { echo "[INFO] $*"; }
   log_error() { echo "[ERROR] $*" >&2; }
   ```

4. **Validate before acting:**
   ```bash
   command -v docker &>/dev/null || { echo "Docker not found"; exit 1; }
   ```

## Contributing

When adding new scripts:

1. Follow the header template
2. Include usage examples
3. Test on supported platforms
4. Add to this README
5. Use shellcheck for validation

```bash
# Validate your script
shellcheck your-script.sh
```

## Planned Scripts

- [ ] `setup/initial-setup.sh` - Post-boot setup
- [ ] `setup/docker-install.sh` - Docker CE installation
- [ ] `maintenance/cleanup.sh` - System cleanup
- [ ] `maintenance/backup.sh` - Configuration backup
- [ ] `security/audit.sh` - Security audit
- [ ] `security/update-ssh-keys.sh` - SSH key management
