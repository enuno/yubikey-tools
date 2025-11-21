---
description: "Run comprehensive security scan for YubiKey tools including vulnerability checks, credential exposure detection, and logging sanitization validation"
allowed-tools: ["Read", "Search", "Grep", "Bash(bandit:*)", "Bash(safety:*)", "Bash(grep:*)", "Bash(find:*)"]
author: "YubiKey Tools Security Team"
version: "1.0"
---

# Security Audit

## Purpose
Perform comprehensive security scanning of YubiKey tools codebase to identify vulnerabilities, hardcoded credentials, sensitive data exposure, and security best practice violations.

## Security Audit Steps

### 1. Python Security Linting (bandit)

```bash
# Run bandit on all Python code with medium/high severity
!bandit -r src/ -ll --format screen

# Generate detailed JSON report
!bandit -r src/ -ll --format json -o security-audit-bandit.json
```

### 2. Dependency Vulnerability Scanning (safety)

```bash
# Check for known security vulnerabilities in dependencies
!safety check --json

# Full vulnerability report
!safety check --full-report
```

### 3. Search for Hardcoded Credentials

```bash
# Search for common credential patterns
@Search:
  patterns:
    - "password\\s*=\\s*[\"'][^\"']+[\"']"
    - "api_key\\s*=\\s*[\"'][^\"']+[\"']"
    - "secret\\s*=\\s*[\"'][^\"']+[\"']"
    - "token\\s*=\\s*[\"'][^\"']+[\"']"
    - "pin\\s*=\\s*[\"']\\d+[\"']"
    - "management_key\\s*=\\s*[\"'][^\"']+[\"']"

# Search for potential credential variables (case-insensitive)
!grep -ri "password\s*=" src/ --include="*.py" || echo "No password assignments found"
!grep -ri "api_key\s*=" src/ --include="*.py" || echo "No API key assignments found"
!grep -ri "secret_key\s*=" src/ --include="*.py" || echo "No secret assignments found"
```

### 4. Check for Sensitive Data in Logs

```bash
# Search for PIN logging
!grep -rn "print.*pin" src/ --include="*.py" || echo "No print(pin) found"
!grep -rn "log.*pin[^_]" src/ --include="*.py" --ignore-case || echo "No log(pin) found"
!grep -rn "logger.*pin[^_]" src/ --include="*.py" --ignore-case || echo "No logger(pin) found"

# Search for key material logging
!grep -rn "print.*key" src/ --include="*.py" | grep -v "yubikey" | grep -v "pubkey" || echo "No suspicious key logging"
!grep -rn "log.*private" src/ --include="*.py" --ignore-case || echo "No private key logging"

# Search for credential logging
!grep -rn "print.*password" src/ --include="*.py" || echo "No password printing"
!grep -rn "log.*management_key" src/ --include="*.py" --ignore-case || echo "No management_key logging"
```

### 5. Validate @sanitize_logging Decorator Usage

```bash
# Find all functions handling PINs/keys without decorator
!find src/ -name "*.py" -type f -exec grep -l "def.*pin" {} \; | while read file; do
  echo "Checking $file for @sanitize_logging decorator"
  grep -B 5 "def.*pin" "$file" | grep -q "sanitize_logging" || echo "WARNING: $file may be missing @sanitize_logging"
done

# Check for sanitize_logging implementation
@Read: src/utils/logging_utils.py
```

### 6. Check for Insecure Cryptographic Usage

```bash
# Search for deprecated algorithms
!grep -rn "MD5\|md5" src/ --include="*.py" || echo "No MD5 usage found"
!grep -rn "SHA1\|sha1" src/ --include="*.py" | grep -v "SHA256" || echo "No SHA1 usage found"

# Search for custom crypto implementations
!grep -rn "def encrypt\|def decrypt\|def hash" src/ --include="*.py" || echo "No custom crypto found"

# Verify use of secrets module for randomness
!grep -rn "random\\.rand" src/ --include="*.py" || echo "No insecure random usage"
!grep -rn "import secrets" src/ --include="*.py" || echo "WARNING: secrets module not used"
```

### 7. Check Input Validation

```bash
# Search for YubiKey operations without validation
!grep -rn "yubikey\\..*(" src/ --include="*.py" | head -20

# Check for PIN validation before use
@Read: src/validators/pin_validator.py
```

### 8. Review Error Handling for Information Disclosure

```bash
# Search for error messages that might expose internals
!grep -rn "except.*:.*print\|except.*:.*log" src/ --include="*.py" | head -20

# Check for stack trace exposure
!grep -rn "traceback\\.print_exc\\|traceback\\.format_exc" src/ --include="*.py" || echo "No traceback exposure"
```

### 9. Generate Security Report

Create **SECURITY_AUDIT_REPORT.md**:

```markdown
# Security Audit Report - YubiKey Tools

**Audit Date**: [ISO 8601 timestamp]
**Auditor**: Claude Security Validator
**Repository**: yubikey-tools
**Commit**: [git commit hash]

---

## Executive Summary

| Category | Status | Issues Found | Severity |
|----------|--------|--------------|----------|
| Python Security (bandit) | [PASS/FAIL] | [N] | [CRITICAL/HIGH/MEDIUM/LOW] |
| Dependencies (safety) | [PASS/FAIL] | [N] | [CRITICAL/HIGH/MEDIUM/LOW] |
| Hardcoded Credentials | [PASS/FAIL] | [N] | [CRITICAL] |
| Sensitive Data Logging | [PASS/FAIL] | [N] | [HIGH] |
| Cryptographic Issues | [PASS/FAIL] | [N] | [HIGH] |
| Input Validation | [PASS/FAIL] | [N] | [MEDIUM] |
| Error Handling | [PASS/FAIL] | [N] | [MEDIUM] |

**Overall Security Posture**: 🟢 SECURE / 🟡 NEEDS ATTENTION / 🔴 CRITICAL ISSUES

---

## Critical Issues (Immediate Action Required)

### Issue #1: [Title]
- **Severity**: CRITICAL
- **Category**: [Credentials/Logging/Crypto]
- **Location**: `path/to/file.py:line`
- **Description**: [What was found]
- **Risk**: [What could happen]
- **Remediation**: [How to fix]

**Fix Command**:
\`\`\`python
# Recommended fix
[Code snippet]
\`\`\`

---

## High Severity Issues

### Issue #1: [Title]
- **Severity**: HIGH
- **Location**: `path/to/file.py:line`
- **Description**: [Issue description]
- **Recommendation**: [Fix recommendation]

---

## Medium Severity Issues

[List of medium severity findings]

---

## Security Best Practices Status

### Implemented ✅
- [ ] @sanitize_logging decorator on sensitive functions
- [ ] Input validation on YubiKey operations
- [ ] Secure random number generation (secrets module)
- [ ] Certificate chain validation
- [ ] No hardcoded credentials
- [ ] Proper error handling without information disclosure

### Missing ⚠️
- [ ] [Security practice not yet implemented]
- [ ] [Security practice not yet implemented]

---

## Bandit Security Scan Results

**Scan Summary**:
- Total Issues: [N]
- High Severity: [N]
- Medium Severity: [N]
- Low Severity: [N]

**High Severity Findings**:
[List from bandit output]

**Full Report**: `security-audit-bandit.json`

---

## Dependency Vulnerability Scan Results

**Safety Check Summary**:
- Vulnerable Packages: [N]
- CVEs Found: [N]

**Vulnerable Dependencies**:
| Package | Current Version | CVE | Severity | Fix Version |
|---------|----------------|-----|----------|-------------|
| [name] | [version] | [CVE-ID] | [HIGH/CRITICAL] | [version] |

**Update Commands**:
\`\`\`bash
pip install [package]==[version]
\`\`\`

---

## Credential and Secret Scanning

### Hardcoded Credentials
[PASS/FAIL]: [Description of findings]

### Environment Variables
[Check for .env files in repository]
[Verify no .env in git history]

### Sensitive Data in Logs
[Results of log scanning]

---

## Cryptographic Security

### Algorithm Usage
- ✅ Using cryptography library
- ✅ No deprecated algorithms (MD5, SHA1 for security)
- ✅ Secure random generation (secrets module)
- [ ] Certificate validation implemented

### Key Management
- [ ] No private keys in repository
- [ ] Proper key storage patterns
- [ ] Secure key generation parameters

---

## YubiKey-Specific Security Checks

### PIN/PUK Handling
- [ ] No PINs in logs or print statements
- [ ] PIN validation before YubiKey operations
- [ ] Secure PIN input methods
- [ ] @sanitize_logging on PIN functions

### YubiKey Operations
- [ ] Attestation validation for FIDO2
- [ ] Certificate chain validation for PIV
- [ ] Touch policy enforcement
- [ ] Proper disconnection handling

### Test Security
- [ ] Test devices documented in fixtures
- [ ] No production keys in tests
- [ ] Test data sanitization

---

## Recommendations

### Immediate (Fix Today)
1. **[Critical Issue]**: [Description and fix]
   - Time: [estimate]
   - Risk: CRITICAL

### Short-Term (This Week)
1. **[High Issue]**: [Description and fix]
   - Time: [estimate]
   - Risk: HIGH

### Long-Term (This Month)
1. **[Medium Issue]**: [Description and fix]
   - Time: [estimate]
   - Risk: MEDIUM

---

## Compliance Status

### OWASP Top 10
- [x] A01: Broken Access Control - [status]
- [x] A02: Cryptographic Failures - [status]
- [x] A03: Injection - [status]
- [x] A04: Insecure Design - [status]
- [x] A05: Security Misconfiguration - [status]
- [x] A06: Vulnerable Components - [status]
- [x] A07: Authentication Failures - [status]
- [x] A08: Software Integrity Failures - [status]
- [x] A09: Logging Failures - [status]
- [x] A10: Server-Side Request Forgery - [status]

---

## Next Steps

1. **Address Critical Issues**: [List]
2. **Review High Severity**: [List]
3. **Schedule Fix Sprint**: [Date]
4. **Re-run Audit**: [Date]
5. **Update Security Documentation**: [Needed updates]

---

**Report Generated**: [Timestamp]
**Next Audit**: [Recommended date]
**Sign-off**: [Auditor]
```

### 10. Display Audit Summary

```
╔════════════════════════════════════════════════════╗
║           SECURITY AUDIT COMPLETED                 ║
╚════════════════════════════════════════════════════╝

PROJECT: YubiKey Tools
AUDIT DATE: [Date and time]

SECURITY STATUS:
  🔴 Critical Issues: [N] - FIX IMMEDIATELY
  🟠 High Issues: [N] - Fix this week
  🟡 Medium Issues: [N] - Schedule for sprint
  🟢 Low Issues: [N] - Monitor

SCAN RESULTS:
  Bandit (Python Security): [PASS/FAIL]
    └─ High severity findings: [N]

  Safety (Dependencies): [PASS/FAIL]
    └─ Vulnerable packages: [N]

  Credential Scan: [PASS/FAIL]
    └─ Hardcoded secrets: [N]

  Sensitive Logging: [PASS/FAIL]
    └─ PIN/key exposure: [N]

YUBIKEY-SPECIFIC:
  ✅ @sanitize_logging implemented
  ✅ PIN validation in place
  ✅ No credential logging
  ⚠️  [Any warnings]

TOP PRIORITIES:
  1. 🔴 [Critical issue to fix]
  2. 🔴 [Critical issue to fix]
  3. 🟠 [High priority issue]

OVERALL SECURITY: 🟢 SECURE / 🟡 NEEDS ATTENTION / 🔴 AT RISK

Full Report: SECURITY_AUDIT_REPORT.md
Bandit JSON: security-audit-bandit.json

RECOMMENDED ACTIONS:
  » Fix critical issues immediately
  » Update vulnerable dependencies
  » Review and address high severity findings
  » Schedule re-audit after fixes
```

## Key Features

- **Comprehensive Scanning**: Python security, dependencies, credentials, logging
- **YubiKey-Specific**: Checks for PIN/PUK handling, key material exposure
- **Automated Detection**: Uses bandit, safety, and pattern matching
- **Detailed Reporting**: Generates SECURITY_AUDIT_REPORT.md
- **Actionable Output**: Prioritized recommendations with fix instructions
- **Compliance Checking**: Maps findings to OWASP Top 10
- **Continuous Security**: Suitable for CI/CD integration

## Severity Classification

| Level | Description | Action | Timeframe |
|-------|-------------|--------|-----------|
| 🔴 Critical | Credential exposure, RCE, auth bypass | Fix immediately | Hours |
| 🟠 High | Sensitive data logging, weak crypto | Fix ASAP | Days |
| 🟡 Medium | Missing validation, information disclosure | Schedule fix | Weeks |
| 🟢 Low | Code quality, minor improvements | Next sprint | Flexible |

## When to Use /security-audit

- Before every commit (via pre-commit hook)
- Before pull requests
- Weekly security monitoring
- After adding new YubiKey operations
- Before releases
- When handling PINs, keys, or credentials
- After dependency updates
- During security reviews

## Best Practices

1. **Regular Audits**: Run before every PR and weekly
2. **Fix Critical First**: Address critical/high before medium/low
3. **Document Fixes**: Update security docs with mitigations
4. **Re-audit**: Run again after fixing issues
5. **Automate**: Integrate into CI/CD pipeline
6. **Track Trends**: Monitor security posture over time
7. **Security Review**: Have Crypto Reviewer agent validate crypto changes
8. **Test Coverage**: Ensure security tests for all critical paths
