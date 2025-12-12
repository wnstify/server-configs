# Contributing to Server Configs

Thank you for your interest in contributing! This document provides guidelines and best practices for contributing to this repository.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Style Guidelines](#style-guidelines)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)

## Code of Conduct

- Be respectful and inclusive
- Focus on constructive feedback
- Help others learn and grow
- Keep discussions professional

## Getting Started

1. **Fork the repository** to your own GitHub account
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/server-configs.git
   cd server-configs
   ```
3. **Create a branch** for your changes:
   ```bash
   git checkout -b feature/your-feature-name
   ```

## How to Contribute

### Types of Contributions Welcome

- **New configurations**: Templates for different use cases
- **Bug fixes**: Corrections to existing configs
- **Documentation**: Improvements to README, comments, or guides
- **Platform support**: Configs for additional distributions
- **Security improvements**: Hardening recommendations
- **Performance tuning**: Optimization suggestions

### Areas Needing Help

- [ ] Additional cloud-init templates (minimal, docker-ready, etc.)
- [ ] RHEL/CentOS/Rocky Linux support
- [ ] Alpine Linux configurations
- [ ] Ansible playbook equivalents
- [ ] Monitoring integration configs (Prometheus, Grafana)
- [ ] Container/Kubernetes configurations
- [ ] Network hardening templates

## Style Guidelines

### YAML Files

```yaml
# Use 2-space indentation
users:
  - name: deploy
    groups: sudo

# Add descriptive comments
# This section configures SSH hardening
runcmd:
  - echo "PermitRootLogin no" >> /etc/ssh/sshd_config

# Use meaningful names
# Good: performance-web-server.yaml
# Bad: config2.yaml
```

### File Naming

- Use lowercase with hyphens: `web-server-standard.yaml`
- Be descriptive: `debian-12-hardened.yaml` not `d12.yaml`
- Use appropriate extensions: `.yaml`, `.conf`, `.sh`

### Comments

Every configuration file should include:

```yaml
# =============================================================================
# Title: Brief description
# Target: Ubuntu 22.04+ / Debian 12+
# Purpose: What this config does
#
# Usage:
#   How to use this configuration
#
# Notes:
#   Any important caveats or requirements
# =============================================================================
```

### Scripts

```bash
#!/bin/bash
# =============================================================================
# Script Name: script-name.sh
# Description: What this script does
# Usage: ./script-name.sh [options]
# =============================================================================

set -euo pipefail  # Always use strict mode

# Use functions for organization
main() {
    # Main logic here
}

main "$@"
```

### Security Guidelines

1. **Never include real credentials** - Use placeholders:
   - `<YOUR_USERNAME>`
   - `<YOUR_SSH_PUBLIC_KEY>`
   - `<YOUR_API_KEY>`

2. **Document security implications** - Note any security trade-offs

3. **Default to secure settings** - Require opt-in for less secure options

4. **Follow least privilege** - Request minimum necessary permissions

## Testing

### Before Submitting

1. **Syntax validation** for YAML files:
   ```bash
   # Using Python
   python -c "import yaml; yaml.safe_load(open('your-file.yaml'))"

   # Using yamllint (recommended)
   yamllint your-file.yaml
   ```

2. **Shell script validation**:
   ```bash
   # Check syntax
   bash -n your-script.sh

   # Use shellcheck (recommended)
   shellcheck your-script.sh
   ```

3. **Cloud-init validation**:
   ```bash
   cloud-init schema --config-file your-cloud-init.yaml
   ```

### Testing in VMs

For significant changes, test in a virtual environment:

```bash
# Using multipass (Ubuntu)
multipass launch --name test-vm --cloud-init your-config.yaml

# Using Vagrant
vagrant up

# Using Docker (limited cloud-init support)
docker run -it ubuntu:22.04 /bin/bash
```

## Submitting Changes

### Commit Messages

Follow conventional commits format:

```
type(scope): brief description

Longer description if needed.

- Bullet points for multiple changes
- Another change

Fixes #123
```

Types:
- `feat`: New feature or configuration
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting, no logic change
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance tasks

Examples:
```
feat(cloud-init): add Docker-ready template

Adds a new cloud-init template pre-configured for Docker workloads.
Includes:
- Docker CE installation
- Docker Compose
- Optimized storage driver settings

docs(readme): update supported platforms table

fix(ssh): correct sshd_config path for Debian 12
```

### Pull Request Process

1. **Update documentation** if your changes affect usage
2. **Add yourself to contributors** (optional)
3. **Fill out the PR template** completely
4. **Link related issues** using "Fixes #123" or "Relates to #456"
5. **Request review** from maintainers

### PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] New configuration/feature
- [ ] Bug fix
- [ ] Documentation update
- [ ] Other (describe)

## Testing Done
- [ ] Syntax validated
- [ ] Tested in VM/container
- [ ] Tested on actual cloud provider

## Checklist
- [ ] My code follows the style guidelines
- [ ] I have commented my code where necessary
- [ ] I have updated the documentation
- [ ] My changes generate no new warnings
- [ ] No sensitive data is included
```

## Questions?

- Open an issue for questions
- Tag with `question` label
- Check existing issues first

---

Thank you for contributing!
