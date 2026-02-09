# Security Cheatsheets Index

Quick reference for finding the right cheatsheet for your security task.

## By Security Task

| Task | Cheatsheet | Key Patterns |
|------|------------|-------------|
| Implement login/logout | `authentication.md` | Password policies, MFA, credential storage |
| Check user permissions | `authorization.md` | RBAC, ABAC, ownership checks |
| Prevent SQL/command injection | `injection-prevention.md` | Parameterized queries, input escaping |
| Validate user input | `input-validation.md` | Allowlists, type coercion, canonicalization |
| Prevent XSS attacks | `xss-prevention.md` | Output encoding, CSP, DOM sanitization |
| Handle API keys/passwords | `secrets-management.md` | Env vars, vaults, rotation |
| Manage user sessions | `session-management.md` | Token generation, expiration, invalidation |
| Encrypt data at rest | `cryptographic-storage.md` | Algorithm selection, key management |
| Handle errors securely | `error-handling.md` | Generic messages, secure logging |

## By Vulnerability Category

| OWASP Category | Primary Cheatsheet(s) |
|----------------|----------------------|
| A01: Broken Access Control | `authorization.md`, `session-management.md` |
| A02: Security Misconfiguration | `error-handling.md`, `secrets-management.md` |
| A03: Supply Chain Failures | (use dependency scanning tools) |
| A04: Cryptographic Failures | `cryptographic-storage.md`, `secrets-management.md` |
| A05: Injection | `injection-prevention.md`, `input-validation.md` |
| A06: Insecure Design | (use threat modeling) |
| A07: Authentication Failures | `authentication.md`, `session-management.md` |
| A08: Integrity Failures | `cryptographic-storage.md` |
| A09: Logging Failures | `error-handling.md` |
| A10: Exception Handling | `error-handling.md` |

## Cheatsheet Format

Each cheatsheet follows this structure:

1. **Overview** - What the cheatsheet covers
2. **DO/DON'T Table** - Quick patterns to follow/avoid
3. **Code Patterns** - Language-agnostic secure implementations
4. **Common Mistakes** - Specific anti-patterns to avoid
5. **Related** - Links to other relevant cheatsheets

## Loading Cheatsheets

```
read-resource("gw-skill://owasp-appsec/references/cheatsheets/authentication.md")
read-resource("gw-skill://owasp-appsec/references/cheatsheets/authorization.md")
read-resource("gw-skill://owasp-appsec/references/cheatsheets/injection-prevention.md")
```