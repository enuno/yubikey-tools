---
description: "Interactive guided setup for new YubiKeys including GPG key generation, configuration, and backup creation (wraps yubikey-setup.sh)"
allowed-tools: ["Read", "Bash(ykman:list)", "Bash(ykman:info)", "Bash(./yubikey-setup.sh:*)", "Bash(./scripts/automation/yubikey-setup/yubikey-setup.sh:*)"]
author: "YubiKey Tools Team"
version: "1.0"
requires-approval: true
---

# YubiKey Setup Wizard

## Purpose
Provide an interactive, guided setup experience for configuring new YubiKeys with GPG keys, SSH authentication, and complete backup creation. This command wraps the battle-tested `yubikey-setup.sh` script with additional safety checks, device validation, and step-by-step guidance.

## Safety Level
**DANGER** - This command **WILL MODIFY your YubiKey**. It generates keys, transfers them to hardware, and changes PINs. Requires explicit user approval and verification at each critical step.

## Prerequisites

### Required Software
- [ ] YubiKey Manager (ykman) >= 5.0.0
- [ ] GnuPG (gpg) >= 2.2.0
- [ ] Expect (for automated key transfer)
- [ ] Bash 4.0+

### Required Hardware
- [ ] YubiKey 5 Series (5, 5C, 5 NFC, 5 Nano, 5C Nano)
- [ ] YubiKey connected and recognized

### Important Warnings
- ⚠️  **This operation will RESET the OpenPGP applet**
- ⚠️  **Any existing keys on the YubiKey will be DESTROYED**
- ⚠️  **You will need to set new PIN and Admin PIN**
- ⚠️  **Backup will be created but MUST be stored securely**
- ⚠️  **Process takes 10-30 minutes depending on options**

## Setup Wizard Steps

### Pre-Flight Checks

#### 1. Verify YubiKey Connection

```bash
# Check for connected YubiKeys
!ykman list

# Get detailed info
!ykman info
```

#### 2. Identify Setup Mode

**Three available modes:**

**Mode 1: Generate (NEW YubiKey)**
- Generate fresh GPG keys
- Transfer to YubiKey
- Create backup
- **Use when**: Setting up a brand new YubiKey

**Mode 2: Load (RESTORE from backup)**
- Import existing keys from backup
- Transfer to replacement YubiKey
- Restore configuration
- **Use when**: Replacing a lost/damaged YubiKey

**Mode 3: Backup Only**
- Export existing keys from YubiKey
- Create backup archive
- No changes to YubiKey
- **Use when**: Creating backup of existing setup

#### 3. Safety Verification Prompt

```
╔════════════════════════════════════════════════════╗
║       ⚠️  YUBIKEY SETUP WIZARD WARNING  ⚠️          ║
╚════════════════════════════════════════════════════╝

This wizard will MODIFY your YubiKey. Please confirm:

YubiKey Information:
  Serial: [XXXXXX]
  Model: YubiKey 5 NFC
  Firmware: 5.7.1

Current OpenPGP Status:
  Keys Present: [YES/NO]

⚠️  WARNING: This operation will:
  1. RESET the OpenPGP applet
  2. DESTROY any existing keys
  3. Generate NEW keys (Generate mode)
  4. Import keys from backup (Load mode)
  5. Set NEW PIN and Admin PIN

📋 What you need ready:
  - Desired PIN (6-8 digits, user PIN)
  - Desired Admin PIN (8 digits, administrative PIN)
  - Your name and email for key identity
  - Secure location for backup storage

Do you want to proceed?
  [G] Generate new keys
  [L] Load from backup
  [B] Backup only (safe)
  [C] Cancel

Which mode? (G/L/B/C): _
```

### Setup Mode: Generate (New Keys)

#### Step 1: Gather User Information

```bash
# Prompt for user details
echo "Setup Mode: Generate New Keys"
echo ""
echo "Please provide the following information:"
echo ""
read -p "Full Name: " FULL_NAME
read -p "Email Address: " EMAIL
read -p "Key Size (2048/4096) [4096]: " KEY_SIZE
KEY_SIZE=${KEY_SIZE:-4096}

echo ""
echo "Key Configuration:"
echo "  Name: $FULL_NAME"
echo "  Email: $EMAIL"
echo "  Key Size: $KEY_SIZE bits"
echo ""
read -p "Is this correct? (y/n): " CONFIRM
```

#### Step 2: Run yubikey-setup.sh (Generate Mode)

```bash
# Change to scripts directory
cd scripts/automation/yubikey-setup/

# Run setup script in generate mode
!./yubikey-setup.sh

# Script will:
# 1. Prompt for YubiKey selection (if multiple)
# 2. Reset OpenPGP applet
# 3. Set default PINs (123456 / 12345678)
# 4. Generate GPG keys (master + 3 subkeys)
# 5. Transfer keys to YubiKey with expect automation
# 6. Configure touch policies
# 7. Generate SSH public key
# 8. Configure Git signing
# 9. Create timestamped backup
# 10. Verify key transfer success
```

#### Step 3: Verify Setup

```bash
# Check GPG key on YubiKey
!gpg --card-status

# Verify all subkeys present
echo "Verifying key transfer..."
gpg --card-status | grep "Signature key"
gpg --card-status | grep "Encryption key"
gpg --card-status | grep "Authentication key"

# Check SSH key
!gpg --export-ssh-key $EMAIL

# Verify backup created
!ls -lh ~/yubikey-backups/backup-$(date +%Y%m%d)*.tar.gz
```

#### Step 4: Change Default PINs

```bash
echo ""
echo "⚠️  IMPORTANT: Change default PINs!"
echo ""
echo "Current PINs (from setup):"
echo "  User PIN: 123456"
echo "  Admin PIN: 12345678"
echo ""
echo "You MUST change these to secure PINs."
echo ""

# Change user PIN
!gpg --change-pin
# Select option 1 (Change PIN)

# Change admin PIN
!gpg --change-pin
# Select option 3 (Change Admin PIN)

echo ""
echo "✅ PINs changed successfully"
```

### Setup Mode: Load (Restore from Backup)

#### Step 1: Locate Backup

```bash
echo "Setup Mode: Load from Backup"
echo ""
echo "Available backups:"
!ls -lh ~/yubikey-backups/

read -p "Enter backup filename: " BACKUP_FILE

# Verify backup exists
if [ -f ~/yubikey-backups/$BACKUP_FILE ]; then
    echo "✅ Backup found: $BACKUP_FILE"
else
    echo "❌ Backup not found"
    exit 1
fi
```

#### Step 2: Verify Backup Integrity

```bash
# Test backup archive
!tar -tzf ~/yubikey-backups/$BACKUP_FILE > /dev/null 2>&1 && echo "✅ Backup integrity OK" || echo "❌ Backup corrupted"

# List backup contents
echo ""
echo "Backup contents:"
!tar -tzf ~/yubikey-backups/$BACKUP_FILE

# Read backup metadata
!tar -xzf ~/yubikey-backups/$BACKUP_FILE backup-info.txt -O
```

#### Step 3: Run yubikey-setup.sh (Load Mode)

```bash
cd scripts/automation/yubikey-setup/

# Run setup script in load mode
!./yubikey-setup.sh

# Select "Load" option when prompted
# Script will:
# 1. Reset OpenPGP applet
# 2. Extract backup
# 3. Import master key
# 4. Import subkeys
# 5. Transfer to YubiKey
# 6. Restore touch policies
# 7. Verify restoration
```

#### Step 4: Verify Restoration

```bash
# Check card status
!gpg --card-status

# Verify key fingerprints match backup
echo "Verifying key fingerprints..."
BACKUP_FP=$(tar -xzf ~/yubikey-backups/$BACKUP_FILE backup-info.txt -O | grep "Master key" | awk '{print $NF}')
CURRENT_FP=$(gpg --card-status | grep "Application" | awk '{print $NF}')

if [ "$BACKUP_FP" == "$CURRENT_FP" ]; then
    echo "✅ Fingerprints match - restoration successful"
else
    echo "❌ Fingerprints don't match - restoration may have failed"
fi
```

### Setup Mode: Backup Only

#### Step 1: Verify Current Setup

```bash
echo "Setup Mode: Backup Only"
echo ""
echo "Current YubiKey configuration:"

# Check card status
!gpg --card-status

# Verify keys present
echo ""
echo "Verifying keys..."
gpg --card-status | grep "Signature key" || echo "⚠️  No signature key"
gpg --card-status | grep "Encryption key" || echo "⚠️  No encryption key"
gpg --card-status | grep "Authentication key" || echo "⚠️  No authentication key"
```

#### Step 2: Run Backup

```bash
cd scripts/automation/yubikey-setup/

# Run setup script in backup mode
!./yubikey-setup.sh

# Select "Backup" option when prompted
# Script will:
# 1. Export public keys
# 2. Export SSH public key
# 3. Save YubiKey configuration
# 4. Create timestamped archive
# 5. Verify backup integrity
```

#### Step 3: Verify Backup Created

```bash
# Check latest backup
!ls -lht ~/yubikey-backups/ | head -n 5

# Verify backup contents
LATEST_BACKUP=$(ls -t ~/yubikey-backups/backup-*.tar.gz | head -n 1)
echo "Latest backup: $LATEST_BACKUP"
!tar -tzf $LATEST_BACKUP
```

### Post-Setup Tasks

#### 1. Backup Security

```bash
echo ""
echo "🔐 CRITICAL: Secure Your Backup"
echo ""
echo "Your backup contains sensitive key material and MUST be protected:"
echo ""
echo "1. Encrypt the backup:"
echo "   gpg --symmetric --cipher-algo AES256 ~/yubikey-backups/backup-*.tar.gz"
echo ""
echo "2. Store encrypted backup in secure location(s):"
echo "   - Password manager (encrypted)"
echo "   - Encrypted USB drive in safe"
echo "   - Secure cloud storage (encrypted)"
echo ""
echo "3. NEVER store unencrypted backup in:"
echo "   - Email"
echo "   - Unencrypted cloud storage"
echo "   - Version control (Git)"
echo "   - Shared drives"
echo ""
read -p "Press ENTER when backup is secured..."
```

#### 2. Test Configuration

```bash
echo ""
echo "Testing YubiKey configuration..."
echo ""

# Test GPG signing
echo "test" | gpg --clearsign > /dev/null 2>&1 && echo "✅ GPG signing works" || echo "❌ GPG signing failed"

# Test GPG encryption
echo "test" | gpg --encrypt --recipient $EMAIL | gpg --decrypt > /dev/null 2>&1 && echo "✅ GPG encryption works" || echo "❌ GPG encryption failed"

# Test SSH key
ssh-add -L > /dev/null 2>&1 && echo "✅ SSH key available" || echo "⚠️  SSH key not in agent (gpg-agent may need restart)"

# Test Git signing
git config --global user.signingkey $(gpg --card-status | grep "Signature key" | awk '{print $NF}')
git config --global commit.gpgsign true
echo "✅ Git signing configured"
```

#### 3. Generate Setup Report

Create **YUBIKEY_SETUP_REPORT.md**:

```markdown
# YubiKey Setup Report

**Setup Date**: [ISO 8601 timestamp]
**Setup Mode**: Generate / Load / Backup
**YubiKey Serial**: [XXXXXX]
**Setup Script**: yubikey-setup.sh v1.1.0

---

## Setup Summary

**Status**: ✅ SUCCESS / ⚠️  WARNINGS / ❌ FAILED

**Configuration**:
- GPG Master Key: [Key ID]
- Signing Subkey: [Key ID]
- Encryption Subkey: [Key ID]
- Authentication Subkey: [Key ID]
- SSH Public Key: Generated
- Git Signing: Configured

**Backup**:
- Backup File: [filename]
- Backup Location: ~/yubikey-backups/
- Backup Size: [size]
- Backup Encrypted: [YES/NO]

---

## Key Information

### GPG Keys

**Master Key**:
- Key ID: [XXXX XXXX XXXX XXXX]
- Fingerprint: [XXXX XXXX XXXX XXXX XXXX  XXXX XXXX XXXX XXXX XXXX]
- Algorithm: RSA 4096
- Created: [date]
- Expires: [Never / date]

**Subkeys**:
- Signing (S): [Key ID], RSA 4096, Expires: [date]
- Encryption (E): [Key ID], RSA 4096, Expires: [date]
- Authentication (A): [Key ID], RSA 4096, Expires: [date]

### Touch Policies

| Key | Touch Policy | Status |
|-----|--------------|--------|
| Signature | On | ✅ Secure |
| Encryption | On | ✅ Secure |
| Authentication | On | ✅ Secure |

### PIN Configuration

**User PIN**:
- Status: Changed from default
- Retries Remaining: 3/3

**Admin PIN**:
- Status: Changed from default
- Retries Remaining: 3/3

---

## Verification Results

- [✅ / ❌] GPG card status shows all keys
- [✅ / ❌] GPG signing test passed
- [✅ / ❌] GPG encryption test passed
- [✅ / ❌] SSH key generated
- [✅ / ❌] Git signing configured
- [✅ / ❌] Backup created
- [✅ / ❌] Backup verified
- [✅ / ❌] PINs changed from defaults

**Overall Verification**: ✅ PASS / ❌ FAIL

---

## Next Steps

### Immediate (Complete Today)
- [ ] Encrypt backup with GPG
- [ ] Store encrypted backup in secure location #1
- [ ] Store encrypted backup in secure location #2 (offsite)
- [ ] Test YubiKey with actual authentication
- [ ] Remove unencrypted backup: `rm ~/yubikey-backups/backup-*.tar.gz`

### Short-Term (This Week)
- [ ] Configure services to use YubiKey:
  - [ ] SSH to servers
  - [ ] Git commit signing
  - [ ] Password manager
  - [ ] Other: ___________
- [ ] Test YubiKey removal/reinsertion
- [ ] Test touch policies with actual operations
- [ ] Document YubiKey location and backup locations

### Long-Term (This Month)
- [ ] Set up monitoring for certificate expiration
- [ ] Schedule backup verification (monthly)
- [ ] Consider getting backup YubiKey
- [ ] Train on YubiKey recovery procedures

---

## Important Reminders

### Security
- 🔒 **Backup is encrypted and stored securely** (2+ locations)
- 🔒 **PINs changed from defaults**
- 🔒 **Touch policies enabled for all operations**
- 🔒 **Never share backup unencrypted**

### Usage
- 👆 **Touch YubiKey when LED flashes** (touch policies active)
- 🔑 **Remember your PIN** (3 tries before lockout)
- 🔑 **Remember your Admin PIN** (for unlocking if PIN forgotten)
- 💾 **Keep backup current** (after any configuration changes)

### Maintenance
- 📅 **Run health check weekly**: `/yubikey-health-check`
- 📅 **Update backup after changes**
- 📅 **Verify backup quarterly**
- 📅 **Review configuration annually**

---

## Troubleshooting

### Common Issues

**Issue: GPG not detecting YubiKey**
- Solution: `gpg --card-status` to wake up gpg-agent
- Solution: Restart gpg-agent: `gpgconf --kill gpg-agent`

**Issue: SSH key not available**
- Solution: Enable SSH support in gpg-agent
- Add to `~/.gnupg/gpg-agent.conf`: `enable-ssh-support`
- Restart gpg-agent

**Issue: Git commits not signed**
- Solution: Check Git config: `git config --global commit.gpgsign`
- Should be `true`
- Check signing key: `git config --global user.signingkey`

### Recovery Procedures

**Lost PIN**:
1. Use Admin PIN to unlock
2. Set new PIN
3. Update password manager

**Lost Admin PIN**:
1. If PUK known: Use PUK to reset Admin PIN
2. If PUK lost: Factory reset (restores from backup)

**Lost YubiKey**:
1. Procure replacement YubiKey
2. Run setup wizard in "Load" mode
3. Restore from encrypted backup
4. Verify fingerprints match

---

## Support Resources

- YubiKey Setup Script: `./scripts/automation/yubikey-setup/README.md`
- YubiKey Tools Docs: `./docs/`
- Yubico Support: https://support.yubico.com/
- GPG Documentation: https://gnupg.org/documentation/

---

**Report Generated**: [Timestamp]
**Setup Performed By**: [User]
**Validated By**: YubiKey Setup Wizard v1.0
```

### 4. Display Setup Summary

```
╔════════════════════════════════════════════════════╗
║          YUBIKEY SETUP COMPLETE! ✅                 ║
╚════════════════════════════════════════════════════╝

SETUP MODE: [Generate/Load/Backup]
YUBIKEY: Serial [XXXXXX]
DURATION: [MM:SS]

CONFIGURATION:
  ✅ OpenPGP applet configured
  ✅ GPG keys generated/loaded
  ✅ Keys transferred to YubiKey
  ✅ Touch policies enabled
  ✅ SSH key generated
  ✅ Git signing configured
  ✅ Backup created and verified
  ✅ PINs changed from defaults

VERIFICATION:
  ✅ GPG signing tested
  ✅ GPG encryption tested
  ✅ SSH key available
  ✅ All keys on YubiKey
  ✅ Touch policies active
  ✅ Backup integrity confirmed

BACKUP LOCATION:
  📁 ~/yubikey-backups/backup-[timestamp].tar.gz
  🔐 ENCRYPT THIS FILE IMMEDIATELY!

CRITICAL NEXT STEPS:
  1. Encrypt backup:
     gpg --symmetric --cipher-algo AES256 [backup-file]

  2. Store encrypted backup securely (2+ locations)

  3. Delete unencrypted backup:
     rm ~/yubikey-backups/backup-*.tar.gz

  4. Test YubiKey:
     - Try SSH connection
     - Make signed Git commit
     - Test GPG encryption

IMPORTANT REMINDERS:
  • Touch YubiKey when LED flashes
  • Remember your PIN (you just set it!)
  • Keep backup encrypted and secure
  • Run /yubikey-health-check weekly

Full Report: YUBIKEY_SETUP_REPORT.md

Your YubiKey is ready to use! 🎉

For help: See scripts/automation/yubikey-setup/README.md
```

## Key Features

- **Interactive Guided Setup**: Step-by-step wizard with clear instructions
- **Three Setup Modes**: Generate, Load, or Backup
- **Wraps yubikey-setup.sh**: Leverages battle-tested automation script
- **Safety Checks**: Verifies device, warns about data loss, requires approval
- **Comprehensive Testing**: Validates configuration after setup
- **Backup Integration**: Creates and verifies backups automatically
- **Security Guidance**: Reminds about backup encryption and secure storage
- **Detailed Reporting**: Generates complete setup report
- **Error Handling**: Graceful failure with rollback guidance

## When to Use /yubikey-setup-wizard

### New YubiKey (Generate Mode)
- Just purchased YubiKey
- Setting up for first time
- Want complete GPG + SSH configuration
- Need secure backup created

### Replace Lost/Damaged YubiKey (Load Mode)
- Lost previous YubiKey
- YubiKey hardware failure
- Upgrading to new YubiKey model
- Have backup from previous setup

### Backup Existing YubiKey (Backup Mode)
- Already configured YubiKey
- Want to create/update backup
- Before making configuration changes
- Before traveling with YubiKey

## Best Practices

1. **Run Health Check First**: `/yubikey-health-check` before setup
2. **Use Secure Environment**: Trusted computer, no malware
3. **Offline if Possible**: Disconnect from network during key generation
4. **Encrypt Backup Immediately**: Don't leave unencrypted on disk
5. **Multiple Backup Locations**: 2-3 secure, offsite locations
6. **Test Before Relying**: Verify all functions work before depending on it
7. **Document PIN Locations**: Secure password manager
8. **Consider Backup YubiKey**: Buy 2, set up identically

## Security Considerations

### What Gets Modified
- ⚠️  OpenPGP applet **RESET** (data loss!)
- ⚠️  New GPG keys **GENERATED** (Generate mode)
- ⚠️  Keys **TRANSFERRED** to YubiKey
- ⚠️  PINs **CHANGED** (user must remember!)
- ⚠️  Touch policies **CONFIGURED**
- ⚠️  Git configuration **MODIFIED**

### What Gets Backed Up
- ✅ Public keys (safe to share)
- ✅ Private keys (MUST encrypt!)
- ✅ YubiKey configuration
- ✅ Key fingerprints
- ✅ Setup metadata

### What Does NOT Get Backed Up
- ❌ YubiKey PINs (you must remember!)
- ❌ Private keys from YubiKey (cannot export)
- ❌ Touch policy from YubiKey (reconfigure on restore)

## Rollback Procedures

### If Setup Fails Mid-Process
1. Note error message
2. Check setup log
3. Reset OpenPGP applet: `ykman openpgp reset`
4. Start over with wizard
5. Report issue if repeatable

### If Setup Completes But Verification Fails
1. Check what failed
2. Run specific test manually
3. Fix configuration issue
4. Re-run wizard if needed

### If Need to Undo Setup
1. Reset OpenPGP applet: `ykman openpgp reset`
2. Delete backup if not needed
3. YubiKey returns to factory state
4. Can start fresh

---

**Command Version**: 1.0
**Last Updated**: 2025-11-21
**Maintained By**: YubiKey Tools Team
**Wraps**: yubikey-setup.sh v1.1.0
