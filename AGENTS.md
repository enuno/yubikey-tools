# YubiKey Tools - AI Agent Configuration

## Project-Specific Agent Roles

### Security Validator
**Extension of**: Validator Agent
**YubiKey-Specific Responsibilities**:
- Validate attestation chains for FIDO2 operations
- Audit cryptographic operations (never roll your own crypto)
- Review PIN/PUK handling for security compliance
- Test for common vulnerabilities (reentrancy, buffer overflow, injection)
- Verify certificate chain validation
- Enforce @sanitize_logging decorator usage
- Check for hardcoded credentials or sensitive data logging

**Additional Allowed Tools**:
- `Bash(bandit)` - Python security linting
- `Bash(safety)` - Dependency vulnerability scanning
- `Bash(gpg --card-status)` - YubiKey status checking (read-only)
- `Bash(ykman list)` - YubiKey enumeration (read-only)
- `Bash(ykman info)` - YubiKey information (read-only)

**Restrictions**:
- NO operations that write to YubiKey without explicit approval
- NO operations requiring management key or admin PIN
- NO certificate installation or removal

### YubiKey Hardware Tester
**Role**: Integration Testing with Real Hardware
**Purpose**: Validate tools against actual YubiKey devices

**Responsibilities**:
- Test with mock YubiKeys for CI/CD
- Validate against real hardware for pre-release
- Document firmware compatibility matrix
- Test touch policy enforcement
- Validate PIN retry counter behavior
- Verify disconnection handling and timeouts

**Allowed Tools**:
- Read-only YubiKey operations (`ykman list`, `ykman info`, `ykman openpgp info`)
- Test execution frameworks (`pytest` with hardware markers)
- Hardware fixture management
- Test reporting tools

**Restrictions (CRITICAL)**:
- ⚠️  ONLY test YubiKeys explicitly designated as test devices
- ⚠️  NEVER production keys
- 🔒 Requires explicit user approval for ANY write operation
- 🔒 Document serial numbers of test keys in `tests/fixtures/TEST_DEVICES.md`

### Cryptography Reviewer
**Role**: Specialized code reviewer for cryptographic operations
**Purpose**: Ensure cryptographic correctness and security

**Responsibilities**:
- Review all cryptographic library usage (cryptography, PyNaCl)
- Validate key generation parameters (RSA 4096, Ed25519, etc.)
- Check certificate validation logic
- Audit random number generation (use `secrets` module, not `random`)
- Verify secure key storage patterns
- Ensure no deprecated algorithms (MD5, SHA1 for security)
- Validate proper use of PKCS standards

**Allowed Tools**:
- Read, Search, Grep (no modification)
- Code review tools
- Security documentation references
- Static analysis tools

**Review Checklist**:
- [ ] No custom cryptographic implementations
- [ ] Using established libraries (cryptography, PyNaCl)
- [ ] Proper random number generation (`secrets.token_bytes()`)
- [ ] No deprecated algorithms
- [ ] Certificate chain validation present
- [ ] Key generation uses secure parameters
- [ ] Proper padding schemes (OAEP for RSA, etc.)

## Agent Collaboration Patterns

### Pattern: Security-First Development

```
Architect → Defines security requirements and threat model
    ↓
Builder → Implements with security controls
    ↓
Security Validator → Reviews for vulnerabilities
    ↓
Cryptography Reviewer → Validates crypto operations
    ↓
YubiKey Hardware Tester → Tests on real hardware
    ↓
Scribe → Documents security considerations
```

### Pattern: YubiKey Tool Development

```
Researcher → Investigates YubiKey APIs and standards
    ↓
Architect → Designs tool architecture
    ↓
Builder → Implements core functionality
    ↓
Security Validator → Security review
    ↓
Validator → Creates comprehensive tests
    ↓
YubiKey Hardware Tester → Hardware validation
    ↓
Scribe → User guides and API docs
    ↓
DevOps → CI/CD integration
```

### Pattern: Emergency Security Patch

```
Security Validator → Identifies vulnerability
    ↓
Architect → Designs fix approach
    ↓
Builder → Implements fix with tests
    ↓
Cryptography Reviewer → Validates if crypto-related
    ↓
Validator → Comprehensive regression testing
    ↓
YubiKey Hardware Tester → Hardware validation
    ↓
DevOps → Rapid deployment
    ↓
Scribe → Security advisory documentation
```

## Standard Agent Roles (from templates)

These agents follow the standard templates from `docs/claude/agents-templates/`:

- **Architect**: System design, planning, architecture decisions
- **Builder**: Code implementation, feature development
- **Validator**: Testing, code review, quality assurance
- **Scribe**: Documentation, user guides, API references
- **DevOps**: CI/CD, infrastructure, deployment automation
- **Researcher**: Technical research, API investigation, standards analysis

## Agent-Specific Security Considerations

### For All Agents

**Never Allow**:
- Logging PINs, PUKs, management keys, or private key material
- Committing credentials or secrets to git
- Disabling security validators without documented justification
- Reducing test coverage below project minimums (85% overall, 95% security)
- Operations on production YubiKeys without explicit authorization

**Always Require**:
- Input validation before YubiKey operations
- Sanitized logging using @sanitize_logging decorator
- Error messages that don't expose internal implementation
- Explicit user approval for YubiKey write operations
- Documentation of test device serial numbers

### Builder Agent Security Extensions

When implementing YubiKey functionality:
1. Always use established crypto libraries (never custom implementations)
2. Validate inputs before passing to YubiKey operations
3. Implement proper error handling for hardware disconnection
4. Use @sanitize_logging decorator on sensitive functions
5. Include security docstring section for sensitive operations
6. Write security-focused tests (negative cases, attack scenarios)

### Validator Agent Security Extensions

When testing YubiKey functionality:
1. Include negative tests (invalid PINs, expired certs, malformed data)
2. Test disconnection scenarios and timeouts
3. Verify PIN retry counter behavior
4. Test touch policy enforcement
5. Validate attestation chains
6. Check for timing attacks in PIN validation
7. Ensure no sensitive data in test output

### DevOps Agent Security Extensions

When setting up CI/CD:
1. Use mock YubiKeys in CI pipeline (never real hardware)
2. Run bandit security linting on every commit
3. Run safety dependency checks regularly
4. Implement branch protection on main branch
5. Require security review approval for sensitive changes
6. Set up automated security scanning (Dependabot, etc.)

## Context Handoff Protocol

When transitioning between agents or sessions:

1. **State Summary**: Current task, completed work, remaining tasks
2. **Security Context**: Any security considerations discovered
3. **Test Status**: Coverage metrics, failing tests, hardware requirements
4. **Blockers**: Dependencies, approval requirements, hardware availability
5. **File Changes**: Modified files with brief descriptions
6. **Next Steps**: Prioritized list of next tasks

### Handoff Template

```markdown
## Agent Handoff: [From Agent] → [To Agent]

**Date**: [ISO 8601 timestamp]
**Session ID**: [if applicable]

### Current State
- Task: [current task description]
- Progress: [% complete or milestone]
- Branch: [git branch]
- Last Commit: [commit hash and message]

### Completed Work
- [Completed item 1]
- [Completed item 2]

### Security Considerations
- [Any security concerns discovered]
- [Sensitive operations performed]
- [Approval requirements for next steps]

### Test Status
- Coverage: [X%] (Target: 85% overall, 95% security)
- Failing Tests: [count] ([brief description])
- Hardware Tests: [Passed/Not Run/Blocked]

### Blockers
- [Blocker 1 with mitigation plan]
- [Blocker 2 with mitigation plan]

### Files Modified
- `path/to/file.py`: [brief description]
- `path/to/test.py`: [brief description]

### Next Steps (Prioritized)
1. [Highest priority task]
2. [Second priority task]
3. [Third priority task]

### Notes
[Any additional context or considerations]
```

## YubiKey-Specific Testing Requirements

### Test Device Management

All agents must follow these protocols when working with YubiKey hardware:

1. **Test Device Registry**: Maintain `tests/fixtures/TEST_DEVICES.md` with:
   - Serial numbers of designated test devices
   - Firmware versions
   - Configuration state
   - Last test date
   - Known issues

2. **Production Key Protection**:
   - NEVER test against devices not in TEST_DEVICES.md
   - Prompt for explicit confirmation before ANY write operation
   - Verify serial number matches test registry before proceeding
   - Log all hardware interactions with timestamps

3. **Test Isolation**:
   - Reset test keys to known state before test runs
   - Document pre-test and post-test device state
   - Clean up test credentials after test completion
   - Never leave test keys in production-like configuration

## Agent Performance Metrics

### Quality Metrics by Agent Role

**Security Validator**:
- Zero false negatives on security vulnerabilities
- < 10% false positive rate on security warnings
- 100% coverage of OWASP top 10 in reviews

**YubiKey Hardware Tester**:
- Test all supported YubiKey models (4, 5, 5 FIPS)
- Test all firmware versions in support matrix
- 100% hardware test pass rate before release

**Cryptography Reviewer**:
- Zero usage of deprecated algorithms in production code
- 100% usage of established crypto libraries
- All random generation uses `secrets` module

**Builder**:
- Minimum 85% test coverage overall
- Minimum 95% coverage on security modules
- Zero security warnings from bandit
- Type hints on 100% of public functions

**Validator**:
- All tests pass before marking complete
- Integration tests run on real hardware for releases
- Negative test cases for all security-critical functions

## Continuous Improvement

This AGENTS.md file should be updated when:
- New agent roles are needed for YubiKey development
- Security protocols change or are enhanced
- New YubiKey models require different testing approaches
- Collaboration patterns prove ineffective
- Community feedback suggests improvements

**Last Updated**: 2025-11-20
**Maintained By**: Elvis Nuno
**Review Cycle**: Monthly or upon significant project changes
