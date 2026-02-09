---
name: owasp-appsec
description: Use when reviewing code for security vulnerabilities, implementing authentication/authorization, threat modeling, or building secure APIs. Covers OWASP Top 10:2025, API Security Top 10:2023, ASVS verification, and security implementation cheatsheets.
---

# OWASP Application Security

## Overview

Security guidance based on OWASP standards for identifying and preventing vulnerabilities in web applications and APIs. Routes agents to the right reference for their specific security task.

## When to Use

**Use this skill when:**
- Performing security code review (finding vulnerabilities, insecure patterns)
- Implementing auth/authz (sessions, passwords, access control)
- Threat modeling (identifying attack vectors, security requirements)
- Building or securing APIs (REST, GraphQL)
- Handling sensitive data (encryption, secrets)

**Don't use for:**
- Network security, firewall rules
- Infrastructure hardening (OS, containers)
- Penetration testing execution (use TESTING-GUIDE.md for methodology only)

## Quick Reference

| Task | Reference | Key Search Terms |
|------|-----------|------------------|
| Identify vulnerability category | `references/TOP10-2025.md` | A01-A10, CWE-ID |
| API security issues | `references/API-TOP10-2023.md` | API1-API10, BOLA, BFLA |
| Verification requirements | `references/ASVS-CHECKLIST.md` | L1, L2, L3, V1-V14 |
| Implementation guidance | `references/cheatsheets/` | See CHEATSHEETS-INDEX.md |
| Testing methodology | `references/TESTING-GUIDE.md` | Phases, test categories |
| Report findings | `assets/security-review-template.md` | Severity, remediation |

## OWASP Top 10:2025 Summary

| ID | Category | Key CWEs | Detection Focus |
|----|----------|----------|-----------------|
| A01 | Broken Access Control | 200, 284, 285, 639 | Missing authz checks, IDOR |
| A02 | Security Misconfiguration | 16, 611, 1004 | Default creds, verbose errors |
| A03 | Supply Chain Failures | 829, 1104 | Outdated deps, unverified sources |
| A04 | Cryptographic Failures | 259, 327, 328, 331 | Weak algorithms, hardcoded keys |
| A05 | Injection | 79, 89, 94, 78 | Unsanitized input to interpreters |
| A06 | Insecure Design | 209, 256, 501 | Missing threat modeling, weak controls |
| A07 | Authentication Failures | 287, 384, 613 | Weak passwords, session issues |
| A08 | Integrity Failures | 345, 494, 502 | Unsigned updates, unsafe deserialization |
| A09 | Logging Failures | 117, 223, 778 | Missing audit logs, log injection |
| A10 | Exception Handling | 209, 390, 754 | Verbose errors, unhandled exceptions |

## OWASP API Security Top 10:2023 Summary

| ID | Category | Key Issue |
|----|----------|-----------|
| API1 | Broken Object Level Authorization | Missing per-object authz checks |
| API2 | Broken Authentication | Weak token validation, credential stuffing |
| API3 | Broken Object Property Level Authorization | Exposing/allowing modification of sensitive properties |
| API4 | Unrestricted Resource Consumption | No rate limiting, unbounded queries |
| API5 | Broken Function Level Authorization | Admin endpoints accessible to users |
| API6 | Unrestricted Access to Sensitive Flows | Automated abuse of business logic |
| API7 | Server Side Request Forgery | Unvalidated URLs in server requests |
| API8 | Security Misconfiguration | CORS, missing headers, debug enabled |
| API9 | Improper Inventory Management | Undocumented endpoints, old versions |
| API10 | Unsafe Consumption of APIs | Trusting third-party responses blindly |

## ASVS Verification Levels

| Level | Name | Use When |
|-------|------|----------|
| L1 | Opportunistic | Any application, basic security baseline |
| L2 | Standard | Apps handling sensitive data, most production apps |
| L3 | Advanced | Critical applications (financial, healthcare, high-value) |

Load `references/ASVS-CHECKLIST.md` for detailed verification requirements by category.

## Workflow: Security Code Review

1. **Identify scope**: What component is being reviewed? (auth, API, data handling, etc.)
2. **Load relevant cheatsheet**: `read-resource("gw-skill://owasp-appsec/references/cheatsheets/{topic}.md")`
3. **Check patterns**: Compare code against DO/DON'T patterns in cheatsheet
4. **Map findings**: Use TOP10-2025.md to categorize any issues found
5. **Report**: Use `assets/security-review-template.md` for consistent output

## Workflow: Implementing Security Controls

1. **Identify control needed**: Authentication, input validation, encryption, etc.
2. **Load cheatsheet**: Get implementation guidance for that control
3. **Follow DO patterns**: Implement using the secure patterns shown
4. **Avoid DON'T patterns**: Explicitly check you're not using anti-patterns
5. **Verify against ASVS**: Check implementation meets verification level requirements

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Blocklist validation | Attackers find bypasses | Use allowlist validation |
| Client-only validation | Bypassable via proxy | Always validate server-side |
| Rolling own crypto | Subtle implementation flaws | Use established libraries |
| Secrets in code/config | Leaks via VCS, logs | Use env vars or secret managers |
| Verbose error messages | Leaks system details | Generic user errors, log details |
| Missing rate limiting | DoS, brute force attacks | Implement per-endpoint limits |
| CORS: `*` origin | Cross-site attacks | Allowlist specific origins |
| JWT in localStorage | XSS can steal tokens | Use httpOnly cookies |

## Cheatsheet Index

| Cheatsheet | Use For |
|------------|---------|
| `authentication.md` | Login, password policies, MFA, session tokens |
| `authorization.md` | Access control, RBAC, ABAC, permission checks |
| `injection-prevention.md` | SQL, command, LDAP, XPath injection |
| `input-validation.md` | Allowlists, type checking, canonicalization |
| `xss-prevention.md` | Output encoding, CSP, DOM manipulation |
| `secrets-management.md` | API keys, credentials, key rotation |
| `session-management.md` | Session IDs, timeouts, invalidation |
| `cryptographic-storage.md` | Encryption, hashing, key management |
| `error-handling.md` | Exception handling, logging, info disclosure |

## Loading Nested Resources

```
# Load main OWASP Top 10 reference
read-resource("gw-skill://owasp-appsec/references/TOP10-2025.md")

# Load specific cheatsheet
read-resource("gw-skill://owasp-appsec/references/cheatsheets/authentication.md")

# Load security review template
read-resource("gw-skill://owasp-appsec/assets/security-review-template.md")
```
