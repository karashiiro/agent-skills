# Session Management Cheatsheet

Secure session handling and token management.

## DO / DON'T

| DO | DON'T |
|----|-------|
| Generate tokens server-side | Trust client-generated tokens |
| Use 128+ bits of entropy | Use predictable/sequential IDs |
| Regenerate session on auth changes | Keep same session ID after login |
| Set secure cookie attributes | Use default cookie settings |
| Implement absolute timeout | Allow sessions to live forever |
| Invalidate session on logout | Only clear client-side cookie |
| Store tokens in httpOnly cookies | Store tokens in localStorage |

## Session Token Generation

```
# SECURE: Cryptographically random tokens
import secrets

def generate_session_id():
    # 32 bytes = 256 bits of entropy
    return secrets.token_hex(32)

# Result: "a3f2b1c4d5e6f7..." (64 hex chars)

# BAD: Predictable patterns
import random
random.randint(1, 1000000)           # Predictable
uuid.uuid1()                          # Includes MAC address, timestamp
base64(username + timestamp)          # Guessable
md5(user_id + secret)                 # Potentially reversible
```

## Cookie Configuration

```
# SECURE cookie settings
Set-Cookie: session_id=abc123;
    Secure;              # HTTPS only
    HttpOnly;            # No JavaScript access
    SameSite=Strict;     # CSRF protection
    Path=/;              # Scope to site root
    Max-Age=3600;        # 1 hour expiry
    Domain=example.com   # Explicit domain

# Code example
response.set_cookie(
    'session_id',
    value=session_id,
    secure=True,         # Only send over HTTPS
    httponly=True,       # JavaScript cannot read
    samesite='Strict',   # Don't send with cross-site requests
    max_age=3600,        # Expires in 1 hour
    path='/',            # Available site-wide
    domain='example.com' # Only this domain
)
```

## Session Lifecycle

```
# Login: Regenerate session
def login(username, password):
    user = authenticate(username, password)
    if user:
        # CRITICAL: Generate new session ID
        old_session_id = get_session_id(request)
        new_session_id = generate_session_id()
        
        # Transfer session data to new ID
        session_data = load_session(old_session_id)
        delete_session(old_session_id)
        
        session_data['user_id'] = user.id
        session_data['login_time'] = now()
        save_session(new_session_id, session_data)
        
        set_session_cookie(response, new_session_id)

# Logout: Destroy session
def logout():
    session_id = get_session_id(request)
    
    # Server-side: Delete session data
    delete_session(session_id)
    
    # Client-side: Clear cookie
    response.delete_cookie('session_id')
    
    # Optional: Add to revocation list (for JWTs)
    revoke_token(session_id)
```

## Session Timeout

```
# Implement both idle and absolute timeouts

class Session:
    def __init__(self):
        self.created_at = now()
        self.last_activity = now()
    
    def is_valid(self):
        # Absolute timeout: 24 hours max
        if now() - self.created_at > hours(24):
            return False
        
        # Idle timeout: 30 minutes of inactivity
        if now() - self.last_activity > minutes(30):
            return False
        
        return True
    
    def touch(self):
        self.last_activity = now()

# Sliding window for non-sensitive operations
def session_middleware(request):
    session = load_session(request)
    
    if not session.is_valid():
        destroy_session(session)
        return redirect('/login')
    
    session.touch()  # Update last activity
    save_session(session)
```

## JWT Tokens

```
# Short-lived access tokens + refresh tokens

# Access token: Short lived, stateless
access_token = jwt.sign({
    'sub': user_id,
    'exp': now() + minutes(15),  # Short expiry!
    'iat': now(),
    'jti': generate_token_id()   # Unique ID for revocation
}, secret_key, algorithm='HS256')

# Refresh token: Longer lived, stored server-side
refresh_token = generate_secure_token()
store_refresh_token(user_id, refresh_token, expires=days(30))

# Token refresh endpoint
def refresh(refresh_token):
    stored = get_refresh_token(refresh_token)
    if not stored or stored.expired:
        return error('Invalid refresh token')
    
    # Rotate refresh token (one-time use)
    delete_refresh_token(refresh_token)
    new_refresh = generate_secure_token()
    store_refresh_token(stored.user_id, new_refresh)
    
    # Issue new access token
    new_access = sign_access_token(stored.user_id)
    
    return {
        'access_token': new_access,
        'refresh_token': new_refresh
    }
```

## Token Storage (Client-Side)

```
# BEST: HttpOnly cookies (default to this)
# Pros: Not accessible to JavaScript, automatic CSRF protection with SameSite
# Cons: Need CSRF tokens for cross-origin, cookie size limits

# ACCEPTABLE: Memory only (for SPAs)
# Store in JavaScript variable, lost on page refresh
# Pros: Not persisted, not in localStorage
# Cons: Poor UX (logout on refresh)

# AVOID: localStorage/sessionStorage
# Pros: Easy to use
# Cons: Accessible to ANY JavaScript (XSS can steal tokens)

# If you MUST use localStorage (e.g., mobile web):
# - Implement short expiry (15 min)
# - Use refresh token rotation
# - Implement strong CSP
# - Clear on logout
localStorage.removeItem('access_token');  // On logout
```

## Multi-Device Sessions

```
# Track all active sessions per user

class UserSession:
    user_id: int
    session_id: str
    device_info: str
    ip_address: str
    created_at: datetime
    last_activity: datetime

def get_active_sessions(user_id):
    return db.query("SELECT * FROM sessions WHERE user_id = ?", user_id)

def terminate_session(user_id, session_id):
    # Verify user owns the session
    session = get_session(session_id)
    if session.user_id != user_id:
        raise AuthorizationError()
    
    delete_session(session_id)

def terminate_all_sessions(user_id, except_current=None):
    # Terminate all other sessions (e.g., after password change)
    sessions = get_active_sessions(user_id)
    for session in sessions:
        if session.id != except_current:
            delete_session(session.id)
```

## Session Fixation Prevention

```
# Attack: Attacker sets victim's session ID before they log in

# DEFENSE: Always regenerate session ID on authentication state change

# On login
def login_success(user):
    regenerate_session_id()  # New ID, copy data
    session['user_id'] = user.id

# On privilege escalation
def elevate_privileges():
    regenerate_session_id()
    session['admin'] = True

# On logout
def logout():
    destroy_session_completely()  # Don't just clear, destroy
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Tokens in URL | Logged, cached, leaked via Referer | Use cookies or Authorization header |
| No server-side invalidation | Logout doesn't work | Delete session server-side |
| Same session after login | Session fixation | Regenerate on auth changes |
| No absolute timeout | Sessions live forever | Max lifetime (24h for web) |
| Tokens in localStorage | XSS can steal tokens | Use httpOnly cookies |

## Related

- `authentication.md` - Login/logout flows
- `xss-prevention.md` - Protecting tokens from XSS
- `authorization.md` - Session-based access control