---
description: "Run integration tests with real YubiKey hardware after verifying test device presence and obtaining user approval"
allowed-tools: ["Read", "Bash(ykman:list)", "Bash(ykman:info)", "Bash(pytest:*)"]
author: "YubiKey Tools Testing Team"
version: "1.0"
---

# Test Hardware

## Purpose
Execute integration tests against real YubiKey hardware with comprehensive safety checks. This command **requires explicit user approval** before performing ANY operations on physical YubiKeys.

## ⚠️ CRITICAL SAFETY NOTICE

This command will:
- ⚠️  Perform **WRITE OPERATIONS** on YubiKeys
- ⚠️  Modify YubiKey configuration during tests
- ⚠️  Reset test YubiKeys to known state
- ⚠️  Requires test devices ONLY (never production keys)

**MANDATORY REQUIREMENTS**:
1. All YubiKeys must be documented in `tests/fixtures/TEST_DEVICES.md`
2. User must explicitly approve test execution
3. Production YubiKeys must be disconnected

## Hardware Testing Steps

### 1. Read Test Device Registry

```bash
# Load test device documentation
@Read: tests/fixtures/TEST_DEVICES.md
```

### 2. Enumerate Connected YubiKeys

```bash
# List all connected devices
!ykman list

# Get detailed info for verification
!ykman info --json 2>/dev/null || ykman info
```

### 3. Verify Test Devices Only

Compare connected devices with registered test devices:

```markdown
## Test Device Verification

### Registered Test Devices
[List from TEST_DEVICES.md]

### Connected Devices
[List from ykman list]

### Verification Status
- ✅ All connected devices are registered test devices
- ⚠️  WARNING: Unregistered devices detected
- ❌ BLOCKED: Production devices connected
```

### 4. Prompt for User Confirmation

**REQUIRED USER APPROVAL**:

```
╔════════════════════════════════════════════════════╗
║          ⚠️  HARDWARE TEST CONFIRMATION  ⚠️          ║
╚════════════════════════════════════════════════════╝

The following YubiKeys will be MODIFIED during testing:

Device #1:
  Serial: [XXXXXX]
  Model: YubiKey 5 NFC
  Firmware: 5.7.1
  Test Registry: ✅ Registered

Device #2:
  Serial: [XXXXXX]
  Model: YubiKey 5C
  Firmware: 5.4.3
  Test Registry: ✅ Registered

OPERATIONS THAT WILL BE PERFORMED:
  - Reset to factory defaults (some tests)
  - Write test certificates to PIV slots
  - Create/delete FIDO2 credentials
  - Modify OATH credentials
  - Change PINs (will be restored)
  - Configure touch policies

⚠️  WARNING: These operations will MODIFY the YubiKeys!

Do you want to proceed with hardware testing?
  [Y] Yes, proceed with testing
  [N] No, cancel hardware tests
  [L] List tests that will run
```

**If user selects NO**: Exit immediately, do not proceed

**If user selects LIST**: Show test plan, then re-prompt

**If user selects YES**: Proceed to step 5

### 5. Pre-Test Device State Capture

```bash
# Capture current state of each test device
!ykman info > pre-test-state-[serial].txt
!ykman piv info >> pre-test-state-[serial].txt 2>/dev/null
!ykman openpgp info >> pre-test-state-[serial].txt 2>/dev/null
!ykman fido info >> pre-test-state-[serial].txt 2>/dev/null
!ykman oath accounts list >> pre-test-state-[serial].txt 2>/dev/null
```

### 6. Run Integration Tests

```bash
# Run all hardware-marked tests
!pytest tests/integration/ -v -m hardware --tb=short

# Generate detailed test report
!pytest tests/integration/ -v -m hardware --html=test-report-hardware.html --self-contained-html

# Generate coverage report
!pytest tests/integration/ -v -m hardware --cov=src --cov-report=html --cov-report=term
```

### 7. Post-Test Device State Verification

```bash
# Capture post-test state
!ykman info > post-test-state-[serial].txt
!ykman piv info >> post-test-state-[serial].txt 2>/dev/null
!ykman openpgp info >> post-test-state-[serial].txt 2>/dev/null
!ykman fido info >> post-test-state-[serial].txt 2>/dev/null
!ykman oath accounts list >> post-test-state-[serial].txt 2>/dev/null

# Compare pre and post state
!diff pre-test-state-[serial].txt post-test-state-[serial].txt
```

### 8. Generate Hardware Test Report

Create **HARDWARE_TEST_REPORT.md**:

```markdown
# Hardware Integration Test Report

**Test Date**: [ISO 8601 timestamp]
**Test Duration**: [HH:MM:SS]
**Devices Tested**: [N]
**User Approval**: ✅ Obtained

---

## Executive Summary

| Metric | Value | Status |
|--------|-------|--------|
| Total Tests | [N] | - |
| Passed | [N] | ✅ |
| Failed | [N] | ❌ |
| Skipped | [N] | ⏭️ |
| Test Coverage | [X%] | [PASS/FAIL] |
| Duration | [HH:MM:SS] | - |

**Overall Status**: ✅ ALL PASS / ⚠️  SOME FAILURES / ❌ CRITICAL FAILURES

---

## Test Devices

### Device #1: Serial [XXXXXX]
- **Model**: YubiKey 5 NFC
- **Firmware**: 5.7.1
- **Pre-Test Status**: [description]
- **Post-Test Status**: [description]
- **State Changes**: [list of intentional changes]
- **Tests Run**: [N]
- **Tests Passed**: [N]
- **Tests Failed**: [N]

### Device #2: Serial [XXXXXX]
[Same structure]

---

## Test Results by Category

### YubiKey Detection and Connection
- ✅ `test_list_yubikeys` - Lists all connected devices
- ✅ `test_get_yubikey_info` - Retrieves device information
- ✅ `test_firmware_version_check` - Validates firmware compatibility
- ✅ `test_connection_timeout` - Handles connection timeouts
- ❌ `test_disconnection_handling` - FAILED: [reason]

### FIDO2 Operations
- ✅ `test_create_fido2_credential` - Creates resident credential
- ✅ `test_delete_fido2_credential` - Deletes credential
- ✅ `test_list_fido2_credentials` - Lists resident credentials
- ✅ `test_fido2_attestation` - Verifies attestation chain
- ✅ `test_fido2_pin_management` - Changes and validates PIN

### PIV Operations
- ✅ `test_generate_piv_key` - Generates key in slot 9a
- ✅ `test_import_certificate` - Imports test certificate
- ✅ `test_export_certificate` - Exports certificate
- ✅ `test_piv_authentication` - Authenticates with PIN
- ✅ `test_touch_policy` - Verifies touch requirement

### OATH Operations
- ✅ `test_add_oath_credential` - Adds TOTP credential
- ✅ `test_generate_totp` - Generates TOTP code
- ✅ `test_delete_oath_credential` - Removes credential
- ✅ `test_oath_password_protection` - Sets and validates password

### OpenPGP Operations
- ✅ `test_openpgp_key_generation` - Generates OpenPGP keys
- ✅ `test_openpgp_signature` - Creates signature
- ✅ `test_openpgp_encryption` - Encrypts/decrypts data
- ✅ `test_openpgp_attestation` - Validates attestation

### Error Handling and Edge Cases
- ✅ `test_invalid_pin` - Rejects invalid PIN format
- ✅ `test_pin_retry_counter` - Tracks failed attempts
- ✅ `test_device_disconnection` - Handles mid-operation disconnect
- ✅ `test_concurrent_access` - Prevents race conditions
- ✅ `test_malformed_certificate` - Rejects invalid certificates

---

## Failed Tests Detail

### Test: `test_disconnection_handling`
- **File**: `tests/integration/test_error_handling.py:142`
- **Error**: AssertionError: Expected YubiKeyConnectionError, got None
- **Output**:
  ```
  [Test output]
  ```
- **Reason**: Device remained connected during test
- **Action Required**: Review test logic or hardware behavior

---

## Test Coverage Report

### Overall Coverage
- **Lines Covered**: [XXXX]/[YYYY] ([X%])
- **Branches Covered**: [XXX]/[YYY] ([X%])
- **Functions Covered**: [XXX]/[YYY] ([X%])

### Module Coverage
| Module | Statements | Missing | Coverage |
|--------|------------|---------|----------|
| src/core/operations/yubikey_detection.py | 150 | 5 | 97% |
| src/core/operations/fido2_operations.py | 200 | 15 | 93% |
| src/core/operations/piv_operations.py | 180 | 10 | 94% |
| src/core/operations/oath_operations.py | 120 | 8 | 93% |
| src/validators/attestation_validator.py | 100 | 2 | 98% |

**Coverage Status**: ✅ PASS (≥85% overall, ≥95% security modules)

---

## Device State Changes

### Intentional Changes (Expected)
- Test certificates installed in PIV slots (will be cleaned)
- FIDO2 test credentials created and deleted
- OATH test credentials created and deleted
- PINs changed during tests (restored to defaults)
- Touch policies temporarily modified

### Unintended Changes (Requires Review)
[Any unexpected state changes]

---

## Performance Metrics

| Operation | Avg Time | Min Time | Max Time | Count |
|-----------|----------|----------|----------|-------|
| YubiKey Detection | [X.XX]s | [X.XX]s | [X.XX]s | [N] |
| PIV Key Generation | [X.XX]s | [X.XX]s | [X.XX]s | [N] |
| FIDO2 Credential Creation | [X.XX]s | [X.XX]s | [X.XX]s | [N] |
| Certificate Import | [X.XX]s | [X.XX]s | [X.XX]s | [N] |
| TOTP Generation | [X.XX]s | [X.XX]s | [X.XX]s | [N] |

---

## Firmware Compatibility

| Firmware Version | Tests Run | Tests Passed | Compatibility |
|------------------|-----------|--------------|---------------|
| 5.7.1 | [N] | [N] | ✅ Fully Compatible |
| 5.4.3 | [N] | [N] | ✅ Compatible |
| 5.2.7 | [N] | [N] | ⚠️  Limited Support |

---

## Security Test Results

### Negative Testing (Attack Scenarios)
- ✅ Invalid PIN format rejected
- ✅ Expired certificates rejected
- ✅ Malformed attestations rejected
- ✅ Replay attacks prevented
- ✅ Buffer overflow attempts handled
- ✅ PIN brute force protection active

### Sensitive Data Handling
- ✅ No PINs in test output
- ✅ No private keys exposed
- ✅ Logging properly sanitized
- ✅ Credentials cleaned after tests

---

## Recommendations

### Immediate Actions
1. **Fix failing tests**: [List]
2. **Review device state**: [Any concerns]
3. **Update test fixtures**: [If needed]

### Test Improvements
1. **Add more edge cases**: [Suggestions]
2. **Improve test coverage**: Target modules < 90%
3. **Performance optimization**: [Slow tests to optimize]

### Device Maintenance
1. **Reset test devices**: Recommended after extensive testing
2. **Update firmware**: [Devices on old firmware]
3. **Document changes**: Update TEST_DEVICES.md

---

## Cleanup Status

### Test Artifacts Removed
- ✅ Test certificates deleted from PIV slots
- ✅ FIDO2 test credentials removed
- ✅ OATH test credentials removed
- ✅ PINs restored to documented defaults
- ✅ Touch policies reset

### Files Generated
- `test-report-hardware.html` - Detailed HTML test report
- `htmlcov/index.html` - Coverage report
- `pre-test-state-*.txt` - Pre-test device states
- `post-test-state-*.txt` - Post-test device states
- `HARDWARE_TEST_REPORT.md` - This report

---

**Report Generated**: [Timestamp]
**Test Environment**: [OS, Python version, ykman version]
**Next Test Run**: [Recommended date]
```

### 9. Display Test Summary

```
╔════════════════════════════════════════════════════╗
║       HARDWARE INTEGRATION TESTS COMPLETE          ║
╚════════════════════════════════════════════════════╝

DEVICES TESTED: [N]
  - Serial [XXXXXX]: YubiKey 5 NFC (FW 5.7.1)
  - Serial [XXXXXX]: YubiKey 5C (FW 5.4.3)

TEST RESULTS:
  Total Tests: [N]
  ✅ Passed: [N]
  ❌ Failed: [N]
  ⏭️  Skipped: [N]

SUCCESS RATE: [XX.X%]

TEST COVERAGE:
  Overall: [XX.X%] (Target: ≥85%)
  Security Modules: [XX.X%] (Target: ≥95%)
  Status: ✅ PASS / ❌ FAIL

DURATION: [HH:MM:SS]

FAILED TESTS:
  1. test_disconnection_handling (error_handling.py:142)
     └─ Reason: [brief description]

DEVICE STATUS:
  ✅ All devices cleaned and restored
  ✅ No unintended state changes
  ✅ Ready for next test run

PERFORMANCE:
  Fastest Test: [test_name] ([X.XX]s)
  Slowest Test: [test_name] ([X.XX]s)
  Average: [X.XX]s per test

Full Reports:
  - HARDWARE_TEST_REPORT.md
  - test-report-hardware.html
  - htmlcov/index.html

NEXT STEPS:
  » Fix [N] failing tests
  » Review slow tests for optimization
  » Update test coverage for [modules]
  » Consider firmware compatibility testing
```

## Key Features

- **Safety First**: Verifies test devices before ANY operations
- **User Approval**: Explicit confirmation required
- **State Tracking**: Captures pre/post-test device state
- **Comprehensive Testing**: FIDO2, PIV, OATH, OpenPGP
- **Detailed Reporting**: Multiple report formats (MD, HTML, coverage)
- **Cleanup Verification**: Ensures devices restored after tests
- **Security Testing**: Includes negative tests and attack scenarios
- **Performance Tracking**: Monitors test execution times

## Safety Guarantees

This command:
- ✅ Verifies devices against TEST_DEVICES.md registry
- ✅ Blocks execution if production devices detected
- ✅ Requires explicit user approval
- ✅ Captures pre-test state for comparison
- ✅ Cleans up test artifacts
- ✅ Restores devices to known state
- ⚠️  **WILL MODIFY** test YubiKeys (with approval)

## When to Use /test-hardware

- Before major releases
- After core library changes
- When adding new YubiKey operations
- To validate firmware compatibility
- After security fixes
- Weekly during active development
- Before merging PRs affecting hardware operations

## Best Practices

1. **Always Verify**: Check TEST_DEVICES.md is current
2. **Disconnect Production**: Remove all production YubiKeys
3. **Review Plan**: Use [L] option to see test plan before proceeding
4. **Monitor Output**: Watch for unexpected failures
5. **Clean State**: Reset devices if tests leave artifacts
6. **Document Failures**: Investigate and fix failing tests promptly
7. **Regular Testing**: Run weekly to catch regressions early
8. **Firmware Matrix**: Test across supported firmware versions
