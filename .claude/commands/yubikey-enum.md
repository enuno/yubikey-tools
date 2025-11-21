---
description: "Safely enumerate connected YubiKeys with read-only operations, displaying serial numbers, firmware versions, and current configuration"
allowed-tools: ["Bash(ykman:list)", "Bash(ykman:info)", "Bash(ykman:openpgp:info)", "Bash(ykman:piv:info)", "Bash(ykman:fido:info)", "Bash(ykman:oath:list)", "Read"]
author: "YubiKey Tools Team"
version: "1.0"
---

# YubiKey Enumeration

## Purpose
Enumerate all connected YubiKey devices and display detailed information including serial numbers, firmware versions, enabled applications, and current configuration. This is a **read-only** operation that never modifies YubiKey state.

## YubiKey Enumeration Steps

### 1. List All Connected YubiKeys

```bash
# List all connected YubiKey devices
!ykman list

# Count devices
!ykman list | wc -l
```

### 2. Get Detailed Info for Each YubiKey

```bash
# For each detected YubiKey, get full information
!ykman info

# Get info in JSON format for parsing
!ykman info --json 2>/dev/null || ykman info
```

### 3. Check OpenPGP Configuration

```bash
# Get OpenPGP applet information
!ykman openpgp info 2>/dev/null || echo "OpenPGP not enabled or not accessible"

# List OpenPGP key slots
!ykman openpgp info --json 2>/dev/null || echo "OpenPGP info not available"
```

### 4. Check PIV Configuration

```bash
# Get PIV applet information
!ykman piv info 2>/dev/null || echo "PIV not enabled or not accessible"

# List PIV certificates
!ykman piv info --json 2>/dev/null || echo "PIV info not available"
```

### 5. Check FIDO2 Configuration

```bash
# Get FIDO2 information
!ykman fido info 2>/dev/null || echo "FIDO2 not enabled or not accessible"

# Check for resident credentials
!ykman fido credentials list 2>/dev/null || echo "FIDO2 credentials not accessible (may require PIN)"
```

### 6. Check OATH Configuration

```bash
# List OATH credentials
!ykman oath accounts list 2>/dev/null || echo "OATH not enabled or not accessible"

# Count OATH credentials
!ykman oath accounts list 2>/dev/null | wc -l || echo "0"
```

### 7. Identify Test vs Production Devices

```bash
# Read test device registry
@Read: tests/fixtures/TEST_DEVICES.md

# Compare serial numbers with test device list
# Flag devices NOT in test registry as production
```

### 8. Generate YubiKey Inventory Report

Create **YUBIKEY_INVENTORY.md**:

```markdown
# YubiKey Inventory Report

**Scan Date**: [ISO 8601 timestamp]
**Scan Tool**: ykman [version]
**Total Devices**: [N]

---

## Connected Devices Summary

| Serial Number | Firmware | Model | Form Factor | Status |
|--------------|----------|-------|-------------|--------|
| [serial] | [version] | YubiKey 5 NFC | USB-A | 🧪 Test Device |
| [serial] | [version] | YubiKey 5C | USB-C | ⚠️  Production |
| [serial] | [version] | YubiKey 5 Nano | USB-A | 🧪 Test Device |

**Legend**:
- 🧪 Test Device: Listed in `tests/fixtures/TEST_DEVICES.md`
- ⚠️  Production: NOT in test device registry
- ✅ Configured: Has active configuration
- ⭕ Blank: Factory default state

---

## Device #1: [Serial Number]

### Basic Information
- **Serial Number**: [serial]
- **Firmware Version**: [X.Y.Z]
- **Model**: YubiKey 5 NFC
- **Form Factor**: USB-A with NFC
- **USB Interface**: CCID + HID
- **NFC Interface**: Enabled

### Device Status
- **Classification**: 🧪 Test Device (in test registry)
- **Configuration**: ✅ Configured
- **Last Seen**: [timestamp]

### Enabled Applications
- ✅ OpenPGP
- ✅ PIV
- ✅ FIDO2
- ✅ OATH
- ❌ FIDO U2F (legacy)
- ❌ OTP

### OpenPGP Applet
- **Version**: [version]
- **PIN Retry Counter**: [N/3]
- **Signature Key**: [Present/Empty]
- **Encryption Key**: [Present/Empty]
- **Authentication Key**: [Present/Empty]
- **Attestation**: [Available/Not available]

### PIV Applet
- **Version**: [version]
- **PIN Retry Counter**: [N/3]
- **PUK Retry Counter**: [N/3]
- **Management Key**: [Default/Custom]
- **Configured Slots**:
  - 9a (Authentication): [Certificate present]
  - 9c (Digital Signature): [Empty]
  - 9d (Key Management): [Empty]
  - 9e (Card Authentication): [Empty]

### FIDO2 Applet
- **Version**: [version]
- **PIN Configured**: [Yes/No]
- **PIN Retry Counter**: [N/8]
- **Resident Credentials**: [N]
- **Always Require UV**: [Enabled/Disabled]
- **Minimum PIN Length**: [digits]

### OATH Applet
- **Version**: [version]
- **Password Protected**: [Yes/No]
- **TOTP Credentials**: [N]
- **HOTP Credentials**: [N]
- **Total Credentials**: [N]

---

## Device #2: [Serial Number]

[Repeat structure for each device]

---

## Test Device Registry Status

### Registered Test Devices ([N] total)
1. **Serial [XXXXXX]**: ✅ Connected
2. **Serial [XXXXXX]**: ❌ Not connected
3. **Serial [XXXXXX]**: ✅ Connected

### Unregistered Devices ([N] total)
1. **Serial [XXXXXX]**: ⚠️  WARNING - Production device connected
2. **Serial [XXXXXX]**: ⚠️  WARNING - Production device connected

**Action Required**:
- If unregistered devices are test devices, add to `tests/fixtures/TEST_DEVICES.md`
- If production devices, disconnect before running integration tests

---

## Firmware Compatibility Matrix

| Firmware Version | Device Count | Support Status | Notes |
|------------------|--------------|----------------|-------|
| 5.7.x | [N] | ✅ Fully Supported | Latest stable |
| 5.4.x | [N] | ✅ Supported | Recommended upgrade |
| 5.2.x | [N] | ⚠️  Legacy Support | Some features limited |
| 4.3.x | [N] | ⚠️  EOL | Upgrade recommended |

---

## Application Enablement Summary

| Application | Devices Enabled | Devices Disabled |
|-------------|-----------------|------------------|
| OpenPGP | [N] | [N] |
| PIV | [N] | [N] |
| FIDO2 | [N] | [N] |
| OATH | [N] | [N] |
| OTP | [N] | [N] |

---

## Configuration Statistics

### PIN Status
- Devices with default PIN: [N]
- Devices with custom PIN: [N]
- Devices with PIN locked: [N]

### Retry Counters
- Average PIN retries remaining: [N/3]
- Devices at risk (1-2 retries): [N]
- Devices locked (0 retries): [N]

### Certificate Status (PIV)
- Devices with certificates: [N]
- Total certificates installed: [N]
- Expired certificates: [N]
- Certificates expiring soon (< 30 days): [N]

### Credentials (OATH)
- Total OATH credentials across devices: [N]
- Average credentials per device: [N]
- Devices password-protected: [N]

---

## Health and Maintenance

### Devices Requiring Attention

#### Critical ⚠️
- **Serial [XXXXXX]**: PIN locked (0 retries)
- **Serial [XXXXXX]**: Expired certificate in PIV slot 9a

#### Warning 🟡
- **Serial [XXXXXX]**: Only 1 PIN retry remaining
- **Serial [XXXXXX]**: Certificate expiring in 15 days
- **Serial [XXXXXX]**: Old firmware (4.3.x)

#### Info ℹ️
- **Serial [XXXXXX]**: Using default management key
- **Serial [XXXXXX]**: Factory default configuration

---

## Recommendations

### Security
1. **Change default PINs**: [N] devices still using defaults
2. **Update firmware**: [N] devices on legacy firmware
3. **Backup keys**: Ensure all devices have documented backups
4. **Review certificates**: [N] certificates expiring soon

### Configuration
1. **Enable touch policy**: For enhanced security on PIV/OpenPGP operations
2. **Set custom management key**: [N] devices using default
3. **Configure attestation**: For FIDO2 credential verification

### Testing
1. **Document unregistered devices**: Add [N] devices to TEST_DEVICES.md or disconnect
2. **Reset test devices**: Ensure clean state for integration tests
3. **Verify test coverage**: All test devices in use?

---

**Report Generated**: [Timestamp]
**Next Scan**: [Recommended date]
**Tool Version**: ykman [version]
```

### 9. Display Enumeration Summary

```
╔════════════════════════════════════════════════════╗
║          YUBIKEY ENUMERATION COMPLETE              ║
╚════════════════════════════════════════════════════╝

DEVICES FOUND: [N]

DEVICE LISTING:
  1. Serial [XXXXXX] - YubiKey 5 NFC (FW: 5.7.1)
     Status: 🧪 Test Device
     Apps: OpenPGP, PIV, FIDO2, OATH
     PIN: ✅ [3/3 retries]

  2. Serial [XXXXXX] - YubiKey 5C (FW: 5.4.3)
     Status: ⚠️  Production Device
     Apps: PIV, FIDO2
     PIN: ⚠️  [1/3 retries] - CAUTION

  3. Serial [XXXXXX] - YubiKey 5 Nano (FW: 5.7.1)
     Status: 🧪 Test Device
     Apps: OpenPGP, PIV, FIDO2, OATH
     PIN: ✅ [3/3 retries]

TEST DEVICE STATUS:
  Registered: [N] devices
  Connected: [N] test devices
  Missing: [N] test devices not connected

WARNINGS:
  ⚠️  [N] production devices connected
  ⚠️  [N] devices with low PIN retries
  ⚠️  [N] certificates expiring soon
  ⚠️  [N] devices on legacy firmware

HEALTH STATUS:
  🟢 Healthy: [N] devices
  🟡 Warning: [N] devices
  🔴 Critical: [N] devices

APPLICATION SUMMARY:
  OpenPGP:  [N] enabled
  PIV:      [N] enabled
  FIDO2:    [N] enabled
  OATH:     [N] enabled

Full Report: YUBIKEY_INVENTORY.md

RECOMMENDED ACTIONS:
  » Review production device warnings
  » Reset PIN retry counters if needed
  » Update test device registry
  » Check certificate expiration
  » Consider firmware updates
```

## Key Features

- **Read-Only**: Never modifies YubiKey state
- **Comprehensive**: Checks all applications (OpenPGP, PIV, FIDO2, OATH)
- **Test Device Detection**: Identifies test vs production devices
- **Health Monitoring**: PIN retry counters, certificate expiration
- **Detailed Reporting**: Generates YUBIKEY_INVENTORY.md
- **Safety Warnings**: Alerts for production devices, low retries
- **Firmware Tracking**: Documents firmware versions for compatibility

## Safety Guarantees

This command is **completely safe** and:
- ✅ Only reads YubiKey information
- ✅ Never modifies PIN, PUK, or management keys
- ✅ Never writes certificates or keys
- ✅ Never changes configuration
- ✅ Never resets or formats devices
- ✅ Does not require PINs (except for FIDO2 credential listing)

## When to Use /yubikey-enum

- Before running integration tests
- To verify test device availability
- When troubleshooting YubiKey issues
- To check device health and PIN retries
- Before batch configuration operations
- To document device inventory
- When onboarding new YubiKeys
- For compliance and audit requirements

## Best Practices

1. **Regular Scans**: Run weekly to monitor device health
2. **Update Registry**: Keep TEST_DEVICES.md current
3. **Monitor Retries**: Watch for low PIN retry counters
4. **Track Firmware**: Ensure compatibility with your tools
5. **Certificate Monitoring**: Track expiration dates
6. **Disconnect Production**: Never test with production YubiKeys
7. **Document Changes**: Update inventory after device changes
8. **Security Awareness**: Always verify test/production status
