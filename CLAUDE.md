<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Create a CLAUDE.md that follows best practices and is tailored for a repo dedicated to developing a collection of tools for configuring and managing yubikeys.

Based on my research into best practices for CLAUDE.md files, YubiKey management, and the Space's coding standards, I'll create a comprehensive CLAUDE.md tailored for a YubiKey tools repository.

```markdown
# YubiKey Configuration and Management Tools - Claude Instructions

## Project Overview

This repository contains a collection of specialized tools for configuring, managing, and automating YubiKey hardware security keys. The project focuses on providing developers and security administrators with reliable, well-tested utilities for YubiKey lifecycle management, batch configuration, security auditing, and integration with enterprise authentication systems.

**Primary Goal**: Deliver production-ready tools that simplify YubiKey deployment and management while maintaining the highest security standards.

**Technology Stack**:
- Primary Languages: Python 3.9+, Bash
- Key Dependencies: python-fido2, libfido2, yubikey-manager (ykman), PyUSB
- Testing: pytest, pytest-cov, unittest
- Security Standards: FIDO2, WebAuthn, PIV, OTP, OpenPGP
- Documentation: Sphinx, Markdown

**Architecture Pattern**: Modular CLI tools with shared library components for common YubiKey operations.

---

## Code Discovery

### Repository Structure
```

yubikey-tools/
├── src/
│   ├── core/           \# Shared library for YubiKey operations
│   ├── tools/          \# Individual CLI utilities
│   ├── validators/     \# Security validation modules
│   └── utils/          \# Helper functions and common utilities
├── tests/
│   ├── unit/           \# Unit tests for each module
│   ├── integration/    \# Integration tests with hardware
│   └── fixtures/       \# Test data and mocks
├── docs/
│   ├── api/            \# API documentation
│   ├── guides/         \# User guides and tutorials
│   └── security/       \# Security considerations
├── scripts/
│   └── automation/     \# Deployment and automation scripts
└── .claude/
├── commands/       \# Custom slash commands
└── hooks/          \# Lifecycle hooks

```

### Naming Conventions
- **Python modules**: `snake_case` (e.g., `yubikey_config.py`, `piv_manager.py`)
- **Python classes**: `PascalCase` (e.g., `YubiKeyManager`, `PIVCertificateHandler`)
- **Python functions/variables**: `snake_case` (e.g., `configure_oath_credentials`, `slot_number`)
- **CLI tools**: `kebab-case` (e.g., `yubikey-batch-config`, `piv-cert-manager`)
- **Test files**: `test_<module_name>.py` (e.g., `test_yubikey_config.py`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `MAX_RETRY_ATTEMPTS`, `DEFAULT_PIN_LENGTH`)

### File Search Patterns
- Configuration files: Look in `src/core/config/` for YAML/JSON config schemas
- YubiKey operation modules: Check `src/core/operations/` for FIDO2, PIV, OTP, OATH handlers
- Security validators: Find in `src/validators/` for attestation, cert validation, policy enforcement
- CLI entry points: Located in `src/tools/` with `__main__.py` blocks or Click/Typer decorators

---

## Code Editing Standards

### Style Guide
- **Python**: Follow PEP 8 strictly, enforced via `black` (line length: 88) and `flake8`
- **Type hints**: Mandatory for all function signatures (Python 3.9+ style)
- **Docstrings**: Google-style docstrings for all public functions, classes, and modules
- **Imports**: Organized with `isort` (stdlib, third-party, local)

### Security-Specific Coding Standards

**Critical**: This project handles hardware security keys and sensitive cryptographic operations. Security is paramount.

1. **Never log or print sensitive data**:
   - PINs, PUKs, management keys, private keys, or any credential material
   - Use `[REDACTED]` or `***` in logs when referencing sensitive operations
   - Implement `@sanitize_logging` decorator for sensitive functions

2. **Input validation**:
   - Validate all user inputs before passing to YubiKey operations
   - Enforce PIN/PUK length and complexity requirements
   - Validate certificate chains and attestations cryptographically

3. **Error handling**:
   - Never expose internal implementation details in error messages
   - Catch hardware exceptions gracefully (disconnection, timeout, access denied)
   - Provide clear, actionable error messages for users without security details

4. **Privilege management**:
   - Request minimum necessary permissions
   - Implement role-based access controls for multi-user tools
   - Document required permissions in function docstrings

5. **Cryptographic operations**:
   - Use established libraries (cryptography, PyNaCl) - never roll your own crypto
   - Verify attestation chains for FIDO2 operations
   - Validate certificate serial numbers and expiry dates for PIV

### Code Example Template

```

from typing import Optional, List
from dataclasses import dataclass
from yubikey_manager import YubiKey
from .validators import validate_pin_format, sanitize_log_output

@dataclass
class PIVSlotConfig:
"""Configuration for a PIV slot on a YubiKey.

    Attributes:
        slot: PIV slot number (0x9a, 0x9c, 0x9d, 0x9e)
        key_algorithm: Algorithm for key generation (RSA2048, ECCP256, etc.)
        subject_dn: Distinguished Name for certificate subject
        pin_policy: PIN policy (DEFAULT, NEVER, ONCE, ALWAYS)
        touch_policy: Touch policy (DEFAULT, NEVER, ALWAYS, CACHED)
    """
    slot: int
    key_algorithm: str
    subject_dn: str
    pin_policy: str = "DEFAULT"
    touch_policy: str = "DEFAULT"
    def configure_piv_slot(
yubikey: YubiKey,
config: PIVSlotConfig,
pin: str,
management_key: Optional[bytes] = None
) -> bool:
"""Configure a PIV slot with the specified parameters.

    Args:
        yubikey: Connected YubiKey instance
        config: PIV slot configuration
        pin: User PIN for authentication (will be redacted in logs)
        management_key: Optional management key (will use default if None)
    
    Returns:
        True if configuration succeeded, False otherwise
    
    Raises:
        ValueError: If PIN format is invalid
        YubiKeyConnectionError: If YubiKey is disconnected during operation
        
    Security:
        - PIN is never logged or stored
        - Requires admin privileges for management key operations
        - Validates certificate chain if importing existing cert
    """
    # Validate inputs before hardware interaction
    if not validate_pin_format(pin):
        raise ValueError("PIN must be 6-8 digits")
    
    try:
        # Authenticate with redacted logging
        logger.info(f"Configuring PIV slot {hex(config.slot)}")
        yubikey.authenticate(pin)  # PIN not logged
        
        # Perform operation
        result = yubikey.piv.generate_key(
            slot=config.slot,
            algorithm=config.key_algorithm,
            pin_policy=config.pin_policy,
            touch_policy=config.touch_policy
        )
        
        logger.info(f"Successfully configured slot {hex(config.slot)}")
        return True
        
    except Exception as e:
        # Sanitize error before logging
        safe_error = sanitize_log_output(str(e))
        logger.error(f"Failed to configure PIV slot: {safe_error}")
        return False
    ```

### Follow-Up Actions After Code Changes

1. **Run tests immediately**:
```

pytest tests/unit/test_<modified_module>.py -v

```

2. **Check security validators** if modifying:
- Authentication flows
- Certificate operations
- PIN/PUK handling
- Attestation verification

3. **Update documentation**:
- API docs if public interface changed
- Security considerations if new risks introduced
- User guides if CLI behavior changed

4. **Run security audit** for sensitive changes:
```

/security-audit

```

---

## Code Quality Standards

### Testing Philosophy

**Principle**: Security tools demand exceptional reliability. All code must be thoroughly tested.

- **Minimum coverage**: 85% overall, 95% for core security modules
- **Test pyramid**: 70% unit, 25% integration, 5% end-to-end
- **Hardware testing**: Use mock YubiKeys for CI/CD, real hardware for pre-release validation
- **Security testing**: Include negative tests for attack scenarios (invalid PINs, forged attestations)

### Testing Commands

```


# Run all tests with coverage

pytest --cov=src --cov-report=html --cov-report=term

# Run only unit tests (no hardware required)

pytest tests/unit/ -v

# Run integration tests (requires YubiKey connected)

pytest tests/integration/ -v --hardware

# Run security-focused tests

pytest -m security -v

# Check for common security issues

bandit -r src/ -ll

# Type checking

mypy src/ --strict

```

### Linting and Formatting

```


# Auto-format code

black src/ tests/
isort src/ tests/

# Check code quality

flake8 src/ tests/ --max-line-length=88
pylint src/ --rcfile=.pylintrc

# Security linting

bandit -r src/ -f screen

```

### Pre-Commit Checklist

Before committing any code:
- [ ] All tests pass locally
- [ ] Code coverage meets minimum thresholds
- [ ] No security warnings from bandit
- [ ] Black and isort formatting applied
- [ ] Type hints added for new functions
- [ ] Docstrings updated
- [ ] Sensitive data sanitized in logs
- [ ] Integration tests pass with real hardware (if applicable)

---

## Tool Usage Permissions

### Allowed Operations

**File System**:
- ✅ Read/write in `src/`, `tests/`, `docs/`, `scripts/`
- ✅ Create new modules following naming conventions
- ✅ Modify configuration files in `.claude/`
- ✅ Generate documentation in `docs/`

**Git Operations**:
- ✅ Stage changes with descriptive commit messages
- ✅ Create feature branches following pattern: `feature/<issue-number>-<short-description>`
- ✅ Create bugfix branches: `bugfix/<issue-number>-<short-description>`
- ✅ View git history and status

**Testing & Validation**:
- ✅ Run pytest suites
- ✅ Execute linting and formatting tools
- ✅ Generate coverage reports
- ✅ Run security scanners (bandit, safety)

**YubiKey Operations** (with explicit user approval):
- ⚠️  Enumerate connected YubiKeys for testing
- ⚠️  Read YubiKey serial numbers and firmware versions
- ⚠️  Execute operations on test YubiKeys only (never production keys)

### Restricted Operations

**Require Explicit Approval**:
- 🔒 Any operation that writes to a YubiKey (config changes, key generation, PIN resets)
- 🔒 Operations requiring management key or admin PIN
- 🔒 Certificate installation or removal
- 🔒 Firmware updates or device resets
- 🔒 Batch operations affecting multiple YubiKeys
- 🔒 Publishing packages to PyPI or other registries
- 🔒 Modifying CI/CD pipeline configurations

**Never Allowed**:
- ❌ Logging or storing PINs, PUKs, or management keys
- ❌ Committing sensitive credentials to git
- ❌ Disabling security validators without documented justification
- ❌ Reducing test coverage thresholds
- ❌ Operations on production YubiKeys without explicit authorization

---

## Git Workflow

### Branch Strategy

- **main**: Production-ready code, protected branch
- **develop**: Integration branch for features
- **feature/***: New features and enhancements
- **bugfix/***: Bug fixes
- **security/***: Security patches (high priority)
- **docs/***: Documentation improvements

### Commit Message Format

Follow conventional commits:

```

<type>(<scope>): <subject>

<body>

<footer>

```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `security`: Security improvement or patch
- `docs`: Documentation changes
- `test`: Test additions or modifications
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `ci`: CI/CD changes

**Examples**:
```

feat(piv): add support for ECC P-384 key generation

Implement ECC P-384 algorithm support for PIV key generation
across all supported slots. Includes attestation validation.

Closes \#123

security(validators): strengthen PIN validation regex

Add additional validation to prevent PIN bypass via special
characters. Implements OWASP recommendations.

CVE: None (proactive hardening)

```

### Pull Request Requirements

All PRs must include:
1. Clear description of changes
2. Test results showing coverage maintained/improved
3. Security impact assessment for sensitive changes
4. Updated documentation if API changed
5. Passing CI/CD pipeline
6. At least one approving review for main branch merges

---

## YubiKey-Specific Development Guidelines

### Hardware Interaction Patterns

1. **Always handle disconnection gracefully**:
```

try:
with YubiKeyConnection() as yk:
\# Perform operations
pass
except YubiKeyConnectionError:
logger.error("YubiKey disconnected during operation")
\# Cleanup and user-friendly error message

```

2. **Implement timeouts for user touch requirements**:
- Default: 30 seconds for touch operations
- Provide clear user feedback: "Touch your YubiKey to continue..."

3. **Support multiple YubiKeys connected simultaneously**:
- Enumerate and allow user selection
- Use serial numbers for identification (not USB path)

4. **Validate firmware compatibility**:
- Check minimum firmware version for features
- Gracefully degrade or provide clear error for unsupported operations

### Common YubiKey Operations Reference

```


# Get YubiKey info

from ykman.device import list_all_devices

devices = list_all_devices()
for device in devices:
print(f"Serial: {device.serial}, Firmware: {device.version}")

# FIDO2 credential management

from fido2.hid import CtapHidDevice
from fido2.ctap2 import Ctap2

dev = next(CtapHidDevice.list_devices())
ctap2 = Ctap2(dev)

# PIV operations

from yubikey_manager.piv import PivController

piv = PivController(yubikey)
piv.authenticate(management_key)
piv.generate_key(slot=0x9a, algorithm=KEY_TYPE.RSA2048)

# OATH (TOTP/HOTP)

from yubikey_manager.oath import OathController

oath = OathController(yubikey)
credentials = oath.list_credentials()

```

---

## Security Considerations

### Threat Model

This project must defend against:
1. **Supply chain attacks**: Validate YubiKey authenticity via attestation
2. **Credential theft**: Never store or log sensitive material
3. **Privilege escalation**: Enforce least-privilege access
4. **Man-in-the-middle**: Validate certificate chains
5. **Physical attacks**: Assume attacker may have physical access to YubiKey

### Security Review Process

All code touching these areas requires security-focused review:
- Authentication and authorization logic
- Cryptographic operations
- Certificate validation
- PIN/PUK handling
- Attestation verification
- Batch operations (prevent accidental mass misconfiguration)

### Security Testing Requirements

Include tests for:
- ✅ Invalid PIN formats and lengths
- ✅ Expired certificates
- ✅ Malformed attestation chains
- ✅ Replay attack scenarios
- ✅ Race conditions in concurrent operations
- ✅ Buffer overflow attempts in inputs
- ✅ SQL injection in logging/storage (if applicable)

---

## Documentation Standards

### Code Documentation

- **All public APIs**: Google-style docstrings with examples
- **Security-sensitive functions**: Include "Security:" section in docstring
- **Complex algorithms**: Inline comments explaining logic
- **Configuration options**: Document all parameters, defaults, and constraints

### User Documentation

Maintain these guides in `docs/`:
1. **Getting Started**: Installation, basic configuration
2. **CLI Reference**: All tools with examples
3. **Security Best Practices**: YubiKey lifecycle management, backup strategies
4. **API Documentation**: Generated from docstrings via Sphinx
5. **Troubleshooting**: Common issues and solutions
6. **Contributing Guide**: How to contribute securely

### Security Documentation

Document in `docs/security/`:
- Threat model and mitigations
- Security architecture decisions
- Vulnerability reporting process
- Security testing procedures
- Compliance considerations (FIPS, Common Criteria)

---

## Collaboration and Agent Patterns

### Agent Roles for This Project

When working with multiple AI agents or sessions:

- **Architect**: Design new tools, plan integrations, security architecture
- **Builder**: Implement features, write tests, integrate libraries
- **Validator**: Review security, test edge cases, validate attestations
- **Scribe**: Document APIs, write guides, maintain security docs
- **Researcher**: Investigate YubiKey APIs, security standards, best practices

### Multi-Agent Workflow Example

For implementing a new YubiKey configuration tool:

1. **Architect** designs the tool interface and security requirements
2. **Builder** implements core functionality with tests
3. **Validator** performs security review and penetration testing
4. **Scribe** creates user documentation and API references
5. **Researcher** validates approach against Yubico developer guidelines

### Context Handoff Pattern

When transitioning between agents/sessions:
1. Summarize current state and completed work
2. Document any security considerations discovered
3. List remaining tasks and dependencies
4. Provide relevant file paths and test commands
5. Note any hardware requirements (specific YubiKey models, firmware versions)

---

## Quick Reference

### Essential Commands

```


# Setup development environment

python -m venv venv
source venv/bin/activate  \# or `venv\Scripts\activate` on Windows
pip install -e .[dev]

# Run full test suite

pytest --cov=src --cov-report=term-missing

# Security audit

bandit -r src/ -ll
safety check

# Format code

black src/ tests/
isort src/ tests/

# Generate documentation

cd docs \&\& make html

# List connected YubiKeys

ykman list

# Check YubiKey info (non-destructive)

ykman info

```

### Important Configuration Files

- `.claude/settings.json`: Claude Code configuration
- `pyproject.toml`: Python project metadata, dependencies
- `pytest.ini`: Test configuration
- `.bandit`: Security linting exclusions
- `mypy.ini`: Type checking configuration

### Critical Paths

- Core library: `src/core/`
- CLI tools: `src/tools/`
- Security validators: `src/validators/`
- Integration tests: `tests/integration/`
- Security documentation: `docs/security/`

---

## External Resources

### Yubico Official Documentation
- [YubiKey Manager (ykman)](https://developers.yubico.com/yubikey-manager/)
- [PIV Tool Documentation](https://developers.yubico.com/yubico-piv-tool/)
- [FIDO2/WebAuthn Guide](https://developers.yubico.com/FIDO2/)
- [YubiKey SDK Documentation](https://developers.yubico.com/Developer_Program/)

### Standards and Specifications
- FIDO2/WebAuthn: https://fidoalliance.org/fido2/
- PIV (NIST SP 800-73): https://csrc.nist.gov/publications/detail/sp/800-73/4/final
- OpenPGP Card: https://gnupg.org/ftp/specs/OpenPGP-smart-card-application-3.4.pdf

### Security Best Practices
- OWASP Secure Coding Practices: https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/
- CWE Top 25: https://cwe.mitre.org/top25/

---

## Project-Specific Notes

### YubiKey Model Support Matrix

Document which YubiKey models are supported for each feature:
- YubiKey 5 Series: Full feature support
- YubiKey 5 FIPS: FIPS-mode considerations
- Security Key Series: Limited to FIDO2/U2F
- YubiKey 4 Series: Legacy support (if applicable)

### Testing with Hardware

- **CI/CD**: Uses mock YubiKey devices
- **Pre-release**: Requires testing with physical YubiKeys across supported models
- **Destructive tests**: Isolated to test-designated YubiKeys only (serial numbers documented)

### Known Limitations

- Some operations require physical touch - cannot be fully automated
- Firmware updates not supported programmatically (Yubico restriction)
- Windows requires elevated privileges for some USB operations

---

## Continuous Improvement

This `CLAUDE.md` file should be updated when:
- New YubiKey features are supported
- Security vulnerabilities are discovered and patched
- Development workflow changes
- New tools or testing strategies are adopted
- Community feedback suggests improvements

**Last Updated**: 11/20/2025
**Maintained By**: Elvis Nuno 
**Review Cycle**: Quarterly or upon significant YubiKey SDK updates
```

This CLAUDE.md file provides comprehensive guidance tailored specifically for a YubiKey tools repository while following the best practices from your Space's coding standards. It emphasizes:[^1][^2][^3]

1. **Security-first approach** - Critical for hardware security key management
2. **Detailed code quality standards** - Mandatory testing, linting, type hints
3. **Clear permission boundaries** - What AI agents can/cannot do with YubiKeys
4. **YubiKey-specific patterns** - Hardware interaction best practices
6. **Multi-agent collaboration** - Role definitions for complex workflows

The structure follows the recommended patterns from the Space files while adding YubiKey-specific security requirements and development guidelines.
