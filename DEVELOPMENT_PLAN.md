# YubiKey Tools - Development Roadmap

## Project Vision
Deliver production-ready tools that simplify YubiKey deployment and management while maintaining the highest security standards.

## Current Status
- ✅ Comprehensive bash automation script (yubikey-setup.sh v1.1.0)
- ✅ CLAUDE.md documentation complete
- ✅ AGENTS.md agent configuration complete
- ✅ Repository structure established
- 🚧 Python library architecture in progress
- 📋 Custom commands and agents pending

## Phase 1: Foundation (Weeks 1-2)
**Goal**: Establish repository structure and core infrastructure

### Tasks
- [x] Create directory structure (src/, tests/, docs/, scripts/)
- [x] Move yubikey-setup.sh to scripts/automation/
- [ ] Set up Python project structure (pyproject.toml, setup.py)
- [ ] Create .gitignore for Python/security
- [ ] Initialize pytest configuration
- [ ] Set up black, isort, flake8, mypy, bandit configurations
- [ ] Create pre-commit hooks
- [ ] Initialize CI/CD pipeline (.github/workflows/)
- [ ] Create __init__.py files in all Python packages

### Deliverables
- ✅ Complete directory structure
- ✅ yubikey-setup.sh organized in scripts/automation/
- 📋 Python build configuration
- 📋 Linting and formatting tools configured
- 📋 Basic CI/CD running

### Success Criteria
- All directories created per CLAUDE.md specification
- Python environment activates without errors
- All linters run successfully (even if no code yet)
- CI/CD pipeline executes on push

## Phase 2: Core Library Development (Weeks 3-6)
**Goal**: Build shared library for YubiKey operations

### src/core/ Components

#### src/core/operations/
- [ ] `__init__.py` - Package initialization
- [ ] `yubikey_detection.py` - YubiKey enumeration and connection
  - `list_yubikeys()` - List all connected YubiKeys
  - `get_yubikey_info(serial)` - Get device information
  - `check_firmware_version()` - Validate firmware compatibility
- [ ] `fido2_operations.py` - FIDO2 credential management
  - List FIDO2 credentials
  - Create resident keys
  - Delete credentials
  - Verify attestation
- [ ] `piv_operations.py` - PIV key and certificate management
  - Generate PIV keys
  - Import certificates
  - Manage slots (0x9a, 0x9c, 0x9d, 0x9e)
  - Certificate validation
- [ ] `oath_operations.py` - OATH (TOTP/HOTP) operations
  - List OATH credentials
  - Add/remove credentials
  - Generate TOTP codes
- [ ] `openpgp_operations.py` - OpenPGP applet management
  - Key generation
  - Key import/export
  - Signature operations

#### src/core/config/
- [ ] `__init__.py` - Package initialization
- [ ] `config_schema.py` - Configuration data models (dataclasses)
- [ ] `config_loader.py` - YAML/JSON configuration loading
- [ ] `defaults.py` - Default values and constants

#### src/validators/
- [ ] `__init__.py` - Package initialization
- [ ] `attestation_validator.py` - FIDO2 attestation verification
- [ ] `certificate_validator.py` - PIV certificate validation
- [ ] `pin_validator.py` - PIN format and complexity validation
- [ ] `policy_validator.py` - Security policy enforcement

#### src/utils/
- [ ] `__init__.py` - Package initialization
- [ ] `logging_utils.py` - Sanitized logging
  - `@sanitize_logging` decorator
  - `sanitize_log_output(text)` function
  - Pattern matching for PINs, keys, credentials
- [ ] `error_handling.py` - YubiKey-specific exception classes
  - `YubiKeyConnectionError`
  - `YubiKeyAuthenticationError`
  - `YubiKeyOperationError`
- [ ] `connection_manager.py` - YubiKey connection lifecycle
  - Context manager for safe connections
  - Automatic cleanup on errors

### Testing Strategy
- Minimum 85% overall coverage, 95% for core security modules
- Mock YubiKeys for unit tests
- Integration tests with test YubiKey devices (documented serial numbers)
- Security tests for attack scenarios (invalid PINs, forged attestations)

### Deliverables
- Complete src/core/ library with comprehensive tests
- API documentation (Sphinx)
- Test coverage reports (HTML and terminal)
- Security audit passing (bandit)

### Success Criteria
- [ ] All core modules implemented with type hints
- [ ] Test coverage ≥ 85% overall, ≥ 95% security modules
- [ ] Zero security warnings from bandit
- [ ] All tests passing in CI/CD
- [ ] API documentation generated

## Phase 3: CLI Tools Development (Weeks 7-10)
**Goal**: Create modular CLI tools using core library

### src/tools/
- [ ] `__init__.py` - Package initialization
- [ ] `yubikey-batch-config` - Configure multiple YubiKeys
  - Read configuration from YAML/JSON
  - Apply settings to multiple devices
  - Dry-run mode
  - Detailed logging and reporting
- [ ] `yubikey-piv-manager` - PIV certificate management
  - Generate keys in PIV slots
  - Import/export certificates
  - Certificate renewal workflows
  - Touch policy configuration
- [ ] `yubikey-fido2-manager` - FIDO2 credential management
  - List resident credentials
  - Delete credentials
  - Generate new credentials
  - PIN management
- [ ] `yubikey-oath-manager` - TOTP/HOTP management
  - Add OATH credentials
  - Generate codes
  - Export/import credentials
  - QR code support
- [ ] `yubikey-audit` - Security auditing and reporting
  - Check PIN retry counters
  - Validate firmware versions
  - Audit certificate expiry
  - Security policy compliance
  - Generate audit reports (JSON, HTML, PDF)
- [ ] `yubikey-backup` - Backup and restore operations
  - Export public keys and configuration
  - Create timestamped backups
  - Restore from backup
  - Verify backup integrity

### Tool Requirements
- Click or Typer for CLI framework
- Consistent command structure (verb-noun pattern)
- JSON output option for scripting
- Verbose/debug modes with sanitized logging
- Dry-run capability for destructive operations
- Exit codes for CI/CD integration (0 = success, 1 = error, 2 = partial)
- Progress bars for long operations
- Color-coded output (success = green, error = red, warning = yellow)

### Deliverables
- 6 CLI tools with comprehensive --help text
- Integration tests for each tool
- User guides in docs/guides/
- Man pages for Linux/macOS

### Success Criteria
- [ ] All tools installed via pip and accessible in PATH
- [ ] Consistent CLI interface across tools
- [ ] Comprehensive error messages with remediation hints
- [ ] All tools support --dry-run
- [ ] Integration tests passing on real hardware

## Phase 4: Custom Commands & Agents (Weeks 11-12)
**Goal**: Create YubiKey-specific development commands

### Custom Commands (.claude/commands/)
- [ ] `security-audit.md` - Run comprehensive security scan
  - Execute bandit on Python code
  - Run safety check on dependencies
  - Search for hardcoded credentials
  - Check for sensitive data in logs
  - Validate @sanitize_logging usage
  - Generate security report
- [ ] `yubikey-enum.md` - List connected YubiKeys (read-only)
  - Display serial numbers
  - Show firmware versions
  - Display current configuration
  - Identify test vs. production devices
- [ ] `test-hardware.md` - Run integration tests with real hardware
  - Verify test YubiKeys present
  - Prompt for confirmation
  - Execute hardware tests
  - Generate detailed test report
- [ ] `validate-crypto.md` - Review cryptographic operations
  - Search for crypto library usage
  - Validate no custom crypto
  - Check random number generation
  - Review certificate validation
  - Verify key generation parameters
- [ ] `check-compliance.md` - Verify security standards compliance
  - Check test coverage thresholds
  - Verify no credentials in code/logs
  - Validate input sanitization
  - Check error handling
  - Generate compliance report
- [ ] `yubikey-backup-verify.md` - Verify backup integrity
  - Check backup directory structure
  - Validate GPG key files
  - Verify public key exports
  - Test restoration dry-run
- [ ] `pin-security-check.md` - Audit PIN handling in code
  - Search for PIN-related variables
  - Verify no PIN logging
  - Check PIN validation functions
  - Validate secure input methods

### Custom Agents (.claude/agents/)
- [ ] `security-validator.md` - YubiKey-specific security review
  - Attestation validation
  - PIN/PUK handling audit
  - Cryptographic operation review
- [ ] `hardware-tester.md` - Physical YubiKey testing
  - Test device management
  - Integration test execution
  - Firmware compatibility validation
- [ ] `crypto-reviewer.md` - Cryptographic code review
  - Crypto library usage review
  - Algorithm validation
  - Key generation audit

### Deliverables
- 7 custom commands with comprehensive documentation
- 3 custom agents with clear responsibilities
- AGENTS.md documentation complete
- Examples of command usage in docs/

### Success Criteria
- [ ] All commands executable via /command-name
- [ ] All agents properly configured
- [ ] Commands integrate with existing tools
- [ ] Documentation includes usage examples

## Phase 5: Documentation & Release (Weeks 13-14)
**Goal**: Comprehensive documentation and v1.0 release

### Documentation
- [ ] API documentation (Sphinx HTML)
  - All modules documented
  - All classes and functions with docstrings
  - Code examples for common operations
- [ ] Getting Started guide
  - Installation instructions
  - Quick start examples
  - Common workflows
- [ ] CLI reference with examples
  - Each tool documented
  - Common usage patterns
  - Troubleshooting tips
- [ ] Security best practices guide
  - YubiKey lifecycle management
  - Backup strategies
  - PIN/PUK management
  - Touch policy recommendations
- [ ] Troubleshooting guide
  - Common errors and solutions
  - Hardware compatibility issues
  - Platform-specific problems
- [ ] Contributing guide
  - Development setup
  - Coding standards
  - Pull request process
  - Security review requirements
- [ ] YubiKey model support matrix
  - Supported models and firmware
  - Feature availability by model
  - Known limitations
- [ ] Threat model documentation
  - Security assumptions
  - Attack scenarios
  - Mitigations implemented

### Release Preparation
- [ ] Version all components to 1.0.0
- [ ] Create CHANGELOG.md
- [ ] Create release notes
- [ ] Tag v1.0.0 in git
- [ ] Publish to PyPI (optional)
- [ ] Create Docker image (optional)
- [ ] Update README with badges (build status, coverage, version)

### Deliverables
- Complete documentation site (Sphinx HTML)
- v1.0.0 release
- PyPI package (optional)
- Docker image (optional)

### Success Criteria
- [ ] All public APIs documented
- [ ] Getting started guide complete
- [ ] All tests passing
- [ ] Security audit clean
- [ ] Version tagged in git
- [ ] Release announcement prepared

## Future Phases (Post-v1.0)

### Phase 6: Advanced Features
- Web UI for YubiKey management (Flask/FastAPI)
- Ansible playbook for fleet management
- Prometheus metrics exporter for monitoring
- Certificate lifecycle automation
- Integration with HashiCorp Vault
- LDAP/Active Directory integration
- Kubernetes operator for YubiKey management

### Phase 7: Enterprise Features
- Multi-tenant support with isolation
- Audit log aggregation and analysis
- Compliance reporting (FIPS 140-2, Common Criteria)
- SSO integration (SAML, OAuth)
- RBAC for multi-user environments
- Certificate authority integration
- Hardware security module (HSM) integration

### Phase 8: Platform Expansion
- Windows native support (currently WSL2 only)
- Mobile companion app (iOS/Android)
- Browser extension for FIDO2 management
- Cloud-based backup service
- Remote YubiKey management

## Success Criteria

### Quality Metrics
- [ ] 85%+ test coverage overall
- [ ] 95%+ coverage on security modules
- [ ] Zero security warnings from bandit
- [ ] All tests passing on CI/CD
- [ ] Documentation coverage 100% of public APIs
- [ ] Type hints on 100% of functions

### Security Metrics
- [ ] Zero hardcoded credentials
- [ ] All sensitive data sanitized in logs
- [ ] PIN validation enforced at all entry points
- [ ] Certificate chains validated cryptographically
- [ ] Attestation verification implemented for FIDO2
- [ ] Security audit passing

### Usability Metrics
- [ ] Clear error messages with remediation hints
- [ ] Comprehensive --help text for all tools
- [ ] Working examples in all guides
- [ ] < 10 minute getting started time for new users
- [ ] < 5 minute setup for yubikey-setup.sh workflow

### Performance Metrics
- [ ] YubiKey operations complete in < 5 seconds
- [ ] Batch operations scale linearly with device count
- [ ] Memory usage < 100MB per process
- [ ] CLI startup time < 1 second

## Risk Management

### Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Hardware availability | High | Low | Maintain test YubiKey inventory (min 3 devices) |
| Firmware compatibility | High | Medium | Test across YubiKey 4, 5, 5 FIPS |
| Platform differences | Medium | Medium | Test on Linux, macOS, WSL2 |
| Upstream API changes | Medium | Low | Pin dependency versions, monitor releases |
| Performance issues | Low | Low | Profile code, optimize hot paths |

### Security Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Credential exposure | Critical | Medium | Implement sanitization early, audit regularly |
| Test data leakage | High | Low | Use fixtures, never real production keys |
| Permission escalation | High | Low | Enforce least privilege, require explicit approval |
| Supply chain attacks | High | Low | Validate YubiKey authenticity, pin dependencies |
| Side-channel attacks | Medium | Very Low | Use constant-time operations where applicable |

### Project Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Scope creep | Medium | High | Strict phase boundaries, defer features to post-v1.0 |
| Schedule delays | Medium | Medium | Buffer time in estimates, prioritize ruthlessly |
| Resource constraints | High | Low | Single maintainer, leverage automation |
| Documentation debt | Medium | Medium | Write docs alongside code, not after |

## Resources

### Hardware Requirements
- YubiKey 5 NFC (minimum 3 for testing)
- YubiKey 5 FIPS (1 for FIPS testing)
- YubiKey 4 (1 for legacy support testing)
- Test fixtures documented with serial numbers in `tests/fixtures/TEST_DEVICES.md`

### Software Dependencies
- Python 3.9+ (recommend 3.11+ for performance)
- python-fido2, libfido2
- yubikey-manager (ykman) CLI and library
- PyUSB
- cryptography, PyNaCl
- pytest, pytest-cov, pytest-mock
- bandit, safety
- black, isort, flake8, mypy
- Sphinx for documentation
- Click or Typer for CLI

### Development Environment
- Linux (primary), macOS (secondary), WSL2 (tertiary)
- Git for version control
- GitHub for CI/CD
- Visual Studio Code or PyCharm (recommended IDEs)

### Documentation Resources
- Yubico Developer Documentation: https://developers.yubico.com/
- FIDO2/WebAuthn Specs: https://fidoalliance.org/fido2/
- PIV Standard (NIST SP 800-73-4): https://csrc.nist.gov/publications/detail/sp/800-73/4/final
- OpenPGP Card Spec: https://gnupg.org/ftp/specs/OpenPGP-smart-card-application-3.4.pdf

## Version History

| Version | Date | Description |
|---------|------|-------------|
| 0.1.0 | 2025-11-20 | Initial development plan created |

## Maintainers

- **Primary**: Elvis Nuno
- **Contributors**: Open to community contributions

## Review and Updates

This development plan should be reviewed and updated:
- Weekly during active development
- Monthly during maintenance phases
- After each phase completion
- When significant scope changes occur
- After security incidents or discoveries

**Last Updated**: 2025-11-20
**Next Review**: 2025-11-27
