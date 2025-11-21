---
description: "Verify YubiKey backup integrity, completeness, and restoration capability without requiring actual YubiKey hardware"
allowed-tools: ["Read", "Bash(ls:*)", "Bash(find:*)", "Bash(gpg:*)", "Bash(file:*)"]
author: "YubiKey Tools Backup Team"
version: "1.0"
---

# YubiKey Backup Verification

## Purpose
Verify the integrity and completeness of YubiKey backups to ensure successful restoration if needed. This command performs read-only verification without modifying any backup files or requiring YubiKey hardware.

## Backup Verification Steps

### 1. Locate Backup Directory

```bash
# Find backup directories
!find . -type d -name "*backup*" -o -name "*yubikey-backup*" 2>/dev/null

# Check common backup locations
!ls -la ~/yubikey-backups/ 2>/dev/null || echo "Not found in ~/yubikey-backups/"
!ls -la ./backups/ 2>/dev/null || echo "Not found in ./backups/"
!ls -la ./yubikey-backups/ 2>/dev/null || echo "Not found in ./yubikey-backups/"
```

### 2. Analyze Backup Structure

```bash
# List all backups with timestamps
!find ./backups -type f -name "*.tar.gz" -o -name "*.gpg" | sort

# Get backup metadata
!ls -lh ./backups/*.tar.gz 2>/dev/null
!ls -lh ./backups/*.gpg 2>/dev/null
```

### 3. Verify Backup Completeness

Check for required files in each backup:

```bash
# Expected backup contents (from yubikey-setup.sh)
# For each backup archive:

# GPG key files
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz | grep "master-key.gpg"
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz | grep "signing-subkey.gpg"
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz | grep "encryption-subkey.gpg"
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz | grep "authentication-subkey.gpg"

# Public keys
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz | grep "public-key.asc"
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz | grep "public-ssh-key.pub"

# Configuration
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz | grep "backup-info.txt"
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz | grep "yubikey-info.txt"

# Revocation certificate
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz | grep "revocation-certificate.asc"
```

### 4. Validate GPG Key Files

```bash
# Extract and verify GPG key structure (without importing)
!gpg --list-packets backup/master-key.gpg 2>/dev/null | head -20
!gpg --list-packets backup/public-key.asc 2>/dev/null | head -20

# Check key attributes
!gpg --with-colons --import-options show-only --import backup/public-key.asc 2>/dev/null
```

### 5. Check Backup Metadata

```bash
# Read backup info file
@Read: backups/latest/backup-info.txt

# Verify it contains:
# - Backup date and time
# - YubiKey serial number
# - Firmware version
# - Key fingerprints
# - Backup creator
```

### 6. Verify File Integrity

```bash
# Check for corruption (file command)
!file backup/*.gpg
!file backup/*.asc
!file backup/*.tar.gz

# Verify archive integrity
!tar -tzf backup-YYYYMMDD-HHMMSS.tar.gz > /dev/null 2>&1 && echo "✅ Archive integrity OK" || echo "❌ Archive corrupted"

# Check file sizes (should not be 0 bytes)
!find backup/ -type f -size 0 2>/dev/null || echo "✅ No zero-byte files"
```

### 7. Validate Encryption (if applicable)

```bash
# Check for encrypted backups
!find backup/ -name "*.gpg" -type f

# Verify GPG encryption headers
!head -c 100 backup/encrypted-backup.tar.gz.gpg | file -
```

### 8. Test Restoration Dry-Run

```bash
# Simulate restoration without importing keys

# 1. Extract backup archive to temp directory
!mkdir -p /tmp/backup-verify-$$
!tar -xzf backup-YYYYMMDD-HHMMSS.tar.gz -C /tmp/backup-verify-$$

# 2. Verify extracted contents
!ls -la /tmp/backup-verify-$$/

# 3. Check GPG key validity
!gpg --dry-run --import /tmp/backup-verify-$$/public-key.asc 2>&1 | grep -i "key\|error"

# 4. Cleanup
!rm -rf /tmp/backup-verify-$$
```

### 9. Generate Backup Verification Report

Create **BACKUP_VERIFICATION_REPORT.md**:

```markdown
# YubiKey Backup Verification Report

**Verification Date**: [ISO 8601 timestamp]
**Backup Location**: [path]
**Verifier**: Backup Verification Tool v1.0

---

## Executive Summary

| Aspect | Status | Details |
|--------|--------|---------|
| Backups Found | ✅ | [N] backup archives |
| Latest Backup | ✅ | [YYYY-MM-DD HH:MM:SS] |
| Backup Completeness | [✅/⚠️/❌] | [XX%] complete |
| File Integrity | [✅/⚠️/❌] | [N/N] files valid |
| GPG Key Validity | [✅/⚠️/❌] | Valid structure |
| Restoration Test | [✅/⚠️/❌] | Dry-run successful |

**Overall Backup Status**: ✅ GOOD / ⚠️  REVIEW NEEDED / ❌ CRITICAL ISSUES

---

## Backup Inventory

### Available Backups
| Backup Date | Size | Format | Completeness | Status |
|-------------|------|--------|--------------|--------|
| 2025-11-20 10:30 | 2.4 MB | tar.gz | 100% | ✅ Complete |
| 2025-11-13 14:15 | 2.3 MB | tar.gz | 100% | ✅ Complete |
| 2025-11-06 09:45 | 2.3 MB | tar.gz | 95% | ⚠️  Missing SSH key |
| 2025-10-30 16:20 | 2.2 MB | tar.gz | 100% | ✅ Complete |

**Total Backups**: [N]
**Oldest Backup**: [date]
**Newest Backup**: [date]

---

## Latest Backup Analysis

### Backup: 2025-11-20-103045

**Metadata**:
- Backup Date: 2025-11-20 10:30:45 UTC
- YubiKey Serial: [XXXXXX]
- Firmware Version: 5.7.1
- Created By: yubikey-setup.sh v1.1.0
- Archive Size: 2.4 MB
- Archive Format: tar.gz (unencrypted)

### File Inventory
```
backup-2025-11-20-103045/
├── master-key.gpg                    ✅ 1.2 MB
├── signing-subkey.gpg                ✅ 512 KB
├── encryption-subkey.gpg             ✅ 512 KB
├── authentication-subkey.gpg         ✅ 512 KB
├── public-key.asc                    ✅ 8 KB
├── public-ssh-key.pub                ✅ 2 KB
├── revocation-certificate.asc        ✅ 4 KB
├── backup-info.txt                   ✅ 1 KB
└── yubikey-info.txt                  ✅ 2 KB
```

**Completeness**: ✅ 9/9 expected files present (100%)

### GPG Key Validation

#### Master Key
```
Type: RSA 4096-bit
Created: 2025-11-20
Expires: Never (set expiration recommended!)
Capabilities: Certify, Sign
Key ID: [XXXX XXXX XXXX XXXX]
Fingerprint: [XXXX XXXX XXXX XXXX XXXX  XXXX XXXX XXXX XXXX XXXX]
```

**Status**: ✅ Valid structure

#### Subkeys
- **Signing**: RSA 4096, expires 2026-11-20 ✅
- **Encryption**: RSA 4096, expires 2026-11-20 ✅
- **Authentication**: RSA 4096, expires 2026-11-20 ✅

**Status**: ✅ All subkeys valid

### Public Key Content
```
# From public-key.asc
✅ User ID present: [Name <email@example.com>]
✅ Self-signature present
✅ Subkey binding signatures present
✅ No revocation signatures (good)
```

### SSH Public Key
```
# From public-ssh-key.pub
ssh-rsa AAAAB3Nza... [email]
```

**Status**: ✅ Valid SSH public key format

### Revocation Certificate
```
# From revocation-certificate.asc
✅ Revocation signature present
✅ Reason code: No reason specified (pre-generated)
✅ Self-signed by master key
```

**Status**: ✅ Valid revocation certificate

### Backup Metadata
```
# From backup-info.txt
Backup created: 2025-11-20 10:30:45 UTC
YubiKey serial: [XXXXXX]
Firmware: 5.7.1
Master key ID: [key ID]
Signing subkey ID: [key ID]
Encryption subkey ID: [key ID]
Authentication subkey ID: [key ID]
SSH public key: yes
Created by: yubikey-setup.sh v1.1.0
```

**Status**: ✅ Complete metadata

---

## File Integrity Checks

### Archive Integrity
```bash
$ tar -tzf backup-2025-11-20-103045.tar.gz > /dev/null
```
**Result**: ✅ Archive intact, no corruption detected

### Individual File Integrity
| File | Size | Type | Status |
|------|------|------|--------|
| master-key.gpg | 1.2 MB | OpenPGP Secret Key | ✅ Valid |
| signing-subkey.gpg | 512 KB | OpenPGP Secret Subkey | ✅ Valid |
| encryption-subkey.gpg | 512 KB | OpenPGP Secret Subkey | ✅ Valid |
| authentication-subkey.gpg | 512 KB | OpenPGP Secret Subkey | ✅ Valid |
| public-key.asc | 8 KB | OpenPGP Public Key | ✅ Valid |
| public-ssh-key.pub | 2 KB | SSH Public Key | ✅ Valid |
| revocation-certificate.asc | 4 KB | OpenPGP Revocation | ✅ Valid |

### Zero-Byte Files
**Result**: ✅ No zero-byte files detected

---

## Restoration Dry-Run Test

### Test Procedure
1. Extract backup archive to temporary directory
2. Verify file structure
3. Validate GPG key format
4. Check key relationships
5. Simulate import (--dry-run)

### Test Results
```
✅ Archive extraction successful
✅ All expected files present
✅ GPG key structure valid
✅ Master/subkey relationships correct
✅ Dry-run import successful
```

**Restoration Viability**: ✅ Backup can be successfully restored

---

## Historical Backup Comparison

### Backup Consistency
| Backup Date | Files | Size | Master Key ID | Status |
|-------------|-------|------|---------------|--------|
| 2025-11-20 | 9 | 2.4 MB | [ID] | ✅ Current |
| 2025-11-13 | 9 | 2.3 MB | [ID] | ✅ Same keys |
| 2025-11-06 | 8 | 2.3 MB | [ID] | ⚠️  Missing SSH |
| 2025-10-30 | 9 | 2.2 MB | [ID] | ✅ Same keys |

**Consistency**: ✅ Master key ID consistent across backups

---

## Security Review

### Encryption Status
- **Archive Encryption**: ⚠️  NOT ENCRYPTED (tar.gz only)
- **Private Key Encryption**: ✅ GPG keys encrypted with passphrase
- **Backup Location**: [path]
- **Access Permissions**: [permissions]

**Recommendation**: Consider encrypting backup archives with GPG

### Access Control
```bash
$ ls -l backup-2025-11-20-103045.tar.gz
-rw------- 1 user user 2.4M Nov 20 10:30 backup-2025-11-20-103045.tar.gz
```

**Status**: ✅ Proper permissions (600 - owner read/write only)

### Storage Location Security
- **Location Type**: [Local/Cloud/External Drive]
- **Physical Security**: [Assessment]
- **Offsite Backup**: [Yes/No]
- **Redundancy**: [N] backup copies

**Recommendation**: Maintain at least 2 offsite backup copies

---

## Issues and Warnings

### Critical Issues ❌
[None found / List issues]

### Warnings ⚠️
1. **Archive Not Encrypted**: Backup archives not encrypted at rest
   - Risk: Compromise of backup storage exposes keys
   - Recommendation: Encrypt with `gpg --encrypt backup.tar.gz`

2. **Master Key No Expiration**: Master key set to never expire
   - Risk: Lost key remains valid indefinitely
   - Recommendation: Set expiration date (e.g., 5 years)

3. **Missing Backup 2025-11-13**: Gap in backup schedule
   - Risk: Potential data loss if key compromised
   - Recommendation: Maintain weekly backup schedule

### Informational ℹ️
1. **Backup Age**: Latest backup is [N] days old
2. **Backup Frequency**: Approximately [N] days between backups
3. **Backup Size Growth**: +5% since first backup (normal)

---

## Backup Best Practices Compliance

| Practice | Status | Notes |
|----------|--------|-------|
| Regular backups | [✅/⚠️/❌] | [frequency] |
| Multiple copies | [✅/⚠️/❌] | [N] copies |
| Offsite storage | [✅/⚠️/❌] | [location] |
| Encrypted backups | [✅/⚠️/❌] | [encryption status] |
| Verified integrity | ✅ | This report |
| Tested restoration | ✅ | Dry-run successful |
| Documented process | ✅ | yubikey-setup.sh |
| Access controls | ✅ | Proper permissions |

**Best Practices Score**: [X/8] ([XX%])

---

## Recommendations

### Immediate Actions
1. **Encrypt backup archives**: Use GPG to encrypt tar.gz files
   ```bash
   gpg --symmetric --cipher-algo AES256 backup.tar.gz
   ```

2. **Create offsite copy**: Store encrypted backup in secure location

3. **Set key expiration**: Update master key with expiration date

### Short-Term (This Week)
1. **Establish backup schedule**: Weekly automated backups
2. **Create backup documentation**: Recovery procedures
3. **Test full restoration**: Restore to test YubiKey

### Long-Term (This Month)
1. **Implement 3-2-1 backup rule**:
   - 3 copies of data
   - 2 different storage types
   - 1 offsite backup
2. **Automate backup verification**: Schedule monthly verification
3. **Document key escrow**: Procedure for emergency access

---

## Recovery Procedure

### How to Restore from This Backup

1. **Extract backup archive**:
   ```bash
   tar -xzf backup-2025-11-20-103045.tar.gz
   cd backup-2025-11-20-103045/
   ```

2. **Import master key**:
   ```bash
   gpg --import master-key.gpg
   # Enter passphrase when prompted
   ```

3. **Import subkeys**:
   ```bash
   gpg --import signing-subkey.gpg
   gpg --import encryption-subkey.gpg
   gpg --import authentication-subkey.gpg
   ```

4. **Transfer to YubiKey**:
   ```bash
   gpg --edit-key [KEY-ID]
   gpg> keytocard
   # Select appropriate slot for each subkey
   ```

5. **Verify transfer**:
   ```bash
   gpg --card-status
   ```

### Estimated Recovery Time
- Manual restoration: ~30 minutes
- Using yubikey-setup.sh: ~10 minutes

---

## Next Steps

1. **Address warnings**: [List priority actions]
2. **Schedule next backup**: [Date]
3. **Schedule next verification**: [Date]
4. **Update backup documentation**: [Needed updates]
5. **Test full restoration**: [Schedule test]

---

**Report Generated**: [Timestamp]
**Next Verification**: [Recommended date]
**Report Valid Until**: [Expiration date]
**Verified By**: Backup Verification Tool
```

### 10. Display Verification Summary

```
╔════════════════════════════════════════════════════╗
║       BACKUP VERIFICATION COMPLETE                 ║
╚════════════════════════════════════════════════════╝

BACKUP STATUS: ✅ GOOD / ⚠️  REVIEW NEEDED / ❌ CRITICAL

BACKUPS FOUND: [N]
  Latest: 2025-11-20 10:30:45 ([N] days ago)
  Oldest: 2025-10-30 16:20:10 ([N] days ago)

LATEST BACKUP DETAILS:
  Completeness:   100% (9/9 files) ✅
  File Integrity: All files valid ✅
  Archive Size:   2.4 MB
  YubiKey Serial: [XXXXXX]
  Firmware:       5.7.1

KEY VALIDATION:
  Master Key:     RSA 4096 ✅
  Signing:        RSA 4096 ✅
  Encryption:     RSA 4096 ✅
  Authentication: RSA 4096 ✅
  SSH Public Key: Valid ✅
  Revocation Cert: Valid ✅

RESTORATION TEST:
  Dry-Run: ✅ PASSED
  Estimated Recovery Time: ~10 minutes

ISSUES:
  🔴 Critical:  [N]
  ⚠️  Warnings:  [N]
  ℹ️  Info:      [N]

WARNINGS:
  » Archive not encrypted (consider GPG encryption)
  » Master key has no expiration date
  » [Other warnings]

RECOMMENDATIONS:
  » Encrypt backup archives for added security
  » Create offsite backup copy
  » Test full restoration to YubiKey
  » Establish automated backup schedule

Full Report: BACKUP_VERIFICATION_REPORT.md

BACKUP VIABILITY: ✅ Backup can be successfully restored
```

## Key Features

- **Read-Only**: Never modifies backup files
- **Comprehensive**: Checks completeness, integrity, validity
- **No Hardware Needed**: Verifies without YubiKey
- **Restoration Testing**: Dry-run simulation
- **Security Assessment**: Encryption and access control review
- **Historical Analysis**: Compares multiple backups
- **Detailed Reporting**: BACKUP_VERIFICATION_REPORT.md
- **Recovery Guidance**: Step-by-step restoration procedure

## When to Use /yubikey-backup-verify

- After creating new backups
- Monthly backup verification
- Before disaster recovery
- After backup storage changes
- When testing recovery procedures
- For compliance audits
- Before YubiKey replacement
- As part of security reviews

## Best Practices

1. **Regular Verification**: Monthly or after each backup
2. **Test Restoration**: Annually test full restoration
3. **Multiple Copies**: Maintain 3+ backup copies
4. **Offsite Storage**: Keep backups in secure offsite location
5. **Encrypt Backups**: Use GPG encryption for archives
6. **Document Recovery**: Keep recovery procedures accessible
7. **Monitor Age**: Don't let backups get stale
8. **Verify Integrity**: Always verify after creating backups
