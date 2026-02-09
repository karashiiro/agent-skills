# Authentication Cheatsheet

Secure implementation patterns for user authentication.

## DO / DON'T

| DO | DON'T |
|----|-------|
| Use established auth libraries | Roll your own authentication |
| Hash passwords with Argon2id/bcrypt/scrypt | Use MD5, SHA1, or plain SHA256 for passwords |
| Enforce minimum 12 character passwords | Require complex character rules |
| Check passwords against breach lists | Only check password length |
| Implement rate limiting on login | Allow unlimited login attempts |
| Regenerate session after login | Keep same session ID after auth |
| Use MFA for sensitive accounts | Rely on passwords alone |
| Return generic error messages | Say "password incorrect" vs "user not found" |

## Password Storage

```
# SECURE: Use adaptive hashing
password_hash = argon2id.hash(password, {
    time_cost: 3,
    memory_cost: 65536,  # 64 MB
    parallelism: 4
})

# VERIFY
is_valid = argon2id.verify(password_hash, submitted_password)

# Algorithm priority (best to acceptable):
# 1. Argon2id (recommended)
# 2. bcrypt (cost >= 10)
# 3. scrypt (N=2^17, r=8, p=1)
```

```
# INSECURE - Never do this
hash = sha256(password)           # No salt, fast to crack
hash = md5(password + salt)        # MD5 is broken
hash = sha256(password + pepper)   # Still too fast
```

## Login Flow

```
function login(username, password):
    # Rate limiting check
    if is_rate_limited(username, request.ip):
        return error("Too many attempts. Try again later.")
    
    user = find_user(username)
    
    # Constant-time comparison even if user not found
    if user is null:
        # Still hash to prevent timing attacks
        argon2id.hash(password)
        record_failed_attempt(username, request.ip)
        return error("Invalid credentials")  # Generic message
    
    if not argon2id.verify(user.password_hash, password):
        record_failed_attempt(username, request.ip)
        return error("Invalid credentials")  # Same message
    
    # Success - regenerate session
    session.regenerate()
    session.user_id = user.id
    clear_failed_attempts(username)
    
    return success()
```

## Rate Limiting Strategy

```
# Implement progressive delays
attempts = get_failed_attempts(username, ip)

if attempts >= 3:
    delay_seconds = min(2 ^ (attempts - 3), 3600)  # Max 1 hour
    if last_attempt + delay_seconds > now:
        return rate_limited

if attempts >= 10:
    require_captcha = true

if attempts >= 20:
    lock_account_temporarily(username)
    notify_user_via_email(username)
```

## MFA Implementation

```
# TOTP (Time-based One-Time Password)
# DO:
- Use 6+ digit codes
- Allow 1-2 time step tolerance (30-60 seconds)
- Store backup codes securely (hashed)
- Require MFA re-enrollment for recovery

# DON'T:
- Send codes via SMS (SIM swap attacks)
- Allow MFA bypass without strong verification
- Store TOTP secrets in plain text
```

## Password Reset

```
# SECURE reset flow
function initiate_reset(email):
    user = find_user_by_email(email)
    
    # Always show same response (prevent enumeration)
    if user is not null:
        token = generate_secure_random(32)  # 256 bits
        store_reset_token(user.id, hash(token), expires=1_hour)
        send_email(email, reset_link_with_token)
    
    return "If email exists, reset instructions sent"

function complete_reset(token, new_password):
    token_hash = hash(token)
    reset = find_valid_reset_by_hash(token_hash)
    
    if reset is null or reset.expired:
        return error("Invalid or expired reset link")
    
    # Invalidate all sessions and the token
    invalidate_all_sessions(reset.user_id)
    delete_reset_token(token_hash)
    update_password(reset.user_id, new_password)
    
    return success()
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Password in URL | Logged in server logs, browser history | Use POST body only |
| Remember me never expires | Stolen tokens valid forever | Set max lifetime (30 days) |
| Email as username displayed | Exposes user emails | Show display name instead |
| No account lockout | Brute force possible | Implement progressive delays |
| Same reset token reusable | Attackers can intercept and reuse | Single-use tokens only |

## Related

- `session-management.md` - Session token handling
- `secrets-management.md` - Storing credentials securely
- `cryptographic-storage.md` - Hashing algorithms