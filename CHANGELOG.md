# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.3.0] - 2025-12-14

### Added
- Performance-optimized cloud-init template (`cloud-init/performance-template.yaml`)
- TCP BBR congestion control for improved network throughput
- Network buffer tuning for high-traffic servers
- File descriptor limits configuration (2M system-wide, 1M per process)
- Connection backlog tuning (65535 for somaxconn, netdev_max_backlog, tcp_max_syn_backlog)
- TCP keepalive optimization for faster dead connection detection
- Comprehensive template comparison documentation in cloud-init README
- Performance verification commands documentation

## [0.2.0] - 2025-12-14

### Added
- GitHub SSH key setup guide for Linux, macOS, and Windows (`docs/git-setup-guide.md`)
- VS Code Git/SSH integration documentation
- Comprehensive troubleshooting section for SSH issues
- Quick reference one-liners for all platforms

## [0.1.0] - 2025-12-14

### Added
- Initial repository structure
- Cloud-init standard template with security hardening
- Comprehensive documentation (README, SECURITY, CONTRIBUTING)
- Directory structure for organized configurations
  - `cloud-init/` - Cloud-init templates
  - `scripts/` - Setup and maintenance scripts
  - `sysctl/` - Kernel parameter configurations
  - `ssh/` - SSH server configurations
  - `security/` - Security hardening configs
  - `docs/` - Additional documentation

### Security
- SSH hardening (key-only auth, no root login)
- Dynamic swap sizing
- SSD/NVMe optimizations
- Automatic security updates via unattended-upgrades

---

## Version History

| Version | Date | Description |
|---------|------|-------------|
| 0.3.0 | 2025-12-14 | Performance cloud-init template |
| 0.2.0 | 2025-12-14 | GitHub SSH setup guide |
| 0.1.0 | 2025-12-14 | Initial release |

## Versioning

This project uses [Semantic Versioning](https://semver.org/):

- **MAJOR** version for incompatible changes
- **MINOR** version for new functionality (backwards-compatible)
- **PATCH** version for bug fixes (backwards-compatible)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to contribute to this project.
