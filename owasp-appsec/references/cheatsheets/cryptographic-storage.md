# Cryptographic Storage Cheatsheet

Encryption, hashing, and key management.

## DO / DON'T

| DO | DON'T |
|----|-------|
| Use AES-256-GCM for encryption | Use DES, 3DES, RC4, ECB mode |
| Use Argon2id for password hashing | Use MD5, SHA1, plain SHA256 |
| Use cryptographic random generators | Use Math.random() or time-based |
| Use established crypto libraries | Roll your own crypto |
| Rotate encryption keys | Use same key forever |
| Store keys in secret managers | Hardcode keys in source |

## Algorithm Selection

```
# Symmetric Encryption
RECOMMENDED: AES-256-GCM (authenticated encryption)
ACCEPTABLE:  AES-256-CBC with HMAC
AVOID:       DES, 3DES, RC4, Blowfish, ECB mode

# Asymmetric Encryption
RECOMMENDED: RSA-2048+ with OAEP, or ECDSA P-256+
AVOID:       RSA-1024, RSA with PKCS#1 v1.5

# Password Hashing
RECOMMENDED: Argon2id (memory-hard)
ACCEPTABLE:  bcrypt (cost 10+), scrypt
AVOID:       MD5, SHA1, SHA256 (even with salt)

# General Hashing (integrity, non-password)
RECOMMENDED: SHA-256, SHA-384, SHA-512
AVOID:       MD5, SHA1 (collision attacks)

# Message Authentication
RECOMMENDED: HMAC-SHA256
AVOID:       HMAC-MD5, HMAC-SHA1
```

## Encryption Patterns

```
# AES-256-GCM encryption (authenticated)

def encrypt(plaintext, key):
    # Generate random IV (never reuse with same key!)
    iv = generate_random_bytes(12)  # 96 bits for GCM
    
    cipher = AES.new(key, AES.MODE_GCM, nonce=iv)
    ciphertext, tag = cipher.encrypt_and_digest(plaintext)
    
    # Return IV + ciphertext + auth tag
    return iv + ciphertext + tag

def decrypt(data, key):
    iv = data[:12]
    ciphertext = data[12:-16]
    tag = data[-16:]
    
    cipher = AES.new(key, AES.MODE_GCM, nonce=iv)
    try:
        plaintext = cipher.decrypt_and_verify(ciphertext, tag)
        return plaintext
    except InvalidTag:
        raise DecryptionError("Data tampered or wrong key")

# NEVER reuse IV with same key - catastrophic for GCM!
```

## Password Hashing

```
# Argon2id (recommended)
def hash_password(password):
    return argon2.hash(
        password,
        time_cost=3,        # Iterations
        memory_cost=65536,  # 64 MB
        parallelism=4       # Threads
    )

def verify_password(hash, password):
    try:
        return argon2.verify(hash, password)
    except VerifyMismatchError:
        return False

# bcrypt (acceptable)
def hash_password_bcrypt(password):
    # Cost factor 10-12 recommended
    return bcrypt.hashpw(password, bcrypt.gensalt(rounds=12))

# NEVER do this for passwords:
hash = sha256(password)              # Too fast, no salt
hash = sha256(password + salt)       # Still too fast
hash = md5(password)                 # Broken algorithm
```

## Random Number Generation

```
# SECURE: Use cryptographic RNG

# Python
import secrets
token = secrets.token_hex(32)         # 64-char hex string
bytes = secrets.token_bytes(32)       # 32 random bytes
code = secrets.randbelow(1000000)     # Random int < 1M

# Node.js
const crypto = require('crypto');
const token = crypto.randomBytes(32).toString('hex');

# Java
SecureRandom random = new SecureRandom();
byte[] bytes = new byte[32];
random.nextBytes(bytes);

# INSECURE: Never use for security
Math.random()                         # Predictable
random.randint(0, 1000000)            # Predictable seed
new Date().getTime()                  # Guessable
uuid.uuid1()                          # Contains MAC, timestamp
```

## Key Management

```
# Key derivation from password (for user-encrypted data)
def derive_key(password, salt):
    return PBKDF2(
        password,
        salt,
        iterations=310000,  # OWASP 2023 recommendation
        hash='sha256',
        key_length=32       # 256 bits
    )

# Key rotation pattern
class EncryptionService:
    def __init__(self):
        self.keys = {
            'v2': load_key('ENCRYPTION_KEY_V2'),  # Current
            'v1': load_key('ENCRYPTION_KEY_V1'),  # Previous
        }
        self.current_version = 'v2'
    
    def encrypt(self, data):
        encrypted = encrypt_aes_gcm(data, self.keys[self.current_version])
        return f"{self.current_version}:{encrypted}"  # Include version
    
    def decrypt(self, data):
        version, encrypted = data.split(':', 1)
        key = self.keys.get(version)
        if not key:
            raise DecryptionError(f"Unknown key version: {version}")
        return decrypt_aes_gcm(encrypted, key)
```

## Data at Rest

```
# Encrypt sensitive fields before storage

class User:
    id: int
    email: str                    # Searchable, not encrypted
    email_hash: str               # For lookup without decryption
    ssn_encrypted: bytes          # Encrypted, not searchable
    
    def set_ssn(self, ssn):
        self.ssn_encrypted = encrypt(ssn, get_data_key())
    
    def get_ssn(self):
        return decrypt(self.ssn_encrypted, get_data_key())

# Envelope encryption for large data
def encrypt_large_file(file_data):
    # Generate data encryption key (DEK)
    dek = generate_random_bytes(32)
    
    # Encrypt data with DEK
    encrypted_data = encrypt(file_data, dek)
    
    # Encrypt DEK with key encryption key (KEK) from KMS
    encrypted_dek = kms.encrypt(dek)
    
    return {
        'encrypted_dek': encrypted_dek,
        'encrypted_data': encrypted_data
    }
```

## Secure Comparison

```
# Use constant-time comparison to prevent timing attacks

# SECURE
import hmac

def secure_compare(a, b):
    return hmac.compare_digest(a, b)

# INSECURE - timing attack possible
def insecure_compare(a, b):
    return a == b  # Returns early on first mismatch!

# Usage for token validation
def validate_token(provided, stored):
    return secure_compare(
        provided.encode(),
        stored.encode()
    )
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Reusing IV/nonce | Catastrophic for GCM | Generate new IV each encryption |
| ECB mode | Pattern leakage | Use GCM or CBC with HMAC |
| Low bcrypt cost | Fast to crack | Use cost 10+ |
| Predictable random | Guessable tokens | Use secrets/SecureRandom |
| Key in source code | Key leaked | Use secret manager |
| No key rotation | Can't recover from leak | Design for rotation |

## Related

- `secrets-management.md` - Storing encryption keys
- `authentication.md` - Password hashing
- `session-management.md` - Token generation