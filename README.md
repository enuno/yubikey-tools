# YubiKey Tools

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/enuno/yubikey-tools)

A comprehensive collection of tools for configuring, managing, and automating YubiKey hardware security keys. This project focuses on providing developers and security administrators with reliable, well-tested utilities for YubiKey lifecycle management, batch configuration, security auditing, and integration with enterprise authentication systems.

## Features

- **Automated YubiKey Setup**: Complete GPG key generation and YubiKey configuration with `yubikey-setup.sh`
- **Python Library**: Modular library for YubiKey operations (PIV, FIDO2, OATH, OpenPGP)
- **CLI Tools**: Command-line utilities for common YubiKey management tasks
- **Security-First**: Built with security best practices, comprehensive auditing, and sanitized logging
- **Hardware Testing**: Integration tests with real YubiKey devices
- **Custom Commands**: YubiKey-specific slash commands for security audits and compliance
- **Agent Framework**: Specialized AI agents for security validation and crypto review

## Quick Start

### Automated YubiKey Setup (Recommended)

The fastest way to configure a YubiKey with GPG keys:

```bash
# Run the automated setup script
./yubikey-setup.sh

# Or from the scripts directory
./scripts/automation/yubikey-setup/yubikey-setup.sh
```

The setup script supports three modes:
- **Generate**: Create new GPG keys and transfer to YubiKey
- **Load**: Import existing keys from backup
- **Backup**: Export configuration and keys

See [yubikey-setup documentation](scripts/automation/yubikey-setup/README.md) for detailed usage.

### Python Library (Coming Soon)

```bash
# Install from source
pip install -e .

# Or install from PyPI (when released)
pip install yubikey-tools
```

## Repository Structure

```
yubikey-tools/
├── src/                       # Python library source
│   ├── core/                  # Core YubiKey operations
│   │   ├── config/            # Configuration schemas
│   │   └── operations/        # FIDO2, PIV, OATH, OpenPGP handlers
│   ├── tools/                 # CLI tools
│   ├── validators/            # Security validation modules
│   └── utils/                 # Helper functions and utilities
│
├── tests/                     # Test suite
│   ├── unit/                  # Unit tests
│   ├── integration/           # Hardware integration tests
│   └── fixtures/              # Test data and mock YubiKeys
│
├── docs/                      # Documentation
│   ├── api/                   # API documentation
│   ├── guides/                # User guides and tutorials
│   └── security/              # Security considerations
│
├── scripts/                   # Automation scripts
│   └── automation/
│       └── yubikey-setup/     # Automated YubiKey setup tool
│
├── .claude/                   # Claude Code configuration
│   ├── commands/              # Custom slash commands
│   └── agents/                # Specialized AI agents
│
├── AGENTS.md                  # AI agent configuration
├── CLAUDE.md                  # Claude Code guidelines
├── DEVELOPMENT_PLAN.md        # Development roadmap
└── TODO.md                    # Task tracking
```

## Custom Commands

YubiKey-specific slash commands for development and security:

- `/security-audit` - Comprehensive security scan (bandit, safety, credential check)
- `/yubikey-enum` - Enumerate connected YubiKeys (read-only)
- `/test-hardware` - Run integration tests with real hardware
- `/validate-crypto` - Review cryptographic operations
- `/check-compliance` - Verify security standards compliance
- `/yubikey-backup-verify` - Verify backup integrity
- `/pin-security-check` - Audit PIN handling in code

## Specialized Agents

AI agents for YubiKey development:

- **Security Validator**: YubiKey-specific security review, attestation validation, PIN auditing
- **Hardware Tester**: Physical YubiKey integration testing and device management
- **Cryptography Reviewer**: Cryptographic code review and algorithm validation

## Documentation

- [Getting Started](docs/guides/getting-started.md) - Installation and basic usage
- [YubiKey Setup](scripts/automation/yubikey-setup/README.md) - Automated setup guide
- [Development Plan](DEVELOPMENT_PLAN.md) - Roadmap and architecture
- [Security](docs/security/threat-model.md) - Threat model and best practices
- [Contributing](CONTRIBUTING.md) - How to contribute

## Security

This project handles hardware security keys and sensitive cryptographic operations. Security is paramount:

- **No credential logging**: PINs, PUKs, and keys are never logged
- **Sanitized logging**: `@sanitize_logging` decorator on sensitive functions
- **Input validation**: All inputs validated before YubiKey operations
- **Test device protection**: Production YubiKeys never used in tests
- **Established crypto libraries**: No custom cryptographic implementations

See [CLAUDE.md](CLAUDE.md) for detailed security standards and [docs/security/](docs/security/) for threat model.

## Development Status

**Current Phase**: Foundation (Phase 1)

- ✅ Repository structure established
- ✅ Automated setup script (yubikey-setup.sh v1.1.0)
- ✅ Documentation and planning complete
- ✅ Custom commands and agents created
- 🚧 Python library in progress
- 📋 CLI tools planned

See [DEVELOPMENT_PLAN.md](DEVELOPMENT_PLAN.md) for detailed roadmap.

## Requirements

- Python 3.9+
- YubiKey Manager (ykman)
- libfido2
- GPG (for OpenPGP operations)

### Python Dependencies

- python-fido2
- yubikey-manager
- PyUSB
- cryptography

See [requirements.txt](requirements.txt) or [pyproject.toml](pyproject.toml) for complete list.

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Development Setup

```bash
# Clone repository
git clone https://github.com/yourusername/yubikey-tools.git
cd yubikey-tools

# Create virtual environment
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Install in development mode
pip install -e .[dev]

# Run tests
pytest

# Run security checks
bandit -r src/
safety check
```

## YubiKey Model Support

| Model | PIV | FIDO2 | OATH | OpenPGP | Status |
|-------|-----|-------|------|---------|--------|
| YubiKey 5 Series | ✅ | ✅ | ✅ | ✅ | Fully Supported |
| YubiKey 5 FIPS | ✅ | ✅ | ✅ | ✅ | Fully Supported |
| Security Key Series | ❌ | ✅ | ❌ | ❌ | FIDO2 Only |
| YubiKey 4 Series | ✅ | ❌ | ✅ | ✅ | Legacy Support |

See [docs/compatibility.md](docs/compatibility.md) for detailed compatibility matrix.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Yubico](https://www.yubico.com/) for YubiKey hardware and documentation
- [python-fido2](https://github.com/Yubico/python-fido2) library
- [yubikey-manager](https://github.com/Yubico/yubikey-manager) CLI tool
- Original yubikey-setup.sh contributors

## Support

- **Issues**: [GitHub Issues](https://github.com/yourusername/yubikey-tools/issues)
- **Documentation**: [Project Wiki](https://github.com/yourusername/yubikey-tools/wiki)
- **Security**: See [SECURITY.md](SECURITY.md) for vulnerability reporting

## Author

Elvis Nuno

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

---

**Status**: Active Development | **Version**: 0.1.0 (Pre-release)
