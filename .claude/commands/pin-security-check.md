---
description: "Audit PIN handling in codebase to ensure no PIN logging, proper validation, and secure input methods"
allowed-tools: ["Read", "Search", "Grep"]
author: "YubiKey Tools Security Team"
version: "1.0"
---

# PIN Security Check

## Purpose
Perform comprehensive audit of PIN/PUK handling in the codebase to ensure PINs are never logged, properly validated, and handled securely throughout the application.

## PIN Security Audit Steps

### 1. Search for PIN Variables and Functions

```bash
# Find all PIN-related variables
!grep -rn "\\bpin\\b\|\\bpuk\\b\|\\bpassword\\b" src/ --include="*.py" --ignore-case | head -50

# Find PIN-related function definitions
!grep -rn "def.*pin\|def.*puk\|def.*password" src/ --include="*.py" --ignore-case

# Find PIN-related class attributes
!grep -rn "self\\.pin\|self\\.puk\|self\\.password" src/ --include="*.py" --ignore-case
```

### 2. Check for PIN Logging

**CRITICAL**: PINs must NEVER appear in logs

```bash
# Search for PIN in log statements
!grep -rn "log.*pin[^_]" src/ --include="*.py" --ignore-case || echo "✅ No PIN logging found"
!grep -rn "logger.*pin[^_]" src/ --include="*.py" --ignore-case || echo "✅ No logger PIN found"
!grep -rn "logging.*pin[^_]" src/ --include="*.py" --ignore-case || echo "✅ No logging PIN found"

# Search for PIN in print statements
!grep -rn "print.*pin[^_]" src/ --include="*.py" --ignore-case || echo "✅ No PIN printing found"
!grep -rn "print.*puk" src/ --include="*.py" --ignore-case || echo "✅ No PUK printing found"

# Search for PIN in f-strings or format
!grep -rn "f[\"'].*{pin" src/ --include="*.py" --ignore-case || echo "✅ No PIN in f-strings"
!grep -rn "format.*pin" src/ --include="*.py" --ignore-case || echo "✅ No PIN in format strings"
```

### 3. Verify @sanitize_logging Decorator Usage

```bash
# Find functions handling PINs
!grep -rn "def.*pin" src/ --include="*.py" --ignore-case | cut -d: -f1-2

# Check if these functions have @sanitize_logging decorator
!for func in $(grep -rn "def.*pin" src/ --include="*.py" --ignore-case | cut -d: -f1-2); do
  echo "Checking $func"
  grep -B 5 "def.*pin" "$func" | grep -q "@sanitize_logging" && echo "✅ Has decorator" || echo "⚠️  Missing decorator"
done
```

### 4. Check PIN Validation

```bash
# Find PIN validation functions
@Read: src/validators/pin_validator.py

# Check for validation before PIN use
!grep -rn "yubikey.*authenticate\|piv.*authenticate\|device.*verify" src/ --include="*.py" -A 5 -B 5
```

### 5. Review PIN Input Methods

```bash
# Find PIN input/collection
!grep -rn "input.*pin\|getpass\|input()" src/ --include="*.py" --ignore-case

# Check for secure input (getpass module)
!grep -rn "import getpass\|from getpass" src/ --include="*.py"
!grep -rn "getpass\\.getpass" src/ --include="*.py"
```

### 6. Check for Hardcoded PINs

**CRITICAL**: No hardcoded PINs allowed

```bash
# Search for PIN assignments with values
!grep -rn "pin\\s*=\\s*[\"']\\d" src/ --include="*.py" || echo "✅ No hardcoded PINs"
!grep -rn "puk\\s*=\\s*[\"']\\d" src/ --include="*.py" || echo "✅ No hardcoded PUKs"
!grep -rn "default_pin\\s*=\\s*[\"']\\d" src/ --include="*.py" || echo "✅ No default PINs"

# Check for numeric string literals that might be PINs
!grep -rn "[\"']\\d{6,8}[\"']" src/ --include="*.py" | grep -v "# " | head -20
```

### 7. Review PIN Storage

```bash
# Check for PIN persistence (should NOT exist)
!grep -rn "pin.*save\|save.*pin\|store.*pin\|cache.*pin" src/ --include="*.py" --ignore-case || echo "✅ No PIN storage found"

# Check for PIN in configuration files
!grep -rn "pin" src/core/config/ --include="*.py" --include="*.yaml" --include="*.json" --ignore-case
```

### 8. Audit PIN Comparison

```bash
# Find PIN comparison logic
!grep -rn "pin\\s*==\|if.*pin" src/ --include="*.py" --ignore-case | head -20

# Check for constant-time comparison (should use secrets.compare_digest)
!grep -rn "secrets\\.compare_digest" src/ --include="*.py"
!grep -rn "hmac\\.compare_digest" src/ --include="*.py"
```

### 9. Review Error Messages

```bash
# Check for PIN exposure in error messages
!grep -rn "raise.*pin\|except.*pin" src/ --include="*.py" --ignore-case -A 2 -B 2
!grep -rn "ValueError.*pin\|Exception.*pin" src/ --include="*.py" --ignore-case
```

### 10. Generate PIN Security Report

Create **PIN_SECURITY_REPORT.md**:

```markdown
# PIN Security Audit Report

**Audit Date**: [ISO 8601 timestamp]
**Auditor**: PIN Security Checker
**Repository**: yubikey-tools
**Commit**: [git commit hash]

---

## Executive Summary

| Security Control | Status | Issues | Severity |
|------------------|--------|--------|----------|
| No PIN Logging | [PASS/FAIL] | [N] | CRITICAL |
| @sanitize_logging Usage | [PASS/FAIL] | [N] | HIGH |
| PIN Validation | [PASS/FAIL] | [N] | HIGH |
| Secure PIN Input | [PASS/FAIL] | [N] | HIGH |
| No Hardcoded PINs | [PASS/FAIL] | [N] | CRITICAL |
| No PIN Storage | [PASS/FAIL] | [N] | CRITICAL |
| Constant-Time Comparison | [PASS/FAIL] | [N] | MEDIUM |
| Error Message Safety | [PASS/FAIL] | [N] | MEDIUM |

**Overall PIN Security**: 🟢 SECURE / 🟡 NEEDS IMPROVEMENT / 🔴 INSECURE

---

## Critical Issues

### Issue #1: PIN Logging Detected
**Status**: [FOUND / NOT FOUND]

If found:
- **Location**: `src/core/operations/piv_operations.py:142`
- **Code**: `logger.info(f"Authenticating with PIN: {pin}")`
- **Risk**: CRITICAL - PIN exposed in logs
- **Remediation**: Remove PIN from log statement

```python
# INSECURE
logger.info(f"Authenticating with PIN: {pin}")

# SECURE
logger.info("Authenticating with PIN")  # No PIN value
```

### Issue #2: Hardcoded PIN
**Status**: [FOUND / NOT FOUND]

If found:
- **Location**: `tests/unit/test_piv.py:56`
- **Code**: `pin = "123456"`
- **Risk**: CRITICAL - Hardcoded credential
- **Remediation**: Use environment variable or fixture

```python
# INSECURE
pin = "123456"  # Hardcoded

# SECURE
pin = os.getenv('TEST_PIN', None)  # From environment
if not pin:
    pytest.skip("TEST_PIN not set")
```

---

## PIN Logging Analysis

### Log Statements Audit
```bash
Total log/logger statements: [N]
Statements mentioning "pin": [N]
Statements logging PIN value: [N] 🔴
```

### Findings

#### Secure Logging ✅
```python
# Found in: src/core/operations/yubikey_detection.py:87
logger.info("PIN validation successful")  # ✅ No PIN value
logger.debug("Authenticating with YubiKey")  # ✅ No PIN value
```

#### Insecure Logging ❌
[List any findings or "None found"]

### Print Statements Audit
```bash
Total print statements: [N]
Statements printing PIN: [N]
```

**Status**: [✅ PASS / ❌ FAIL]

---

## @sanitize_logging Decorator Coverage

### Decorator Implementation
```python
# From: src/utils/logging_utils.py
@Read: src/utils/logging_utils.py
```

**Implementation Status**: [✅ IMPLEMENTED / ❌ MISSING]

### Coverage Analysis
```
Functions handling PINs: [N]
Functions with @sanitize_logging: [N]
Coverage: [XX%]
```

### Missing Decorator
[List functions that need @sanitize_logging]

1. `src/core/operations/piv_operations.py:authenticate()`
   - Handles PIN parameter
   - Returns sensitive data
   - **Action**: Add @sanitize_logging decorator

2. `src/tools/yubikey-config.py:set_pin()`
   - Handles old and new PIN
   - **Action**: Add @sanitize_logging decorator

**Recommendation**: Add @sanitize_logging to ALL functions with PIN parameters

---

## PIN Validation

### Validation Implementation
```python
# From: src/validators/pin_validator.py
@Read: src/validators/pin_validator.py
```

**Functions Found**:
- ✅ `validate_pin_format(pin: str) -> bool`
- ✅ `validate_pin_complexity(pin: str) -> bool`
- ✅ `validate_puk_format(puk: str) -> bool`

### Validation Checks Implemented
- [✅ / ❌] Length validation (6-8 digits)
- [✅ / ❌] Numeric-only validation
- [✅ / ❌] Complexity checks (no "000000", "123456")
- [✅ / ❌] Type checking (string input)

### Validation Before Use
```
YubiKey operations found: [N]
Operations with prior validation: [N]
Validation coverage: [XX%]
```

**Status**: [✅ GOOD / ⚠️  NEEDS IMPROVEMENT]

### Examples

#### Good Practice ✅
```python
# From: src/core/operations/piv_operations.py:120
if not validate_pin_format(pin):
    raise ValueError("Invalid PIN format")
yubikey.authenticate(pin)
```

#### Missing Validation ⚠️
[List any operations without validation]

---

## PIN Input Methods

### Input Method Analysis
```bash
# Input methods found
import getpass: [FOUND / NOT FOUND]
input() usage: [N] instances
getpass.getpass() usage: [N] instances
```

### Secure Input ✅
```python
# From: src/tools/yubikey-config.py:45
import getpass
pin = getpass.getpass("Enter PIN: ")  # ✅ SECURE - hidden input
```

### Insecure Input ❌
```python
# If found
pin = input("Enter PIN: ")  # ❌ INSECURE - visible input
```

**Recommendation**: Always use `getpass.getpass()` for PIN input

---

## Hardcoded PINs

### Hardcoded Credential Scan
```bash
# Search results
PIN assignments with values: [N]
PUK assignments with values: [N]
Numeric string literals: [N]
```

### Findings

#### Production Code
**Status**: [✅ CLEAN / ❌ HARDCODED PINs FOUND]

[List any findings]

#### Test Code
**Status**: [✅ ACCEPTABLE / ⚠️  REVIEW NEEDED]

```python
# From: tests/fixtures/test_pins.py
TEST_PIN = "123456"  # ⚠️  Acceptable for tests ONLY
TEST_PUK = "12345678"  # ⚠️  Acceptable for tests ONLY
```

**Note**: Hardcoded test PINs are acceptable IF:
1. Used only in test code
2. Clearly marked as test data
3. Never used with production YubiKeys

---

## PIN Storage and Persistence

### Storage Audit
```bash
# Search results
PIN save/store operations: [N]
PIN cache operations: [N]
PIN in configuration: [N]
```

**Status**: [✅ NO STORAGE / ❌ PIN STORAGE FOUND]

### Findings
[List any PIN storage mechanisms found]

**CRITICAL**: PINs should NEVER be stored or cached

---

## PIN Comparison Security

### Comparison Methods
```bash
# Direct comparison
pin == other_pin: [N] instances

# Constant-time comparison
secrets.compare_digest(): [N] instances
hmac.compare_digest(): [N] instances
```

### Analysis

#### Insecure Comparison ⚠️
```python
# Direct comparison (timing attack vulnerable)
if pin == stored_hash:  # ⚠️  TIMING ATTACK
    return True
```

#### Secure Comparison ✅
```python
# Constant-time comparison
if secrets.compare_digest(pin_hash, stored_hash):  # ✅ SECURE
    return True
```

**Status**: [✅ SECURE / ⚠️  NEEDS IMPROVEMENT]

**Recommendation**: Use `secrets.compare_digest()` for PIN/hash comparison

---

## Error Message Safety

### Error Message Audit
```bash
# PIN in exceptions
Exceptions mentioning PIN: [N]
Error messages with PIN value: [N]
```

### Findings

#### Safe Error Messages ✅
```python
# From: src/validators/pin_validator.py:25
if not pin.isdigit():
    raise ValueError("PIN must contain only digits")  # ✅ No PIN value
```

#### Unsafe Error Messages ❌
[List any error messages exposing PIN values]

**Status**: [✅ SAFE / ❌ INFORMATION DISCLOSURE]

---

## Best Practices Compliance

### PIN Security Checklist
- [ ] ✅ No PINs in log statements
- [ ] ✅ No PINs in print statements
- [ ] ✅ @sanitize_logging on PIN functions
- [ ] ✅ PIN validation before use
- [ ] ✅ Secure input (getpass)
- [ ] ✅ No hardcoded PINs in production
- [ ] ✅ No PIN storage/caching
- [ ] ✅ Constant-time comparison
- [ ] ✅ Safe error messages
- [ ] ✅ No PIN in f-strings/format

**Compliance Score**: [X/10] ([XX%])

---

## Code Examples

### ✅ Secure PIN Handling Pattern
```python
import getpass
from typing import Optional
from src.validators.pin_validator import validate_pin_format
from src.utils.logging_utils import sanitize_logging

@sanitize_logging
def authenticate_piv(yubikey, pin: Optional[str] = None) -> bool:
    """
    Authenticate with PIV applet.

    Args:
        yubikey: YubiKey device
        pin: User PIN (will be sanitized in logs)

    Returns:
        True if authentication successful

    Security:
        - PIN never logged
        - PIN validated before use
        - Uses secure input if not provided
    """
    if pin is None:
        pin = getpass.getpass("Enter PIN: ")

    if not validate_pin_format(pin):
        raise ValueError("Invalid PIN format: must be 6-8 digits")

    try:
        yubikey.authenticate(pin)
        logger.info("PIV authentication successful")  # No PIN value
        return True
    except AuthenticationError:
        logger.error("PIV authentication failed")  # No PIN value
        return False
```

### ❌ Insecure PIN Handling (DO NOT USE)
```python
def authenticate_insecure(yubikey, pin):
    # ❌ No @sanitize_logging
    # ❌ No validation
    # ❌ Logs PIN value
    logger.info(f"Authenticating with PIN: {pin}")  # CRITICAL ISSUE

    try:
        yubikey.authenticate(pin)
    except Exception as e:
        # ❌ Exposes PIN in error
        print(f"Failed with PIN {pin}: {e}")  # CRITICAL ISSUE

    # ❌ Stores PIN
    self.last_pin = pin  # CRITICAL ISSUE
```

---

## Recommendations

### Critical (Fix Immediately)
1. **Remove PIN logging**: [List locations]
2. **Remove hardcoded PINs**: [List locations]
3. **Remove PIN storage**: [List locations]

### High Priority (This Week)
1. **Add @sanitize_logging**: [List functions]
2. **Add PIN validation**: [List operations]
3. **Replace input() with getpass()**: [List locations]

### Medium Priority (This Sprint)
1. **Implement constant-time comparison**: For PIN hashes
2. **Review test PINs**: Ensure properly isolated
3. **Audit error messages**: Remove any PIN exposure

---

## Testing PIN Security

### Recommended Tests
```python
def test_no_pin_in_logs(caplog):
    """Verify PIN never appears in logs"""
    pin = "123456"
    authenticate(yubikey, pin)

    for record in caplog.records:
        assert "123456" not in record.message
        assert pin not in record.message

def test_sanitize_logging_decorator():
    """Verify decorator redacts PINs"""
    @sanitize_logging
    def func_with_pin(pin):
        return f"PIN is {pin}"

    result = func_with_pin("123456")
    assert "123456" not in result
    assert "[REDACTED]" in result
```

---

## Next Steps

1. **Fix critical issues**: [List priority fixes]
2. **Add missing decorators**: [N] functions need @sanitize_logging
3. **Enhance validation**: [Improvements needed]
4. **Review test code**: Ensure test PINs properly isolated
5. **Re-audit**: Run /pin-security-check after fixes

---

**Report Generated**: [Timestamp]
**Next Audit**: [Recommended date]
**Auditor**: PIN Security Checker v1.0
```

### 11. Display Security Check Summary

```
╔════════════════════════════════════════════════════╗
║          PIN SECURITY CHECK COMPLETE               ║
╚════════════════════════════════════════════════════╝

PIN SECURITY STATUS: 🟢 SECURE / 🟡 NEEDS ATTENTION / 🔴 CRITICAL

SECURITY CONTROLS:
  No PIN Logging:        ✅ PASS
  @sanitize_logging:     ⚠️  [N] functions missing decorator
  PIN Validation:        ✅ PASS
  Secure Input:          ✅ Using getpass
  No Hardcoded PINs:     ✅ PASS
  No PIN Storage:        ✅ PASS
  Constant-Time Comparison: ⚠️  NEEDS IMPROVEMENT
  Error Message Safety:  ✅ PASS

FINDINGS:
  🔴 Critical:  [N]
  🟠 High:      [N]
  🟡 Medium:    [N]
  🟢 Low:       [N]

COMPLIANCE: [8/10] (80%) ⚠️

CRITICAL ACTIONS REQUIRED:
  » [Critical issue 1]
  » [Critical issue 2]

HIGH PRIORITY:
  » Add @sanitize_logging to [N] functions
  » Implement constant-time PIN comparison
  » [Other high priority items]

Full Report: PIN_SECURITY_REPORT.md

OVERALL ASSESSMENT:
  PIN handling follows security best practices with minor
  improvements needed. No critical security issues found.
```

## Key Features

- **Comprehensive Audit**: All PIN handling reviewed
- **Critical Security**: Focuses on PIN logging and exposure
- **Decorator Validation**: Checks @sanitize_logging usage
- **Input Security**: Verifies secure PIN input methods
- **Hardcoded Detection**: Finds hardcoded credentials
- **Storage Check**: Ensures no PIN persistence
- **Detailed Reporting**: PIN_SECURITY_REPORT.md
- **Code Examples**: Secure vs insecure patterns

## When to Use /pin-security-check

- After implementing PIN handling
- Before security reviews
- Before releases
- After code refactoring
- When adding new YubiKey operations
- Monthly security audits
- After onboarding new developers
- For compliance audits

## Best Practices

1. **Regular Audits**: Run after any PIN handling changes
2. **Never Log PINs**: Use @sanitize_logging everywhere
3. **Validate Always**: Check PIN format before use
4. **Secure Input**: Always use getpass for PIN entry
5. **No Storage**: Never store or cache PINs
6. **Constant-Time**: Use secrets.compare_digest for comparisons
7. **Safe Errors**: Never expose PINs in error messages
8. **Test Isolation**: Keep test PINs separate from production
