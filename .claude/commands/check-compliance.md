---
description: "Verify security standards compliance including test coverage, credential handling, input sanitization, and error handling"
allowed-tools: ["Read", "Search", "Grep", "Bash(pytest:*)", "Bash(coverage:*)"]
author: "YubiKey Tools Compliance Team"
version: "1.0"
---

# Check Compliance

## Purpose
Verify adherence to security standards, coding best practices, and project-specific compliance requirements defined in CLAUDE.md.

## Compliance Checking Steps

### 1. Test Coverage Compliance

```bash
# Run tests with coverage
!pytest --cov=src --cov-report=term-missing --cov-report=json

# Parse coverage results
!coverage report --precision=2

# Check security module coverage specifically
!coverage report --include="src/validators/*,src/utils/logging_utils.py" --precision=2
```

### 2. Analyze Coverage Data

```python
import json
with open('coverage.json') as f:
    data = json.load(f)
    overall_coverage = data['totals']['percent_covered']

    security_modules = [
        'src/validators/',
        'src/utils/logging_utils.py'
    ]

    # Calculate security module coverage
```

### 3. Check for Credentials in Code

```bash
# Search for hardcoded credentials
!grep -rn "password\s*=\s*[\"']" src/ --include="*.py" || echo "✅ No hardcoded passwords"
!grep -rn "api_key\s*=\s*[\"']" src/ --include="*.py" || echo "✅ No hardcoded API keys"
!grep -rn "secret\s*=\s*[\"']" src/ --include="*.py" || echo "✅ No hardcoded secrets"
!grep -rn "token\s*=\s*[\"']" src/ --include="*.py" || echo "✅ No hardcoded tokens"

# Check for PINs in code
!grep -rn "pin\s*=\s*[\"']\\d" src/ --include="*.py" || echo "✅ No hardcoded PINs"
!grep -rn "management_key\s*=\s*[\"']" src/ --include="*.py" || echo "✅ No hardcoded management keys"
```

### 4. Verify Logging Sanitization

```bash
# Check for @sanitize_logging decorator implementation
@Read: src/utils/logging_utils.py

# Find functions handling sensitive data
!grep -rn "def.*pin\|def.*password\|def.*key\|def.*secret" src/ --include="*.py" | head -20

# Check if @sanitize_logging is used
!grep -B 5 "def.*pin\|def.*password" src/ --include="*.py" | grep -c "@sanitize_logging"
```

### 5. Validate Input Sanitization

```bash
# Check for input validation before YubiKey operations
@Read: src/validators/pin_validator.py
@Read: src/validators/certificate_validator.py

# Find YubiKey operation calls
!grep -rn "yubikey\\.\|device\\." src/ --include="*.py" | head -20

# Check for validation before operations
!grep -B 5 "yubikey\\..*(" src/ --include="*.py" | grep "validate\|check\|verify" || echo "⚠️  May be missing validation"
```

### 6. Review Error Handling

```bash
# Find exception handling
!grep -rn "except.*:" src/ --include="*.py" | head -30

# Check for information disclosure in error messages
!grep -rn "except.*:.*print\|except.*:.*log" src/ --include="*.py"

# Verify no stack traces exposed
!grep -rn "traceback\\.print_exc\|traceback\\.format_exc" src/ --include="*.py" || echo "✅ No traceback exposure"
```

### 7. Check Type Hints Coverage

```bash
# Find functions without type hints
!grep -rn "^def " src/ --include="*.py" | grep -v " -> " | head -20

# Run mypy for type checking
!mypy src/ --strict --show-error-codes 2>&1 | head -50
```

### 8. Verify Documentation Coverage

```bash
# Check for docstrings on public functions
!grep -rn "^def [^_]" src/ --include="*.py" -A 3 | grep -c '"""' || echo "0"
!grep -rn "^class [^_]" src/ --include="*.py" -A 3 | grep -c '"""' || echo "0"

# Count total public functions/classes
!grep -rn "^def [^_]\|^class [^_]" src/ --include="*.py" | wc -l
```

### 9. Generate Compliance Report

Create **COMPLIANCE_REPORT.md**:

```markdown
# Security Compliance Report

**Audit Date**: [ISO 8601 timestamp]
**Repository**: yubikey-tools
**Commit**: [git commit hash]
**Compliance Framework**: CLAUDE.md + OWASP + YubiKey Security Standards

---

## Executive Summary

| Requirement | Status | Score | Target | Result |
|-------------|--------|-------|--------|--------|
| Test Coverage (Overall) | [PASS/FAIL] | [XX%] | ≥85% | [✅/❌] |
| Test Coverage (Security) | [PASS/FAIL] | [XX%] | ≥95% | [✅/❌] |
| No Hardcoded Credentials | [PASS/FAIL] | [N found] | 0 | [✅/❌] |
| Logging Sanitization | [PASS/FAIL] | [XX%] | 100% | [✅/❌] |
| Input Validation | [PASS/FAIL] | [XX%] | 100% | [✅/❌] |
| Error Handling | [PASS/FAIL] | [N issues] | 0 | [✅/❌] |
| Type Hints | [PASS/FAIL] | [XX%] | 100% | [✅/❌] |
| Documentation | [PASS/FAIL] | [XX%] | 100% | [✅/❌] |

**Overall Compliance**: ✅ COMPLIANT / ⚠️  PARTIAL / ❌ NON-COMPLIANT

---

## Test Coverage Analysis

### Overall Coverage
```
Module                          Statements   Missing   Coverage
─────────────────────────────────────────────────────────────────
src/core/operations/            450          23        94.9%
src/core/config/                120          8         93.3%
src/validators/                 200          5         97.5%
src/utils/                      150          12        92.0%
src/tools/                      300          45        85.0%
─────────────────────────────────────────────────────────────────
TOTAL                          1220          93        92.4%
```

**Status**: ✅ PASS (≥85% target met)

### Security Module Coverage
```
Module                               Statements   Missing   Coverage
───────────────────────────────────────────────────────────────────
src/validators/pin_validator.py     45           1         97.8%
src/validators/certificate_validator.py  60      2         96.7%
src/validators/attestation_validator.py  55      1         98.2%
src/utils/logging_utils.py           40           0         100.0%
───────────────────────────────────────────────────────────────────
TOTAL                                200          4         98.0%
```

**Status**: ✅ PASS (≥95% target met)

### Uncovered Critical Paths
[List any security-critical code without test coverage]

---

## Credential Security

### Hardcoded Credentials Scan
- **Passwords**: ✅ None found
- **API Keys**: ✅ None found
- **Secrets**: ✅ None found
- **Tokens**: ✅ None found
- **PINs**: ✅ None found
- **Management Keys**: ✅ None found

**Status**: ✅ PASS

### Environment Variable Usage
```bash
# Found in: src/core/config/config_loader.py
TEST_PIN = os.getenv('TEST_YUBIKEY_PIN')  # ✅ CORRECT
MGMT_KEY = os.getenv('TEST_MGMT_KEY')     # ✅ CORRECT
```

**Status**: ✅ PASS - Using environment variables appropriately

### Git History Scan
```bash
# Check for credentials in git history
!git log --all --full-history -- "*.py" | grep -i "password\|secret\|key" || echo "✅ Clean history"
```

**Status**: [✅ PASS / ⚠️  REVIEW NEEDED]

---

## Logging Sanitization

### @sanitize_logging Decorator Implementation
```python
# Found in: src/utils/logging_utils.py
@Read: src/utils/logging_utils.py

# Implementation:
✅ Decorator defined
✅ Redacts PINs (pattern: \d{6,8})
✅ Redacts keys (pattern: [A-F0-9]{32,})
✅ Redacts secrets (pattern: secret.*=.*)
✅ Replaces with [REDACTED]
```

**Status**: ✅ IMPLEMENTED

### Usage Coverage
```
Functions handling sensitive data: [N]
Functions with @sanitize_logging:  [N]
Coverage:                           [XX%]
```

**Required Actions**:
- [ ] Add @sanitize_logging to `src/core/operations/piv_operations.py:authenticate()`
- [ ] Add @sanitize_logging to `src/tools/yubikey-config:set_pin()`

**Status**: [✅ PASS / ⚠️  INCOMPLETE]

---

## Input Validation

### PIN Validation
```python
# Found in: src/validators/pin_validator.py
✅ Format validation (6-8 digits)
✅ Complexity checks
✅ Length validation
✅ Non-numeric rejection
```

### Certificate Validation
```python
# Found in: src/validators/certificate_validator.py
✅ Expiration checking
✅ Chain of trust verification
✅ Key usage validation
✅ Signature verification
```

### Input Validation Before YubiKey Operations
```
Total YubiKey operations:     [N]
Operations with validation:   [N]
Validation coverage:          [XX%]
```

**Status**: [✅ PASS / ⚠️  NEEDS IMPROVEMENT]

---

## Error Handling

### Exception Handling Patterns
```python
# Good example (found in: [location])
try:
    yubikey.authenticate(pin)
except YubiKeyAuthenticationError:
    logger.error("Authentication failed")  # ✅ No internal details
    raise

# Bad example (if found):
except Exception as e:
    print(f"Error: {e}")  # ❌ May expose internals
```

### Information Disclosure Check
- **Stack traces in logs**: [FOUND / NOT FOUND]
- **Internal errors exposed**: [FOUND / NOT FOUND]
- **Debug info in production**: [FOUND / NOT FOUND]

**Status**: [✅ PASS / ⚠️  REVIEW NEEDED]

### Custom Exception Classes
```python
# Found in: src/utils/error_handling.py
✅ YubiKeyConnectionError
✅ YubiKeyAuthenticationError
✅ YubiKeyOperationError
✅ YubiKeyNotFoundError
```

**Status**: ✅ PASS - Proper exception hierarchy

---

## Type Hints Coverage

### Type Hint Statistics
```
Total functions:                 [N]
Functions with type hints:       [N]
Functions with return types:     [N]
Type hint coverage:              [XX%]
```

### mypy Compliance
```bash
!mypy src/ --strict

Found [N] errors in [N] files
```

**Status**: [✅ PASS / ⚠️  NEEDS IMPROVEMENT]

### Missing Type Hints
[List functions missing type hints]

---

## Documentation Coverage

### Docstring Statistics
```
Public functions:                [N]
Functions with docstrings:       [N]
Public classes:                  [N]
Classes with docstrings:         [N]
Documentation coverage:          [XX%]
```

### Docstring Quality
- [ ] ✅ Google-style docstrings used
- [ ] ✅ Args documented
- [ ] ✅ Returns documented
- [ ] ✅ Raises documented
- [ ] ✅ Examples provided
- [ ] ✅ Security sections for sensitive functions

**Status**: [✅ PASS / ⚠️  NEEDS IMPROVEMENT]

---

## YubiKey-Specific Compliance

### PIN/PUK Handling
- [ ] ✅ No PINs in logs
- [ ] ✅ No PINs in print statements
- [ ] ✅ PIN validation before use
- [ ] ✅ @sanitize_logging on PIN functions
- [ ] ✅ Secure PIN input methods

**Status**: [✅ PASS / ⚠️  ISSUES FOUND]

### YubiKey Operations Safety
- [ ] ✅ Test device verification
- [ ] ✅ User approval for write operations
- [ ] ✅ Production device protection
- [ ] ✅ Device disconnection handling
- [ ] ✅ Touch policy enforcement

**Status**: [✅ PASS / ⚠️  ISSUES FOUND]

### Cryptographic Operations
- [ ] ✅ No custom crypto implementations
- [ ] ✅ Established libraries only
- [ ] ✅ Secure random generation (secrets)
- [ ] ✅ No deprecated algorithms
- [ ] ✅ Certificate chain validation

**Status**: [✅ PASS / ⚠️  ISSUES FOUND]

---

## OWASP Top 10 Compliance

| Control | Status | Notes |
|---------|--------|-------|
| A01: Broken Access Control | ✅ | Test device registry enforced |
| A02: Cryptographic Failures | ✅ | Using established libraries |
| A03: Injection | ✅ | Input validation implemented |
| A04: Insecure Design | ✅ | Security-first architecture |
| A05: Security Misconfiguration | ✅ | Secure defaults |
| A06: Vulnerable Components | ✅ | Dependencies scanned |
| A07: Authentication Failures | ✅ | PIN validation enforced |
| A08: Software Integrity Failures | ✅ | Attestation verification |
| A09: Logging Failures | ✅ | Sanitized logging |
| A10: Server-Side Request Forgery | N/A | No network operations |

**OWASP Compliance**: ✅ 9/9 applicable controls passed

---

## Project-Specific Requirements (from CLAUDE.md)

### Security Standards
- [ ] ✅ Check deprecated packages (via safety)
- [ ] ✅ Verify no hardcoded credentials
- [ ] ✅ Follow principle of least privilege
- [ ] ✅ Audit for common vulnerabilities
- [ ] ✅ Implement proper error handling

### Code Quality Standards
- [ ] ✅ Python: Type hints on all functions
- [ ] ✅ Python: PEP 8 compliance
- [ ] ✅ Testing: 85%+ overall coverage
- [ ] ✅ Testing: 95%+ security module coverage
- [ ] ✅ Documentation: Docstrings on public APIs

### YubiKey Security Requirements
- [ ] ✅ Never log PINs, PUKs, or keys
- [ ] ✅ Test against mainnet forks (if applicable)
- [ ] ✅ Document transaction flows
- [ ] ✅ Implement proper gas estimation (N/A)

**Project Compliance**: [XX/YY] requirements met ([XX%])

---

## Recommendations

### Critical (Fix Immediately)
1. **[Issue]**: [Description]
   - Location: [file:line]
   - Risk: CRITICAL
   - Fix: [Remediation]

### High Priority (This Week)
1. **Improve security module coverage**: Target 95%+
2. **Add missing @sanitize_logging decorators**: [List locations]
3. **Complete type hint coverage**: [List functions]

### Medium Priority (This Sprint)
1. **Enhance docstring coverage**: Target 100% public APIs
2. **Add input validation**: [List operations]
3. **Review error messages**: Ensure no information disclosure

### Low Priority (Future)
1. **Performance optimization**: [Suggestions]
2. **Additional test cases**: [Suggestions]

---

## Compliance Trends

### Historical Compliance
| Date | Coverage | Credentials | Logging | Overall |
|------|----------|-------------|---------|---------|
| 2025-11-20 | 92% | ✅ | ✅ | ✅ |
| 2025-11-13 | 88% | ✅ | ⚠️  | ⚠️  |
| 2025-11-06 | 85% | ✅ | ⚠️  | ⚠️  |

**Trend**: 📈 IMPROVING

---

## Action Items

### Immediate
- [ ] Fix [N] critical compliance issues
- [ ] Add missing @sanitize_logging decorators
- [ ] Improve security module test coverage to 95%+

### Short-Term (This Week)
- [ ] Complete type hint coverage
- [ ] Enhance docstring coverage
- [ ] Address input validation gaps

### Long-Term (This Month)
- [ ] Establish compliance monitoring automation
- [ ] Create compliance dashboard
- [ ] Schedule quarterly compliance reviews

---

**Report Generated**: [Timestamp]
**Next Compliance Check**: [Recommended date]
**Compliance Officer**: [Name/Team]
**Approved By**: [Reviewer]
```

### 10. Display Compliance Summary

```
╔════════════════════════════════════════════════════╗
║           COMPLIANCE CHECK COMPLETE                ║
╚════════════════════════════════════════════════════╝

OVERALL COMPLIANCE: ✅ COMPLIANT / ⚠️  PARTIAL / ❌ NON-COMPLIANT

COMPLIANCE METRICS:
  Test Coverage (Overall):     [92%] ✅ (≥85%)
  Test Coverage (Security):    [98%] ✅ (≥95%)
  Hardcoded Credentials:       [0]   ✅
  Logging Sanitization:        [95%] ⚠️  (100% target)
  Input Validation:            [100%] ✅
  Error Handling:              [100%] ✅
  Type Hints:                  [98%] ⚠️  (100% target)
  Documentation:               [92%] ⚠️  (100% target)

OWASP TOP 10:
  Compliant Controls:  9/9 (100%) ✅

PROJECT-SPECIFIC (CLAUDE.md):
  Requirements Met:    15/17 (88%) ⚠️

ISSUES REQUIRING ATTENTION:
  🔴 Critical:  [N]
  🟠 High:      [N]
  🟡 Medium:    [N]
  🟢 Low:       [N]

TOP PRIORITIES:
  » Add @sanitize_logging to [N] functions
  » Complete type hints for [N] functions
  » Improve documentation for [N] public APIs

Full Report: COMPLIANCE_REPORT.md

NEXT STEPS:
  » Address high-priority issues
  » Schedule follow-up compliance check
  » Update compliance tracking
```

## Key Features

- **Comprehensive Compliance**: Tests all security and quality standards
- **Coverage Analysis**: Detailed test coverage with security focus
- **Credential Scanning**: Detects hardcoded secrets
- **Logging Validation**: Verifies sanitization implementation
- **Input Validation**: Checks for proper input checking
- **Error Handling**: Ensures no information disclosure
- **OWASP Compliance**: Maps to OWASP Top 10
- **Project-Specific**: Validates CLAUDE.md requirements
- **Trend Tracking**: Historical compliance data

## When to Use /check-compliance

- Before pull requests
- Weekly during active development
- Before releases
- After security fixes
- During code reviews
- Monthly security reviews
- After adding new features
- For compliance audits

## Best Practices

1. **Regular Checks**: Run weekly or before PRs
2. **Fix Critical First**: Address compliance failures immediately
3. **Track Trends**: Monitor compliance over time
4. **Automate**: Integrate into CI/CD pipeline
5. **Document**: Keep compliance reports for audits
6. **Review**: Have Security Validator agent review findings
7. **Continuous Improvement**: Aim for 100% compliance
8. **Stay Current**: Update standards as best practices evolve
