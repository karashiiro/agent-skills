# OWASP Testing Guide Summary

Structured methodology for security testing.

## Testing Phases

| Phase | Purpose | When |
|-------|---------|------|
| 1. Reconnaissance | Understand attack surface | Before testing begins |
| 2. Configuration Testing | Check platform security | Early in engagement |
| 3. Identity Management | Test auth/authz flows | Core testing phase |
| 4. Input Handling | Injection and validation | Core testing phase |
| 5. Error Handling | Information disclosure | Throughout testing |
| 6. Cryptography | Data protection | When handling sensitive data |
| 7. Business Logic | Abuse scenarios | After understanding app |
| 8. Client-Side | XSS, DOM issues | Web/mobile apps |

---

## Phase 1: Reconnaissance

**Goal**: Map the application's attack surface.

```
Tests:
- Enumerate all endpoints (API, web pages)
- Identify technologies (frameworks, servers, languages)
- Map authentication/authorization boundaries
- Identify data entry points
- Document third-party integrations

Tools:
- Burp Suite Spider/Crawler
- OWASP ZAP
- Manual exploration with browser devtools
```

**Cheatsheet**: N/A (discovery phase)

---

## Phase 2: Configuration Testing

**Goal**: Verify platform and application are securely configured.

```
Tests:
- Default credentials
- Debug/admin endpoints exposed
- Directory listing enabled
- Security headers present (CSP, HSTS, X-Frame-Options)
- TLS configuration (version, ciphers)
- HTTP methods allowed (disable TRACE, unnecessary OPTIONS)
- Error messages (verbose vs generic)
- File upload restrictions

Checklist:
[ ] No default credentials
[ ] Debug mode disabled
[ ] Security headers configured
[ ] TLS 1.2+ only
[ ] Unnecessary HTTP methods disabled
[ ] Generic error messages
```

**Cheatsheets**: `error-handling.md`, `secrets-management.md`

---

## Phase 3: Identity Management Testing

**Goal**: Verify authentication and authorization are secure.

### Authentication Tests

```
Tests:
- Password policy enforcement
- Account lockout/rate limiting
- Password reset flow security
- Session management (cookies, tokens)
- MFA implementation
- Remember me functionality
- Logout completeness

Attack scenarios:
- Credential stuffing (breach lists)
- Brute force with common passwords
- Session fixation
- Token prediction
- Password reset token abuse

Checklist:
[ ] Passwords >= 12 characters
[ ] Rate limiting on login (5 attempts)
[ ] Session regenerated after login
[ ] Secure cookie flags (HttpOnly, Secure, SameSite)
[ ] Password reset tokens single-use, time-limited
[ ] Logout invalidates session server-side
```

**Cheatsheets**: `authentication.md`, `session-management.md`

### Authorization Tests

```
Tests:
- Horizontal privilege escalation (access other users' data)
- Vertical privilege escalation (access admin functions)
- IDOR (Insecure Direct Object References)
- Missing function-level access control
- Parameter tampering

Attack scenarios:
- Change user_id in requests
- Access /admin without admin role
- Modify role parameter in request
- Access API endpoints directly (bypass UI)

Checklist:
[ ] Every endpoint has authorization check
[ ] Users can only access own resources
[ ] Admin functions require admin role
[ ] Authorization checked server-side
[ ] Unpredictable resource IDs (UUIDs)
```

**Cheatsheets**: `authorization.md`

---

## Phase 4: Input Handling Testing

**Goal**: Verify all input is properly validated and sanitized.

### Injection Tests

```
Tests:
- SQL injection (all parameters)
- Command injection (file paths, system calls)
- LDAP injection (if applicable)
- XPath injection (if applicable)
- NoSQL injection (MongoDB, etc.)
- Template injection

Payloads to try:
- SQL: ' OR '1'='1, ' UNION SELECT, ; DROP TABLE
- Command: ; ls, | cat /etc/passwd, `whoami`
- NoSQL: {"$gt": ""}, {"$ne": null}

Checklist:
[ ] All queries use parameterization
[ ] No user input in system commands
[ ] Input type validation (string, int, etc.)
[ ] Allowlist validation for structured input
```

**Cheatsheets**: `injection-prevention.md`, `input-validation.md`

### XSS Tests

```
Tests:
- Reflected XSS (search, error pages)
- Stored XSS (comments, profiles, any saved content)
- DOM XSS (client-side JavaScript)

Payloads to try:
- <script>alert(1)</script>
- <img src=x onerror=alert(1)>
- javascript:alert(1)
- '"><script>alert(1)</script>
- {{7*7}} (template injection)

Checklist:
[ ] Output encoding for HTML context
[ ] Content Security Policy implemented
[ ] No innerHTML with user data
[ ] URL validation (no javascript:)
```

**Cheatsheets**: `xss-prevention.md`

---

## Phase 5: Error Handling Testing

**Goal**: Verify errors don't leak sensitive information.

```
Tests:
- Trigger application errors (invalid input, missing params)
- Force database errors
- Request non-existent resources
- Send malformed requests

Look for:
- Stack traces in responses
- Database error messages
- File paths revealed
- Framework/version information
- Different error messages for valid vs invalid users

Checklist:
[ ] Generic error messages returned
[ ] Detailed errors logged server-side only
[ ] Consistent errors (no user enumeration)
[ ] Custom error pages (no default framework errors)
```

**Cheatsheets**: `error-handling.md`

---

## Phase 6: Cryptography Testing

**Goal**: Verify sensitive data is properly protected.

```
Tests:
- TLS/SSL configuration
- Password hashing algorithm
- Encryption at rest
- Key management
- Random number generation

Checklist:
[ ] TLS 1.2+ with strong ciphers
[ ] Passwords hashed with Argon2id/bcrypt
[ ] Sensitive data encrypted at rest
[ ] No hardcoded secrets in code
[ ] Cryptographic randomness for tokens
```

**Cheatsheets**: `cryptographic-storage.md`, `secrets-management.md`

---

## Phase 7: Business Logic Testing

**Goal**: Verify business rules can't be bypassed.

```
Tests:
- Workflow bypass (skip steps in multi-step process)
- Parameter manipulation (price, quantity, discount)
- Race conditions (double spending)
- Abuse of features (referral fraud, review bombing)
- Rate limiting bypass

Examples:
- Change price in cart request
- Apply discount code multiple times
- Submit order faster than validation
- Create fake referrals

Checklist:
[ ] Server-side validation of all business rules
[ ] Multi-step processes enforce sequence
[ ] Idempotency for sensitive operations
[ ] Rate limiting on abuse-prone features
```

**Cheatsheets**: `input-validation.md`, `authorization.md`

---

## Phase 8: Client-Side Testing

**Goal**: Verify client-side security controls.

```
Tests:
- DOM XSS vulnerabilities
- Sensitive data in client storage
- Postmessage security
- Websocket security
- Client-side access control (bypassed via API)

Checklist:
[ ] No sensitive data in localStorage
[ ] postMessage validates origin
[ ] WebSocket connections authenticated
[ ] Access control enforced server-side
[ ] Client-side code doesn't expose secrets
```

**Cheatsheets**: `xss-prevention.md`, `session-management.md`

---

## Manual vs Automated Testing

| Test Type | Automated | Manual |
|-----------|-----------|--------|
| SQL Injection | SQLmap, Burp Scanner | Complex injection, second-order |
| XSS | ZAP, Burp | DOM XSS, context-specific |
| Configuration | Nmap, SSL Labs | Business logic impact |
| Authentication | Hydra (rate limiting) | Workflow analysis |
| Authorization | Custom scripts | IDOR discovery, privilege escalation |
| Business Logic | N/A | Always manual |
| Cryptography | testssl.sh, crypto-audit | Key management review |

**Rule**: Automated tools find ~30% of issues. Manual testing required for business logic, complex authorization, and context-specific vulnerabilities.

---

## Testing Checklist Summary

```
[ ] Reconnaissance complete - attack surface mapped
[ ] Configuration hardened - no defaults, headers set
[ ] Authentication secure - strong passwords, rate limiting, MFA
[ ] Authorization verified - no IDOR, no privilege escalation
[ ] Injection tested - SQL, command, XSS, template
[ ] Error handling secure - no info leakage
[ ] Cryptography verified - strong algorithms, secure storage
[ ] Business logic tested - no workflow bypass, rate limiting
[ ] Client-side secure - no DOM XSS, secure storage
```