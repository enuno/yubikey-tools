---
description: "Review cryptographic operations for correctness, security best practices, and proper use of established crypto libraries"
allowed-tools: ["Read", "Search", "Grep"]
author: "YubiKey Tools Cryptography Team"
version: "1.0"
---

# Validate Cryptography

## Purpose
Perform comprehensive review of all cryptographic operations in the codebase to ensure proper usage of established libraries, secure algorithms, and adherence to cryptographic best practices.

## Cryptography Validation Steps

### 1. Search for Cryptographic Library Usage

```bash
# Find all crypto library imports
!grep -rn "^import cryptography\|^from cryptography" src/ --include="*.py"
!grep -rn "^import nacl\|^from nacl" src/ --include="*.py"
!grep -rn "^import hashlib\|^from hashlib" src/ --include="*.py"
!grep -rn "^import secrets\|^from secrets" src/ --include="*.py"
!grep -rn "^import random\|^from random" src/ --include="*.py"
```

### 2. Check for Custom Cryptographic Implementations

**CRITICAL**: Custom crypto is NEVER acceptable

```bash
# Search for custom crypto implementations (red flags)
!grep -rn "def encrypt\|def decrypt" src/ --include="*.py"
!grep -rn "def hash\|def sign\|def verify" src/ --include="*.py"
!grep -rn "def generate_key\|def derive_key" src/ --include="*.py"
!grep -rn "class.*Cipher\|class.*Hash" src/ --include="*.py"

# These should ALL be using established libraries, not custom implementations
```

### 3. Validate Random Number Generation

```bash
# Check for secure random generation (secrets module)
!grep -rn "secrets\\.token_bytes\|secrets\\.token_hex" src/ --include="*.py"
!grep -rn "secrets\\.randbits\|secrets\\.SystemRandom" src/ --include="*.py"

# Check for INSECURE random usage (should NOT exist in production)
!grep -rn "random\\.random\|random\\.randint" src/ --include="*.py" || echo "✅ No insecure random usage"
!grep -rn "random\\.choice\|random\\.sample" src/ --include="*.py" || echo "✅ No insecure random usage"

# Verify os.urandom usage (acceptable, but secrets is preferred)
!grep -rn "os\\.urandom" src/ --include="*.py"
```

### 4. Check for Deprecated Algorithms

```bash
# Search for deprecated/weak hash algorithms
!grep -rn "hashlib\\.md5\|hashlib\\.sha1" src/ --include="*.py" || echo "✅ No MD5/SHA1 for security"
!grep -rn "MD5\\|md5" src/ --include="*.py" | grep -v "# " | grep -v comment || echo "✅ No MD5 usage"
!grep -rn "SHA1\\|sha1" src/ --include="*.py" | grep -v "SHA256" | grep -v "# " || echo "✅ No SHA1 usage"

# Search for weak encryption algorithms
!grep -rn "DES\\|3DES\\|RC4\\|Blowfish" src/ --include="*.py" || echo "✅ No weak ciphers"

# Search for ECB mode (insecure for most use cases)
!grep -rn "MODE_ECB\|ECB" src/ --include="*.py" || echo "✅ No ECB mode usage"
```

### 5. Review RSA Key Generation and Usage

```bash
# Find RSA key generation
!grep -rn "RSA\\.generate\|generate_private_key.*RSA" src/ --include="*.py"

# Check RSA key sizes (should be ≥2048, preferably ≥4096)
!grep -rn "key_size\\s*=\\s*\\d+" src/ --include="*.py"
!grep -rn "RSA.*2048\|RSA.*4096" src/ --include="*.py"

# Verify RSA padding schemes (should use OAEP, not PKCS1v15 for encryption)
!grep -rn "padding\\.OAEP\|padding\\.PSS" src/ --include="*.py"
!grep -rn "padding\\.PKCS1v15" src/ --include="*.py"
```

### 6. Review Elliptic Curve Cryptography

```bash
# Find ECC usage
!grep -rn "ec\\.generate_private_key\|ECDSA\|ECDH" src/ --include="*.py"

# Check curve choices (prefer Ed25519, X25519, or NIST P-256/P-384)
!grep -rn "Ed25519\|X25519\|SECP256R1\|SECP384R1" src/ --include="*.py"
!grep -rn "SECP256K1" src/ --include="*.py" # Bitcoin curve, context-dependent
```

### 7. Validate Certificate Handling

```bash
# Find certificate operations
!grep -rn "x509\\.load.*certificate\|x509\\.CertificateBuilder" src/ --include="*.py"

# Check certificate validation
!grep -rn "verify_certificate\|verify.*chain" src/ --include="*.py"
!grep -rn "certificate\\.not_valid_before\|not_valid_after" src/ --include="*.py"

# Check for proper certificate verification
@Read: src/validators/certificate_validator.py
```

### 8. Review Key Derivation Functions

```bash
# Find KDF usage
!grep -rn "PBKDF2\|Scrypt\|Argon2\|HKDF" src/ --include="*.py"

# Check iteration counts and parameters
!grep -rn "iterations\\s*=\\s*\\d+" src/ --include="*.py"
!grep -rn "length\\s*=\\s*\\d+\|dklen\\s*=\\s*\\d+" src/ --include="*.py"
```

### 9. Check Attestation Validation

```bash
# Find FIDO2 attestation handling
!grep -rn "attestation\|verify.*attestation" src/ --include="*.py"

# Read attestation validator
@Read: src/validators/attestation_validator.py
```

### 10. Generate Cryptography Validation Report

Create **CRYPTO_VALIDATION_REPORT.md**:

```markdown
# Cryptography Validation Report

**Validation Date**: [ISO 8601 timestamp]
**Validator**: Cryptography Reviewer Agent
**Repository**: yubikey-tools
**Commit**: [git commit hash]

---

## Executive Summary

| Category | Status | Issues | Severity |
|----------|--------|--------|----------|
| Custom Crypto Implementation | [PASS/FAIL] | [N] | [CRITICAL] |
| Random Number Generation | [PASS/FAIL] | [N] | [CRITICAL] |
| Deprecated Algorithms | [PASS/FAIL] | [N] | [HIGH] |
| Key Generation | [PASS/FAIL] | [N] | [HIGH] |
| Certificate Validation | [PASS/FAIL] | [N] | [HIGH] |
| Padding Schemes | [PASS/FAIL] | [N] | [MEDIUM] |
| Key Derivation | [PASS/FAIL] | [N] | [MEDIUM] |

**Overall Cryptographic Security**: 🟢 SECURE / 🟡 NEEDS IMPROVEMENT / 🔴 INSECURE

---

## Critical Issues

### Issue #1: Custom Cryptographic Implementation
**Status**: [FOUND / NOT FOUND]

If found:
- **Location**: `path/to/file.py:line`
- **Code**: `[code snippet]`
- **Risk**: CRITICAL - Never roll your own crypto
- **Remediation**: Replace with established library (cryptography, PyNaCl)

### Issue #2: Insecure Random Number Generation
**Status**: [FOUND / NOT FOUND]

If found:
- **Location**: `path/to/file.py:line`
- **Code**: `random.random()` or `random.randint()`
- **Risk**: CRITICAL - Cryptographically weak randomness
- **Remediation**: Use `secrets` module

```python
# INSECURE
import random
key = random.randbytes(32)  # Predictable!

# SECURE
import secrets
key = secrets.token_bytes(32)  # Cryptographically secure
```

---

## High Severity Issues

### Deprecated Hash Algorithms
**MD5 Usage**: [FOUND / NOT FOUND]
- Location: [if found]
- Remediation: Replace with SHA-256 or higher

**SHA1 Usage for Security**: [FOUND / NOT FOUND]
- Location: [if found]
- Context: [Is this for compatibility or security?]
- Remediation: Use SHA-256 or SHA-3

### Weak Encryption Algorithms
**DES/3DES/RC4**: [FOUND / NOT FOUND]
- Remediation: Use AES-GCM or ChaCha20-Poly1305

### ECB Mode Usage
**Status**: [FOUND / NOT FOUND]
- Risk: Deterministic encryption, pattern leakage
- Remediation: Use GCM, CBC, or CTR mode with proper IV

---

## Cryptographic Library Usage

### Established Libraries ✅
- **cryptography**: [USED / NOT USED]
  - Version: [X.Y.Z]
  - Usage: RSA, ECC, X.509, PKCS#11
- **PyNaCl**: [USED / NOT USED]
  - Version: [X.Y.Z]
  - Usage: Ed25519, X25519, authenticated encryption
- **python-fido2**: [USED / NOT USED]
  - Version: [X.Y.Z]
  - Usage: FIDO2 attestation, credentials

### Library Versions
| Library | Installed | Latest | Status |
|---------|-----------|--------|--------|
| cryptography | [X.Y.Z] | [X.Y.Z] | ✅ / ⚠️ |
| PyNaCl | [X.Y.Z] | [X.Y.Z] | ✅ / ⚠️ |
| python-fido2 | [X.Y.Z] | [X.Y.Z] | ✅ / ⚠️ |

---

## Random Number Generation

### Secure Methods Found ✅
```python
# Files using secrets module
src/core/operations/fido2_operations.py:45
  └─ secrets.token_bytes(32)

src/utils/crypto_utils.py:23
  └─ secrets.token_hex(16)
```

### Insecure Methods Found ⚠️
```python
# Files using random module (should be secrets)
[List files if any found]
```

**Recommendation**: Replace ALL `random` usage with `secrets` for cryptographic purposes.

---

## RSA Key Generation and Usage

### Key Generation Parameters
```python
# Found in: src/core/operations/piv_operations.py:120
private_key = rsa.generate_private_key(
    public_exponent=65537,  # ✅ Standard exponent
    key_size=4096,          # ✅ Strong key size
    backend=default_backend()
)
```

**Assessment**:
- ✅ Key size ≥2048 bits
- ✅ Standard public exponent (65537)
- ✅ Using established library

### RSA Padding Schemes
```python
# Encryption padding (found in: [location])
padding.OAEP(
    mgf=padding.MGF1(algorithm=hashes.SHA256()),
    algorithm=hashes.SHA256(),
    label=None
)  # ✅ CORRECT - OAEP with SHA-256

# Signature padding (found in: [location])
padding.PSS(
    mgf=padding.MGF1(hashes.SHA256()),
    salt_length=padding.PSS.MAX_LENGTH
)  # ✅ CORRECT - PSS with SHA-256
```

**Assessment**:
- ✅ OAEP for encryption (not PKCS1v15)
- ✅ PSS for signatures (not PKCS1v15)
- ✅ SHA-256 or higher for MGF1

### Issues Found
[List any RSA issues]

---

## Elliptic Curve Cryptography

### Curves in Use
```python
# Ed25519 for signatures (found in: [location])
private_key = ed25519.Ed25519PrivateKey.generate()  # ✅ EXCELLENT

# X25519 for key exchange (found in: [location])
private_key = x25519.X25519PrivateKey.generate()    # ✅ EXCELLENT

# NIST P-256 for PIV (found in: [location])
private_key = ec.generate_private_key(ec.SECP256R1())  # ✅ GOOD
```

**Assessment**:
- ✅ Modern curves (Ed25519, X25519)
- ✅ NIST curves where required (PIV compatibility)
- ❌ [Any weak curves like P-192]

---

## Certificate Handling

### Certificate Validation
```python
# Found in: src/validators/certificate_validator.py
def validate_certificate_chain(cert_chain):
    # Check expiration
    if cert.not_valid_after < datetime.now():  # ✅ GOOD
        raise CertificateExpiredError()

    # Verify signature
    issuer_key.verify(...)  # ✅ GOOD

    # Check key usage
    if not check_key_usage(cert, 'digitalSignature'):  # ✅ GOOD
        raise InvalidKeyUsageError()
```

**Assessment**:
- ✅ Expiration checking
- ✅ Signature verification
- ✅ Key usage validation
- ✅ Chain of trust verification

### Issues Found
[List any certificate handling issues]

---

## Key Derivation Functions

### KDF Usage
```python
# PBKDF2 (found in: [location])
kdf = PBKDF2HMAC(
    algorithm=hashes.SHA256(),
    length=32,
    salt=salt,
    iterations=600000,  # ✅ GOOD - OWASP recommends ≥600k for SHA-256
    backend=default_backend()
)

# HKDF (found in: [location])
hkdf = HKDF(
    algorithm=hashes.SHA256(),
    length=32,
    salt=salt,
    info=b'application specific',
    backend=default_backend()
)  # ✅ GOOD
```

**Assessment**:
- ✅ Sufficient iteration count
- ✅ Appropriate salt usage
- ✅ Modern hash functions

---

## FIDO2 Attestation Validation

### Attestation Verification
```python
# Found in: src/validators/attestation_validator.py
@Read: src/validators/attestation_validator.py
```

**Assessment**:
- [✅ / ⚠️ / ❌] Signature verification
- [✅ / ⚠️ / ❌] Certificate chain validation
- [✅ / ⚠️ / ❌] AAGUID validation
- [✅ / ⚠️ / ❌] Attestation format support

---

## Best Practices Compliance

### Cryptographic Best Practices
- [ ] ✅ Use established libraries only (no custom crypto)
- [ ] ✅ Secure random number generation (secrets module)
- [ ] ✅ No deprecated algorithms (MD5, SHA1 for security, DES, RC4)
- [ ] ✅ Strong key sizes (RSA ≥2048, AES ≥128)
- [ ] ✅ Proper padding schemes (OAEP, PSS)
- [ ] ✅ Modern elliptic curves (Ed25519, X25519, P-256+)
- [ ] ✅ Sufficient KDF iterations (PBKDF2 ≥600k for SHA-256)
- [ ] ✅ Certificate chain validation
- [ ] ✅ Attestation verification
- [ ] ✅ Constant-time comparisons for secrets

### YubiKey-Specific Cryptography
- [ ] ✅ PIV key generation uses approved algorithms
- [ ] ✅ OpenPGP key generation follows RFC 4880
- [ ] ✅ FIDO2 attestation properly validated
- [ ] ✅ Certificate expiration checked
- [ ] ✅ Touch policy enforcement
- [ ] ✅ PIN validation before crypto operations

---

## Recommendations

### Immediate Actions (Critical)
1. **Replace custom crypto**: [List any custom implementations to replace]
2. **Fix insecure random**: [List files using random instead of secrets]
3. **Remove deprecated algorithms**: [List MD5/SHA1/DES usage]

### Short-Term Improvements (High Priority)
1. **Update weak RSA keys**: Upgrade any keys < 2048 bits to 4096 bits
2. **Add constant-time comparisons**: For PIN/secret verification
3. **Implement certificate pinning**: For attestation root certificates

### Long-Term Enhancements (Medium Priority)
1. **Consider post-quantum**: Monitor PQC standards (NIST selections)
2. **Hardware-backed keys**: Explore HSM integration where appropriate
3. **Formal verification**: Consider formal crypto proofs for critical operations

---

## Compliance and Standards

### Standards Adherence
- **NIST SP 800-57**: Key management ✅
- **NIST SP 800-131A**: Cryptographic algorithms ✅
- **FIPS 140-2**: [If applicable] [✅ / ⚠️ / ❌]
- **RFC 4880**: OpenPGP ✅
- **RFC 8017**: PKCS#1 (RSA) ✅
- **FIDO2/WebAuthn**: Attestation ✅

---

**Report Generated**: [Timestamp]
**Reviewed By**: Cryptography Reviewer Agent
**Next Review**: [Recommended date]
```

### 11. Display Validation Summary

```
╔════════════════════════════════════════════════════╗
║        CRYPTOGRAPHY VALIDATION COMPLETE            ║
╚════════════════════════════════════════════════════╝

CRYPTOGRAPHIC SECURITY: 🟢 SECURE / 🟡 NEEDS ATTENTION / 🔴 INSECURE

CRITICAL CHECKS:
  Custom Crypto:        ✅ None found
  Insecure Random:      ✅ Using secrets module
  Deprecated Algorithms: ✅ None found

ALGORITHM USAGE:
  RSA:  ✅ 4096-bit with OAEP/PSS
  ECC:  ✅ Ed25519, X25519, P-256
  Hash: ✅ SHA-256, SHA-384, SHA-512
  KDF:  ✅ PBKDF2 (600k+ iterations)

LIBRARIES:
  cryptography: ✅ v[X.Y.Z] (latest)
  PyNaCl:       ✅ v[X.Y.Z] (latest)
  python-fido2: ✅ v[X.Y.Z] (latest)

ISSUES FOUND:
  🔴 Critical:  [N]
  🟠 High:      [N]
  🟡 Medium:    [N]
  🟢 Low:       [N]

TOP RECOMMENDATIONS:
  » [Recommendation 1]
  » [Recommendation 2]
  » [Recommendation 3]

Full Report: CRYPTO_VALIDATION_REPORT.md

OVERALL ASSESSMENT:
  Cryptographic implementations follow security best practices
  and use established, well-vetted libraries.
```

## Key Features

- **Comprehensive Review**: All crypto operations analyzed
- **Best Practices**: Checks against NIST, OWASP, RFC standards
- **Library Validation**: Ensures use of established crypto libraries
- **Algorithm Strength**: Verifies modern, secure algorithms
- **YubiKey-Specific**: PIV, OpenPGP, FIDO2 crypto validation
- **Detailed Reporting**: CRYPTO_VALIDATION_REPORT.md
- **Actionable Recommendations**: Prioritized by severity

## When to Use /validate-crypto

- Before security reviews
- After implementing crypto operations
- Before major releases
- When adding PIV/FIDO2/OpenPGP features
- After updating crypto libraries
- During security audits
- When onboarding new developers
- Monthly as part of security review

## Best Practices

1. **Regular Reviews**: Run monthly or after crypto changes
2. **Library Updates**: Keep crypto libraries current
3. **No Custom Crypto**: NEVER implement custom cryptography
4. **Strong Parameters**: Use recommended key sizes and iterations
5. **Modern Algorithms**: Prefer Ed25519, X25519, AES-GCM
6. **Validate Chains**: Always verify certificate chains
7. **Secure Random**: Use `secrets` module, never `random`
8. **Document Decisions**: Explain crypto choices in code
