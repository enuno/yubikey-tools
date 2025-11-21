# YubiKey Tools - Task List

Last Updated: 2025-11-20

## Immediate Tasks (This Week)

### Repository Structure ✅
- [x] Create src/ directory with subdirectories (core, tools, validators, utils)
- [x] Create tests/ directory with subdirectories (unit, integration, fixtures)
- [x] Create docs/ directory with subdirectories (api, guides, security)
- [x] Create scripts/automation/ directory
- [x] Move yubikey-setup/ to scripts/automation/
- [x] Create symlink to yubikey-setup.sh in project root
- [x] Create .claude/hooks/ directory

### Python Project Setup
- [ ] Create pyproject.toml with project metadata
  - Project name, version, description
  - Author and license information
  - Minimum Python version (3.9+)
  - Entry points for CLI tools
- [ ] Define dependencies in pyproject.toml
  - python-fido2 (FIDO2 operations)
  - yubikey-manager (ykman integration)
  - PyUSB (USB communication)
  - cryptography (cryptographic operations)
  - click or typer (CLI framework)
- [ ] Define dev dependencies in pyproject.toml
  - pytest, pytest-cov, pytest-mock (testing)
  - black, isort (formatting)
  - flake8, pylint (linting)
  - mypy (type checking)
  - bandit (security linting)
  - safety (dependency scanning)
  - sphinx, sphinx-rtd-theme (documentation)
- [ ] Create setup.py or setup.cfg (if needed for compatibility)
- [ ] Create .gitignore for Python projects
  - __pycache__/, *.py[cod], *.so
  - .pytest_cache/, .coverage, htmlcov/
  - dist/, build/, *.egg-info/
  - .venv/, venv/, env/
  - .mypy_cache/, .bandit
  - .DS_Store (macOS)
  - *.key, *.pem, *.gpg (sensitive files)
- [ ] Initialize pytest.ini configuration
  - Test discovery patterns
  - Coverage settings
  - Hardware test markers
  - Warning filters

### __init__.py Files
- [ ] Create src/__init__.py
- [ ] Create src/core/__init__.py
- [ ] Create src/core/config/__init__.py
- [ ] Create src/core/operations/__init__.py
- [ ] Create src/tools/__init__.py
- [ ] Create src/validators/__init__.py
- [ ] Create src/utils/__init__.py
- [ ] Create tests/__init__.py
- [ ] Create tests/unit/__init__.py
- [ ] Create tests/integration/__init__.py

### Development Environment Configuration
- [ ] Set up black configuration in pyproject.toml
  - Line length: 88
  - Target Python version: 3.9+
  - Exclude patterns
- [ ] Set up isort configuration in pyproject.toml
  - Profile: black (for compatibility)
  - Line length: 88
  - Import order: stdlib, third-party, local
- [ ] Set up flake8 configuration (.flake8)
  - Max line length: 88
  - Ignore compatibility with black
  - Exclude directories
- [ ] Set up mypy configuration (mypy.ini or pyproject.toml)
  - Strict mode enabled
  - Disallow untyped defs
  - Warn return any
- [ ] Set up bandit configuration (.bandit or pyproject.toml)
  - Exclude test directories
  - Medium/high severity only
- [ ] Create .pre-commit-config.yaml
  - black hook
  - isort hook
  - flake8 hook
  - mypy hook (optional in pre-commit)
  - trailing whitespace
  - end-of-file fixer

### Documentation ✅ (Partial)
- [x] Create AGENTS.md
- [x] Create DEVELOPMENT_PLAN.md
- [x] Create TODO.md (this file)
- [ ] Create docs/guides/getting-started.md
- [ ] Create docs/security/threat-model.md
- [ ] Create docs/security/testing-protocol.md
- [ ] Update README.md with repository structure
- [ ] Create CONTRIBUTING.md
- [ ] Create CHANGELOG.md
- [ ] Initialize Sphinx for API docs (docs/api/)
  - conf.py configuration
  - index.rst main page
  - API reference structure

### CI/CD
- [ ] Create .github/workflows/tests.yml
  - Run pytest on all Python versions (3.9, 3.10, 3.11, 3.12)
  - Generate coverage reports
  - Upload to codecov (optional)
- [ ] Create .github/workflows/lint.yml
  - Run black --check
  - Run isort --check
  - Run flake8
  - Run mypy
- [ ] Create .github/workflows/security.yml
  - Run bandit
  - Run safety check
  - Check for secrets in code
- [ ] Set up GitHub branch protection rules
  - Require CI passing
  - Require code review
  - No force pushes to main

## Near-Term Tasks (Next 2 Weeks)

### Core Library - Detection & Connection
- [ ] Implement src/core/operations/yubikey_detection.py
  - [ ] `list_yubikeys()` - Enumerate connected devices
  - [ ] `get_yubikey_info(serial)` - Get device information
  - [ ] `check_firmware_version(serial, min_version)` - Validate firmware
  - [ ] Unit tests with mocks (90%+ coverage)
- [ ] Implement src/utils/connection_manager.py
  - [ ] `YubiKeyConnection` context manager
  - [ ] Automatic cleanup on errors
  - [ ] Connection timeout handling
  - [ ] Unit tests

### Core Library - FIDO2
- [ ] Implement src/core/operations/fido2_operations.py
  - [ ] `list_fido2_credentials(device)` - List resident credentials
  - [ ] `create_credential(device, rp, user)` - Create new credential
  - [ ] `delete_credential(device, credential_id)` - Delete credential
  - [ ] `verify_attestation(attestation)` - Validate attestation chain
  - [ ] Unit tests with mock FIDO2 device

### Core Library - PIV
- [ ] Implement src/core/operations/piv_operations.py
  - [ ] `generate_piv_key(device, slot, algorithm)` - Generate key in slot
  - [ ] `import_certificate(device, slot, cert)` - Import certificate
  - [ ] `export_certificate(device, slot)` - Export certificate
  - [ ] `list_piv_slots(device)` - List configured slots
  - [ ] Unit tests with mock PIV controller

### Core Library - OATH
- [ ] Implement src/core/operations/oath_operations.py
  - [ ] `list_oath_credentials(device)` - List TOTP/HOTP
  - [ ] `add_oath_credential(device, name, secret)` - Add credential
  - [ ] `delete_oath_credential(device, name)` - Remove credential
  - [ ] `generate_totp(device, name)` - Generate TOTP code
  - [ ] Unit tests

### Validators
- [ ] Implement src/validators/pin_validator.py
  - [ ] `validate_pin_format(pin: str) -> bool` - Format validation
  - [ ] `validate_pin_complexity(pin: str) -> bool` - Complexity check
  - [ ] `validate_puk_format(puk: str) -> bool` - PUK validation
  - [ ] Unit tests with edge cases (empty, too short, too long, non-numeric)
- [ ] Implement src/validators/certificate_validator.py
  - [ ] `validate_certificate_chain(cert_chain)` - Chain validation
  - [ ] `check_certificate_expiry(cert)` - Expiry check
  - [ ] `validate_certificate_usage(cert, usage)` - Key usage validation
  - [ ] Unit tests with valid and invalid certificates

### Validators - Security
- [ ] Implement src/validators/attestation_validator.py
  - [ ] `validate_fido2_attestation(attestation)` - FIDO2 attestation
  - [ ] `verify_attestation_signature(attestation, public_key)` - Signature verification
  - [ ] Unit tests with sample attestations
- [ ] Implement src/validators/policy_validator.py
  - [ ] `validate_touch_policy(policy)` - Touch policy validation
  - [ ] `validate_pin_policy(policy)` - PIN policy validation
  - [ ] Unit tests

### Utils
- [ ] Implement src/utils/logging_utils.py
  - [ ] `@sanitize_logging` decorator - Sanitize function outputs
  - [ ] `sanitize_log_output(text: str) -> str` - Redact sensitive data
  - [ ] `get_logger(name: str)` - Get configured logger
  - [ ] Unit tests for redaction patterns (PINs, keys, secrets)
- [ ] Implement src/utils/error_handling.py
  - [ ] `YubiKeyConnectionError` exception
  - [ ] `YubiKeyAuthenticationError` exception
  - [ ] `YubiKeyOperationError` exception
  - [ ] `YubiKeyNotFoundError` exception
  - [ ] Exception hierarchy and documentation

### Configuration
- [ ] Implement src/core/config/defaults.py
  - [ ] Default PIN/PUK lengths
  - [ ] Timeout values
  - [ ] Retry counters
  - [ ] Algorithm defaults
- [ ] Implement src/core/config/config_schema.py
  - [ ] `PIVSlotConfig` dataclass
  - [ ] `FIDO2Config` dataclass
  - [ ] `OATHConfig` dataclass
  - [ ] Validation methods

### Custom Commands ✅ (Partial)
- [ ] Create .claude/commands/security-audit.md
- [ ] Create .claude/commands/yubikey-enum.md
- [ ] Create .claude/commands/test-hardware.md
- [ ] Create .claude/commands/validate-crypto.md
- [ ] Create .claude/commands/check-compliance.md
- [ ] Create .claude/commands/yubikey-backup-verify.md
- [ ] Create .claude/commands/pin-security-check.md

### Custom Agents ✅ (Partial)
- [ ] Create .claude/agents/security-validator.md
- [ ] Create .claude/agents/hardware-tester.md
- [ ] Create .claude/agents/crypto-reviewer.md

### Test Infrastructure
- [ ] Create tests/fixtures/TEST_DEVICES.md
  - Document test YubiKey serial numbers
  - Document firmware versions
  - Document expected configurations
- [ ] Create tests/fixtures/mock_yubikey.py
  - Mock YubiKey class for unit tests
  - Simulate YubiKey operations
  - Configurable responses
- [ ] Create tests/conftest.py
  - pytest fixtures for YubiKey mocks
  - Hardware test markers
  - Skip conditions for integration tests

## Medium-Term Tasks (Month 2)

### CLI Tools Architecture
- [ ] Research CLI framework (Click vs Typer)
- [ ] Design common CLI patterns
  - Verbose/debug output
  - JSON output mode
  - Dry-run capability
  - Progress indicators
- [ ] Create CLI tool template
- [ ] Implement shared CLI utilities

### CLI Tools - Implementation
- [ ] Implement yubikey-audit tool
  - Check PIN retry counters
  - Validate firmware versions
  - Audit certificate expiry
  - Generate reports
- [ ] Implement yubikey-piv-manager tool
  - Key generation
  - Certificate management
  - Slot configuration
- [ ] Implement yubikey-fido2-manager tool
  - Credential management
  - PIN management
  - Attestation retrieval
- [ ] Implement yubikey-oath-manager tool
  - Credential management
  - Code generation
  - QR code support
- [ ] Implement yubikey-batch-config tool
  - Batch configuration from YAML
  - Multi-device support
  - Dry-run mode
- [ ] Implement yubikey-backup tool
  - Export configuration
  - Timestamped backups
  - Integrity verification

### Integration Tests
- [ ] Set up test YubiKey fixtures
  - Procure test devices
  - Document serial numbers
  - Reset to known state
- [ ] Create integration test suite
  - Test detection and enumeration
  - Test FIDO2 operations
  - Test PIV operations
  - Test OATH operations
- [ ] Add hardware test marker in pytest
  - `@pytest.mark.hardware`
  - Skip if no hardware available
  - Document hardware requirements

### Documentation - Guides
- [ ] Write docs/guides/getting-started.md
  - Installation instructions
  - Quick start examples
  - Common workflows
- [ ] Write docs/guides/cli-reference.md
  - Each tool documented
  - Command examples
  - Options and flags
- [ ] Write docs/security/best-practices.md
  - YubiKey lifecycle management
  - Backup strategies
  - PIN/PUK management
  - Touch policy recommendations
- [ ] Write docs/security/troubleshooting.md
  - Common errors and solutions
  - Hardware compatibility issues
  - Platform-specific problems

### Security Testing
- [ ] Implement comprehensive security test suite
  - Test invalid PIN formats
  - Test expired certificates
  - Test malformed attestations
  - Test replay attack scenarios
- [ ] Add fuzzing tests for input validation
  - Fuzz PIN inputs
  - Fuzz configuration files
  - Fuzz certificate inputs
- [ ] Document security architecture decisions
  - Threat model
  - Mitigations
  - Assumptions

## Long-Term Tasks (Month 3+)

### Advanced Features
- [ ] Design batch configuration tool architecture
- [ ] Implement certificate lifecycle automation
- [ ] Create Ansible playbook integration
- [ ] Design web UI (optional)
  - Flask or FastAPI backend
  - React or Vue.js frontend
  - WebAuthn integration

### Enterprise Features
- [ ] Design multi-tenant architecture
- [ ] Implement audit log aggregation
- [ ] Create compliance reporting
  - FIPS 140-2 compliance
  - Common Criteria alignment
- [ ] Implement RBAC for multi-user environments

### Documentation - API and Release
- [ ] Generate Sphinx API documentation
  - All modules documented
  - Code examples
  - Cross-references
- [ ] Create YubiKey model support matrix
  - Supported models
  - Feature availability
  - Known limitations
- [ ] Write CONTRIBUTING.md
  - Development setup
  - Coding standards
  - PR process

### Release Preparation
- [ ] Prepare v1.0.0 release notes
- [ ] Version all components to 1.0.0
- [ ] Create CHANGELOG.md with all changes
- [ ] Tag v1.0.0 in git
- [ ] Publish to PyPI (optional)
- [ ] Create Docker image (optional)
- [ ] Update README with badges

## Blocked Tasks

_No blocked tasks currently_

## Completed Tasks

### Session 2025-11-20
- [x] Created directory structure per CLAUDE.md
- [x] Moved yubikey-setup/ to scripts/automation/
- [x] Created symlink to yubikey-setup.sh
- [x] Created AGENTS.md
- [x] Created DEVELOPMENT_PLAN.md
- [x] Created TODO.md (this file)

## Notes

### Critical Reminders
- All tasks involving YubiKey write operations require explicit user approval
- Test YubiKeys must be documented in tests/fixtures/TEST_DEVICES.md
- Security-critical changes require Security Validator agent review
- Never log or store PINs, PUKs, or management keys
- Use @sanitize_logging decorator on all sensitive functions

### Testing Philosophy
- Minimum 85% overall coverage, 95% for security modules
- Mock YubiKeys for unit tests (fast, no hardware)
- Real YubiKeys for integration tests (slow, requires hardware)
- Security tests for attack scenarios (negative testing)

### Code Quality Standards
- Type hints mandatory for all functions
- Google-style docstrings for all public APIs
- Black formatting (line length 88)
- Zero security warnings from bandit
- All tests passing before merge

## Priority Legend
- **P0**: Critical, blocking other work
- **P1**: High priority, needed soon
- **P2**: Medium priority, nice to have
- **P3**: Low priority, future enhancement

Current priorities focus on P0 and P1 tasks in "Immediate Tasks" and "Near-Term Tasks" sections.
