---
description: "Create secure backups of YubiKey public keys, configuration metadata, and certificates for disaster recovery"
allowed-tools: ["Read", "Write", "Bash(ykman:list)", "Bash(ykman:info)", "Bash(ykman:piv:info)", "Bash(ykman:openpgp:info)", "Bash(ykman:fido:info)", "Bash(ykman:oath:list)", "Bash(gpg:*)", "Bash(ssh-add:*)", "Bash(openssl:*)"]
author: "YubiKey Tools Team"
version: "1.0"
---

# YubiKey Backup Tool

## Purpose
Create comprehensive, secure backups of YubiKey public keys, certificates, configuration metadata, and OATH credentials for disaster recovery. This tool exports **public information only** - private keys never leave the YubiKey hardware.

## Safety Level
**CAUTION** - File operations and public key exports, no YubiKey writes or private key access

## Prerequisites
- YubiKey connected
- YubiKey Manager (ykman) installed
- GPG installed (for OpenPGP operations)
- OpenSSL installed (for certificate operations)
- Write permissions to backup directory

## What Gets Backed Up

### ✅ Safe to Backup (Public Information)
- **GPG Public Keys**: Exported from YubiKey OpenPGP applet
- **SSH Public Keys**: Derived from GPG authentication subkey
- **PIV Certificates**: X.509 certificates from all PIV slots
- **OATH Credentials**: TOTP/HOTP account names and parameters (NOT secrets)
- **Device Metadata**: Serial number, firmware version, form factor
- **Configuration**: Enabled applications, PIN retry counters, touch policies

### ❌ Never Backed Up (Remains on YubiKey)
- Private keys (cannot be exported from YubiKey by design)
- PINs, PUKs, management keys
- OATH credential secrets
- FIDO2 resident credential private keys

## Backup Steps

### 1. Initialize Backup Session

```bash
# Set backup directory
BACKUP_DIR="$HOME/.yubikey-backups/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Create backup metadata file
echo "# YubiKey Backup" > "$BACKUP_DIR/BACKUP_MANIFEST.md"
echo "**Date**: $(date -u +"%Y-%m-%d %H:%M:%S UTC")" >> "$BACKUP_DIR/BACKUP_MANIFEST.md"
echo "**Backup Tool**: yubikey-backup v1.0" >> "$BACKUP_DIR/BACKUP_MANIFEST.md"
echo "" >> "$BACKUP_DIR/BACKUP_MANIFEST.md"
```

### 2. Enumerate Connected YubiKeys

```bash
# List all connected devices
!ykman list

# For each YubiKey, create separate backup directory
SERIAL=$(ykman info | grep "Serial number" | awk '{print $3}')
DEVICE_DIR="$BACKUP_DIR/yubikey-$SERIAL"
mkdir -p "$DEVICE_DIR"
```

### 3. Export Device Information

```bash
# Export full device info
!ykman info > "$DEVICE_DIR/device_info.txt"
!ykman info --json > "$DEVICE_DIR/device_info.json" 2>/dev/null

# Document device classification
@Read: tests/fixtures/TEST_DEVICES.md
# Check if device is test or production
```

### 4. Export GPG/OpenPGP Keys

```bash
# Export OpenPGP public key
!gpg --card-status > "$DEVICE_DIR/gpg_card_status.txt" 2>&1

# Get key ID from card
KEY_ID=$(gpg --card-status 2>/dev/null | grep "General key info" | awk '{print $NF}' | cut -d'/' -f2)

if [ -n "$KEY_ID" ]; then
  # Export public key in ASCII-armored format
  !gpg --armor --export "$KEY_ID" > "$DEVICE_DIR/gpg_public_key.asc"

  # Export public key in binary format
  !gpg --export "$KEY_ID" > "$DEVICE_DIR/gpg_public_key.gpg"

  # Export minimal public key (no signatures)
  !gpg --armor --export-options export-minimal --export "$KEY_ID" > "$DEVICE_DIR/gpg_public_key_minimal.asc"

  # Document key details
  !gpg --with-colons --list-keys "$KEY_ID" > "$DEVICE_DIR/gpg_key_details.txt"

  echo "✅ Exported GPG public key: $KEY_ID"
else
  echo "⚠️  No GPG key found on YubiKey"
fi
```

### 5. Export SSH Public Key

```bash
# Export SSH public key from GPG authentication subkey
if [ -n "$KEY_ID" ]; then
  # Get authentication subkey
  AUTH_KEYGRIP=$(gpg --with-keygrip --list-secret-keys "$KEY_ID" 2>/dev/null | grep -A 1 "\[A\]" | tail -1 | awk '{print $3}')

  if [ -n "$AUTH_KEYGRIP" ]; then
    # Export SSH format
    !gpg --export-ssh-key "$KEY_ID" > "$DEVICE_DIR/ssh_public_key.pub" 2>/dev/null

    # Alternative: Use ssh-add if gpg-agent is running
    !ssh-add -L 2>/dev/null | grep "cardno:" > "$DEVICE_DIR/ssh_public_key_from_agent.pub"

    echo "✅ Exported SSH public key"
  else
    echo "⚠️  No SSH authentication subkey found"
  fi
fi
```

### 6. Export PIV Certificates

```bash
# Create PIV directory
mkdir -p "$DEVICE_DIR/piv"

# Get PIV info
!ykman piv info > "$DEVICE_DIR/piv/piv_info.txt" 2>/dev/null

# Export certificates from all slots
for slot in 9a 9c 9d 9e 82 83 84 85 86 87 88 89 8a 8b 8c 8d 8e 8f 90 91 92 93 94 95; do
  # Try to export certificate
  if ykman piv certificates export $slot "$DEVICE_DIR/piv/slot_${slot}_cert.pem" 2>/dev/null; then
    echo "✅ Exported PIV certificate from slot $slot"

    # Export certificate details
    openssl x509 -in "$DEVICE_DIR/piv/slot_${slot}_cert.pem" -noout -text > "$DEVICE_DIR/piv/slot_${slot}_cert_details.txt" 2>/dev/null

    # Export certificate in DER format
    openssl x509 -in "$DEVICE_DIR/piv/slot_${slot}_cert.pem" -outform DER -out "$DEVICE_DIR/piv/slot_${slot}_cert.der" 2>/dev/null

    # Export public key from certificate
    openssl x509 -in "$DEVICE_DIR/piv/slot_${slot}_cert.pem" -pubkey -noout > "$DEVICE_DIR/piv/slot_${slot}_pubkey.pem" 2>/dev/null
  fi
done

# Document PIV slot usage
cat > "$DEVICE_DIR/piv/SLOTS.md" <<'EOF'
# PIV Slot Reference

## Standard Slots
- **9a**: PIV Authentication (SSH, login)
- **9c**: Digital Signature (email, documents)
- **9d**: Key Management (encryption)
- **9e**: Card Authentication (physical access)

## Retired Key Management Slots
- **82-95**: Retired certificates (20 slots)

**Note**: Private keys remain on YubiKey and cannot be exported.
EOF
```

### 7. Export OATH Credentials Metadata

**IMPORTANT**: This exports credential **names and parameters only**, NOT the shared secrets.

```bash
# Create OATH directory
mkdir -p "$DEVICE_DIR/oath"

# Get OATH info
!ykman oath info > "$DEVICE_DIR/oath/oath_info.txt" 2>/dev/null

# List OATH credentials (names only, no secrets)
!ykman oath accounts list > "$DEVICE_DIR/oath/oath_accounts.txt" 2>/dev/null

# Count credentials
OATH_COUNT=$(ykman oath accounts list 2>/dev/null | wc -l)
echo "✅ Documented $OATH_COUNT OATH credentials (names only)"

# Create OATH recovery instructions
cat > "$DEVICE_DIR/oath/RECOVERY_INSTRUCTIONS.md" <<'EOF'
# OATH Credential Recovery

## What Was Backed Up
- **Credential names**: Account identifiers (e.g., "GitHub:user@example.com")
- **Credential types**: TOTP vs HOTP
- **Metadata**: Creation dates, usage counts

## What Was NOT Backed Up
- **Shared secrets**: The cryptographic secrets remain on the YubiKey only
- **Current TOTP codes**: Time-based codes are not stored

## Recovery Process
To restore OATH credentials on a new YubiKey:
1. Contact each service provider (GitHub, AWS, etc.)
2. Request 2FA reset or re-enrollment
3. Scan new QR code or enter new secret
4. Verify new credential works before disabling old YubiKey

## Prevention for Future
Consider using a password manager with 2FA backup codes:
- Store backup codes in 1Password, Bitwarden, etc.
- Keep backup codes offline in secure location
- Use multiple YubiKeys (primary + backup) for same accounts
EOF
```

### 8. Export FIDO2 Credential Metadata

```bash
# Create FIDO2 directory
mkdir -p "$DEVICE_DIR/fido2"

# Get FIDO2 info
!ykman fido info > "$DEVICE_DIR/fido2/fido2_info.txt" 2>/dev/null

# List FIDO2 credentials (requires PIN)
# Note: This is read-only but requires PIN for privacy protection
echo "⚠️  FIDO2 credential listing requires PIN"
echo "This is optional - you can skip if PIN is not available"

# Document FIDO2 recovery process
cat > "$DEVICE_DIR/fido2/RECOVERY_INSTRUCTIONS.md" <<'EOF'
# FIDO2 Credential Recovery

## What Was Backed Up
- **Device capability**: FIDO2 feature enabled/disabled
- **PIN status**: Whether PIN is configured
- **Credential count**: Number of resident credentials

## What Was NOT Backed Up
- **Private keys**: FIDO2 credentials are hardware-bound and non-exportable
- **Credential details**: For privacy, credential RPs are not exported without PIN

## Recovery Process
FIDO2 credentials cannot be transferred to a new YubiKey. For each service:
1. Log in using alternative authentication method
2. Remove old YubiKey from account security settings
3. Register new YubiKey as security key
4. Verify new key works before removing old one

## Best Practices
- Always register 2+ security keys per account (primary + backup)
- Store backup key in secure location separate from primary
- Document which services use FIDO2 authentication
- Keep account recovery codes in secure password manager
EOF
```

### 9. Create Configuration Snapshot

```bash
# Create comprehensive configuration document
cat > "$DEVICE_DIR/CONFIGURATION.md" <<EOF
# YubiKey Configuration Snapshot

**Backup Date**: $(date -u +"%Y-%m-%d %H:%M:%S UTC")
**Device Serial**: $SERIAL

---

## Device Information

$(cat "$DEVICE_DIR/device_info.txt")

---

## Application Status

### PIV
$(ykman piv info 2>/dev/null || echo "PIV not enabled or not configured")

### OpenPGP
$(ykman openpgp info 2>/dev/null || echo "OpenPGP not enabled or not configured")

### FIDO2
$(ykman fido info 2>/dev/null || echo "FIDO2 not enabled")

### OATH
$(ykman oath info 2>/dev/null || echo "OATH not enabled")

---

## PIN Retry Counters

$(ykman piv info 2>/dev/null | grep -i "tries remaining" || echo "No PIN retry information available")

---

## Touch Policies

$(ykman piv info 2>/dev/null | grep -i "touch" || echo "No touch policy information available")

---

## Notes

This backup contains **public information only**. Private keys remain on the YubiKey
hardware and cannot be exported by design. This is a security feature.

To restore configuration on a new YubiKey, you must:
1. Generate new key pairs on the new device
2. Import the backed-up certificates (if still valid)
3. Re-register with services for FIDO2/OATH
4. Configure the same policies (PIN, touch, etc.)

EOF
```

### 10. Generate Backup Archive

```bash
# Create tarball of backup
cd "$BACKUP_DIR"
tar -czf "yubikey-${SERIAL}-backup-$(date +%Y%m%d_%H%M%S).tar.gz" "yubikey-$SERIAL/"

# Generate checksums
sha256sum "yubikey-${SERIAL}-backup-$(date +%Y%m%d_%H%M%S).tar.gz" > "yubikey-${SERIAL}-backup-$(date +%Y%m%d_%H%M%S).tar.gz.sha256"

echo "✅ Created backup archive with SHA-256 checksum"
```

### 11. Verify Backup Integrity

```bash
# Verify archive integrity
tar -tzf "yubikey-${SERIAL}-backup-$(date +%Y%m%d_%H%M%S).tar.gz" > /dev/null 2>&1
if [ $? -eq 0 ]; then
  echo "✅ Backup archive integrity verified"
else
  echo "❌ Backup archive is corrupted!"
  exit 1
fi

# Verify checksums
sha256sum -c "yubikey-${SERIAL}-backup-$(date +%Y%m%d_%H%M%S).tar.gz.sha256"

# List archive contents
echo ""
echo "Backup Contents:"
tar -tzf "yubikey-${SERIAL}-backup-$(date +%Y%m%d_%H%M%S).tar.gz" | head -20
```

### 12. Generate Backup Report

Create **BACKUP_REPORT.md**:

```markdown
# YubiKey Backup Report

**Backup Date**: [ISO 8601 timestamp]
**Generated By**: YubiKey Backup Tool v1.0
**Devices Backed Up**: [N]
**Backup Location**: [Path]

---

## Executive Summary

| Metric | Value |
|--------|-------|
| Devices Backed Up | [N] |
| GPG Keys Exported | [N] |
| PIV Certificates Exported | [N] |
| OATH Credentials Documented | [N] |
| Backup Size | [X] MB |
| Archive Integrity | ✅ Verified |

**Backup Status**: ✅ SUCCESS / ⚠️ PARTIAL / ❌ FAILED

---

## Device #1: Serial [XXXXXX]

**Basic Information**:
- Model: YubiKey 5 NFC
- Firmware: 5.7.1
- Form Factor: USB-A with NFC
- Classification: 🧪 Test Device / ⚠️ Production Device

**Backup Status**: ✅ COMPLETE / ⚠️ PARTIAL / ❌ FAILED

### Exported Items

#### GPG/OpenPGP
- ✅ Public key exported (ID: [KEY_ID])
- ✅ Key details documented
- ✅ Minimal key variant created
- ✅ Card status saved

**Files**:
- `gpg_public_key.asc` (ASCII-armored)
- `gpg_public_key.gpg` (binary)
- `gpg_public_key_minimal.asc` (minimal)
- `gpg_key_details.txt` (metadata)
- `gpg_card_status.txt` (card info)

#### SSH
- ✅ SSH public key exported
- ⚠️ No SSH key found (no authentication subkey)

**Files**:
- `ssh_public_key.pub` (OpenSSH format)

#### PIV Certificates
- ✅ Slot 9a (Authentication): CN=John Doe
- ✅ Slot 9c (Signature): CN=John Doe
- ⚪ Slot 9d (Encryption): Empty
- ⚪ Slot 9e (Card Auth): Empty

**Files**:
- `piv/slot_9a_cert.pem` (PEM format)
- `piv/slot_9a_cert.der` (DER format)
- `piv/slot_9a_cert_details.txt` (X.509 details)
- `piv/slot_9a_pubkey.pem` (public key only)
- [Repeat for each slot]

#### OATH Credentials
- ✅ [N] credentials documented (names only)
- ⚠️ Secrets NOT backed up (remain on YubiKey)

**Files**:
- `oath/oath_accounts.txt` (credential names)
- `oath/oath_info.txt` (OATH applet info)
- `oath/RECOVERY_INSTRUCTIONS.md` (recovery guide)

#### FIDO2 Credentials
- ✅ Device capability documented
- ⚠️ Credentials NOT exportable (hardware-bound)

**Files**:
- `fido2/fido2_info.txt` (FIDO2 applet info)
- `fido2/RECOVERY_INSTRUCTIONS.md` (recovery guide)

#### Configuration
- ✅ Device information saved
- ✅ Application status documented
- ✅ PIN retry counters recorded
- ✅ Touch policies documented

**Files**:
- `device_info.txt` (human-readable)
- `device_info.json` (machine-readable)
- `CONFIGURATION.md` (comprehensive snapshot)

---

## Backup Archive

**Archive File**: `yubikey-[SERIAL]-backup-[TIMESTAMP].tar.gz`
**Archive Size**: [X] MB
**Compression**: gzip
**Checksum (SHA-256)**: [hash]
**Integrity**: ✅ Verified

**Archive Contents**:
```
yubikey-[SERIAL]/
├── device_info.txt
├── device_info.json
├── CONFIGURATION.md
├── gpg_public_key.asc
├── gpg_key_details.txt
├── ssh_public_key.pub
├── piv/
│   ├── piv_info.txt
│   ├── SLOTS.md
│   ├── slot_9a_cert.pem
│   ├── slot_9a_cert_details.txt
│   └── [...]
├── oath/
│   ├── oath_accounts.txt
│   ├── oath_info.txt
│   └── RECOVERY_INSTRUCTIONS.md
└── fido2/
    ├── fido2_info.txt
    └── RECOVERY_INSTRUCTIONS.md
```

---

## Recovery Instructions

### Immediate Recovery (Same YubiKey)
If this YubiKey is lost but you have a backup YubiKey with the same keys:
1. Use the backup YubiKey immediately
2. No recovery needed - keys are duplicated

### Full Recovery (New YubiKey Required)
If all YubiKeys are lost and you need a new device:

#### 1. GPG Key Recovery
**If you have the private key backed up elsewhere** (not on YubiKey):
```bash
# Import backed-up private key
gpg --import private_key_backup.asc

# Transfer to new YubiKey
gpg --edit-key [KEY_ID]
> keytocard
```

**If private key was only on YubiKey** (common case):
- Private key is **permanently lost** (this is by design)
- Generate new GPG key on new YubiKey
- Distribute new public key to contacts/keyservers
- Update GitHub/GitLab with new key

#### 2. SSH Key Recovery
**If GPG authentication subkey is recovered**:
```bash
# Extract SSH key from recovered GPG key
gpg --export-ssh-key [KEY_ID] > new_ssh_key.pub

# Add to SSH agent
ssh-add -L
```

**If GPG key is lost**:
- Generate new SSH key on new YubiKey
- Update `~/.ssh/authorized_keys` on servers
- Update GitHub/GitLab SSH keys

#### 3. PIV Certificate Recovery
**If certificates are still valid**:
```bash
# Generate new key pair on new YubiKey
ykman piv keys generate 9a new_public_key.pem

# Import backed-up certificate (if key pair matches)
ykman piv certificates import 9a backed_up_cert.pem
```

**If certificates don't match new keys**:
- Generate new key pairs
- Request new certificates from CA
- Import new certificates

#### 4. OATH Credential Recovery
**Required**: Contact each service provider
- Request 2FA reset/re-enrollment
- Scan new QR code or enter new secret
- Verify credential works
- Store backup codes in secure location

**Services to Update**:
- [List documented OATH accounts from oath_accounts.txt]

#### 5. FIDO2 Credential Recovery
**Required**: Re-register with each service
- Log in with alternative method
- Remove old security key from account
- Register new YubiKey
- Verify new key works

**Services to Update**:
- [List known FIDO2-enabled services]

---

## Storage Recommendations

### Backup Archive Storage
**Primary Copy**:
- Encrypted external drive (BitLocker, LUKS, FileVault)
- Stored in secure location (safe, bank vault)

**Secondary Copy** (optional):
- Encrypted cloud storage (AWS S3, Backblaze B2)
- Use client-side encryption (Cryptomator, rclone crypt)
- Different location than primary

**Tertiary Copy** (for critical deployments):
- Offline backup (DVD, USB in sealed envelope)
- Geographic separation from primary/secondary

### Security Measures
- ✅ Encrypt backup archives with strong passphrase
- ✅ Store encryption passphrase separately
- ✅ Test recovery process periodically
- ✅ Update backups after configuration changes
- ✅ Document backup location(s) in secure location
- ✅ Set calendar reminder to verify backup integrity (quarterly)

### DO NOT Store
- ❌ Backups on same computer as YubiKey
- ❌ Unencrypted backups on cloud storage
- ❌ PINs, PUKs, or management keys in backup
- ❌ Private keys (impossible anyway - they can't leave YubiKey)

---

## Restoration Testing

**Last Tested**: [Date or "Never tested"]
**Test Type**: [Partial / Full / Never tested]
**Test Result**: [Success / Partial Success / Failed / Not tested]

### Recommended Test Schedule
- **After initial backup**: Verify archive extraction and file readability
- **Quarterly**: Test public key import on test system
- **Annually**: Full recovery dry run on spare YubiKey

### Test Checklist
- [ ] Extract backup archive
- [ ] Verify checksum
- [ ] Import GPG public key on test system
- [ ] Import SSH public key on test system
- [ ] Read PIV certificate details with OpenSSL
- [ ] Review OATH credential list
- [ ] Verify configuration documentation completeness

---

## Backup Metadata

**Backup Tool Version**: 1.0
**YubiKey Manager Version**: [ykman version]
**GPG Version**: [gpg version]
**Operating System**: [OS and version]
**Backup Duration**: [X] seconds
**User**: [username]
**Hostname**: [hostname]

---

## Next Steps

### Immediate (Today)
- [ ] Verify backup archive integrity
- [ ] Store backup in secure location
- [ ] Document backup location in password manager
- [ ] Test backup extraction on different computer

### Short-term (This Week)
- [ ] Create secondary backup copy (different location)
- [ ] Encrypt backup archive with strong passphrase
- [ ] Test importing GPG public key
- [ ] Review OATH credentials list and document services

### Long-term (This Month)
- [ ] Schedule quarterly backup updates
- [ ] Set calendar reminder for backup integrity verification
- [ ] Document recovery procedures in team wiki
- [ ] Consider backup YubiKey with duplicated keys

---

**Report Generated**: [Timestamp]
**Report Valid Until**: [Next backup recommended date]
**Next Backup Due**: [Date, based on change frequency]
```

### 13. Display Backup Summary

```
╔════════════════════════════════════════════════════╗
║           YUBIKEY BACKUP COMPLETE                  ║
╚════════════════════════════════════════════════════╝

DEVICES BACKED UP: [N]

BACKUP LOCATION: [Path]

EXPORTED ITEMS:
  ✅ GPG Public Keys: [N]
  ✅ SSH Public Keys: [N]
  ✅ PIV Certificates: [N]
  ✅ OATH Credentials (metadata): [N]
  ✅ FIDO2 Info (metadata): [N]
  ✅ Configuration Snapshots: [N]

ARCHIVE STATUS:
  ✅ Archive created: yubikey-[SERIAL]-backup-[TIMESTAMP].tar.gz
  ✅ Size: [X] MB
  ✅ Integrity verified (SHA-256)
  ✅ Compression: gzip

IMPORTANT REMINDERS:
  ⚠️  Private keys remain on YubiKey (cannot be exported)
  ⚠️  OATH secrets remain on YubiKey (cannot be exported)
  ⚠️  FIDO2 credentials remain on YubiKey (cannot be exported)
  ✅ All public information and metadata backed up successfully

SECURITY RECOMMENDATIONS:
  » Encrypt backup archive before storage
  » Store in secure location (encrypted drive, safe)
  » Create secondary backup in different location
  » Test recovery process periodically
  » Update backup after configuration changes

NEXT ACTIONS:
  » Store backup archive securely
  » Document backup location in password manager
  » Set calendar reminder for next backup (3 months)
  » Test restoration on spare YubiKey (recommended)

Full Report: BACKUP_REPORT.md

This backup enables recovery of public keys and configuration.
For full recovery, consider using backup YubiKeys with duplicated keys.
```

## Key Features

- **Comprehensive Coverage**: All applications (PIV, FIDO2, OATH, OpenPGP)
- **Public Keys Only**: Never attempts to export private keys (impossible by design)
- **Archive Creation**: Timestamped, compressed archives with checksums
- **Integrity Verification**: SHA-256 checksums and archive testing
- **Recovery Guidance**: Detailed instructions for each credential type
- **Metadata Preservation**: Configuration snapshots for future reference
- **Security-First**: No sensitive data in backups, encryption recommended
- **Automation-Ready**: Can be integrated into backup schedules

## When to Use /yubikey-backup

### Regular Backup Schedule
- **After initial setup**: Immediately after configuring new YubiKey
- **After changes**: Any time credentials or configuration changes
- **Periodic**: Monthly or quarterly for unchanged configurations
- **Before risky operations**: Before firmware updates, factory resets

### Specific Scenarios
- Planning to travel with YubiKey (backup before trip)
- Adding new credentials or certificates
- Rotating certificates (backup both old and new)
- Device lifecycle management (backup before decommissioning)
- Compliance requirements (audit trail, disaster recovery)

### Integration with Workflows

```bash
# After yubikey-setup
/yubikey-setup-wizard
/yubikey-backup  # Backup new configuration

# Regular maintenance
/yubikey-health-check  # Check device health
/yubikey-backup        # Backup if changes detected

# Before risky operations
/yubikey-backup        # Backup current state
[perform risky operation]
/yubikey-health-check  # Verify health after operation

# Disaster recovery preparation
/yubikey-backup        # Create backup
[test restoration on spare YubiKey]
[encrypt and store backup securely]
```

## Backup Schedule Recommendations

### Production Devices
- **Initial**: Immediately after setup
- **Regular**: Monthly backups
- **Change-driven**: After any configuration change
- **Pre-travel**: Before taking YubiKey out of secure location

### Development/Test Devices
- **Initial**: After setup
- **Regular**: Quarterly backups
- **Change-driven**: After significant changes
- **Pre-reset**: Before factory resets or testing

### Personal Devices
- **Initial**: After setup
- **Regular**: Quarterly backups
- **Change-driven**: After adding/removing credentials
- **Annual**: Full backup with recovery test

## Recovery Scenarios

### Scenario 1: YubiKey Lost (Have Backup YubiKey)
**Best case**: Backup YubiKey has same keys
- ✅ Switch to backup YubiKey immediately
- ✅ No recovery needed (keys are duplicated)
- ⚠️ Order replacement YubiKey
- ⚠️ Replicate keys to new device when it arrives

### Scenario 2: YubiKey Lost (No Backup YubiKey)
**Moderate case**: Have public key backups
- ⚠️ Private keys are permanently lost
- ✅ Generate new keys on new YubiKey
- ⚠️ Distribute new public keys to contacts
- ⚠️ Update services (GitHub, SSH, etc.)
- ⚠️ Re-register OATH/FIDO2 credentials

### Scenario 3: YubiKey Lost (No Backups at All)
**Worst case**: No backups, no backup YubiKey
- ❌ All keys permanently lost
- ❌ Must re-register with all services
- ❌ Lose access to encrypted data (if no key backup)
- ❌ Maximum recovery time and effort

**Prevention**: Always maintain backups and backup YubiKey!

## Best Practices

1. **Backup Immediately**: Don't wait - backup right after setup
2. **Multiple Copies**: At least 2 copies in different locations
3. **Encrypt Archives**: Always encrypt before storing
4. **Test Recovery**: Periodically test restoration process
5. **Update Regularly**: Backup after any configuration change
6. **Document Location**: Record backup location in password manager
7. **Separate Storage**: Don't store backup on same computer as YubiKey
8. **Backup YubiKey**: Best protection is a second YubiKey with duplicated keys

## Troubleshooting

### GPG Export Fails
**Problem**: `gpg --export` returns no data
**Solutions**:
1. Check if GPG knows about the key: `gpg --card-status`
2. Import public key from YubiKey: `gpg --card-edit` then `fetch`
3. Verify key is on YubiKey: `ykman openpgp info`

### PIV Certificate Export Fails
**Problem**: `ykman piv certificates export` fails
**Solutions**:
1. Check if slot has certificate: `ykman piv info`
2. Verify slot number is correct (9a, 9c, 9d, 9e)
3. Check YubiKey connection: `ykman list`

### Archive Checksum Mismatch
**Problem**: SHA-256 verification fails
**Solutions**:
1. Archive may be corrupted during creation
2. Recreate backup from scratch
3. Verify disk is not failing (run disk check)
4. Try different backup location

### Backup Size Too Large
**Problem**: Backup archive exceeds expected size
**Solutions**:
1. Check for duplicate files
2. Exclude unnecessary files
3. Use higher compression: `gzip -9`
4. Split into multiple archives if needed

## Security Considerations

- **Read-Only Operations**: Backup only reads from YubiKey, never writes
- **No Private Keys**: Private keys cannot and will not be exported
- **File Permissions**: Set restrictive permissions on backup archives (chmod 600)
- **Encryption**: Encrypt archives before storing on cloud or removable media
- **Secure Deletion**: Use secure deletion for old backups (shred, srm)
- **Passphrase Protection**: Use strong, unique passphrases for archive encryption
- **Metadata Privacy**: Archive contains serial numbers and key IDs (consider sensitivity)

## Compliance and Audit

### Audit Trail
- Backup report includes timestamp, user, hostname
- SHA-256 checksums provide integrity verification
- Configuration snapshots enable point-in-time recovery
- Backup history provides compliance evidence

### Regulatory Considerations
- **GDPR**: Backups may contain personal data (consider data retention)
- **PCI-DSS**: Certificate backups may be required for key management
- **HIPAA**: Backup encryption required for protected health information
- **SOC 2**: Backup and recovery procedures are control evidence

---

**Command Version**: 1.0
**Last Updated**: 2025-11-21
**Maintained By**: YubiKey Tools Team
