# OWASP ASVS 5.0 Verification Checklist

Application Security Verification Standard requirements organized by level and category.

## Verification Levels

| Level | Name | Assurance | Use For |
|-------|------|-----------|--------|
| L1 | Opportunistic | Baseline | Any application, minimum security |
| L2 | Standard | Moderate | Apps with sensitive data, most production apps |
| L3 | Advanced | High | Critical apps (financial, healthcare, government) |

**Note**: Higher levels include all lower level requirements.

---

## V1: Architecture, Design, and Threat Modeling

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 1.1 |   | ✓ | ✓ | Verify security is addressed in design phase |
| 1.2 |   | ✓ | ✓ | Verify threat model exists for the application |
| 1.3 |   |   | ✓ | Verify all high-value business logic has threat model |
| 1.4 |   | ✓ | ✓ | Verify all input has clear trust boundaries |
| 1.5 |   |   | ✓ | Verify cryptographic modules fail securely |

---

## V2: Authentication

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 2.1.1 | ✓ | ✓ | ✓ | Verify passwords are at least 12 characters |
| 2.1.2 | ✓ | ✓ | ✓ | Verify passwords can be at least 64 characters |
| 2.1.3 | ✓ | ✓ | ✓ | Verify password truncation does not occur |
| 2.1.4 | ✓ | ✓ | ✓ | Verify any printable Unicode character allowed in passwords |
| 2.1.5 | ✓ | ✓ | ✓ | Verify users can change their password |
| 2.1.6 | ✓ | ✓ | ✓ | Verify password change requires current password |
| 2.1.7 |   | ✓ | ✓ | Verify passwords checked against breached password list |
| 2.1.8 |   | ✓ | ✓ | Verify password strength meter helps users |
| 2.1.9 |   | ✓ | ✓ | Verify no periodic credential rotation required |
| 2.2.1 | ✓ | ✓ | ✓ | Verify anti-automation controls for credential stuffing |
| 2.2.2 | ✓ | ✓ | ✓ | Verify weak authenticator use is limited |
| 2.2.3 |   | ✓ | ✓ | Verify secure recovery mechanism exists |
| 2.3.1 |   | ✓ | ✓ | Verify system-generated passwords are random |
| 2.3.2 |   |   | ✓ | Verify MFA required for sensitive transactions |
| 2.3.3 |   |   | ✓ | Verify hardware-backed MFA for highest assurance |

---

## V3: Session Management

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 3.1.1 | ✓ | ✓ | ✓ | Verify session tokens are generated server-side |
| 3.2.1 | ✓ | ✓ | ✓ | Verify session token regenerated after login |
| 3.2.2 | ✓ | ✓ | ✓ | Verify session tokens have at least 128 bits entropy |
| 3.2.3 | ✓ | ✓ | ✓ | Verify session tokens stored securely (httpOnly, secure) |
| 3.3.1 | ✓ | ✓ | ✓ | Verify logout invalidates session server-side |
| 3.3.2 |   | ✓ | ✓ | Verify session timeout after inactivity |
| 3.3.3 |   | ✓ | ✓ | Verify absolute session timeout exists |
| 3.3.4 |   |   | ✓ | Verify user can terminate all active sessions |
| 3.4.1 | ✓ | ✓ | ✓ | Verify cookie-based tokens have Secure attribute |
| 3.4.2 | ✓ | ✓ | ✓ | Verify cookie-based tokens have HttpOnly attribute |
| 3.4.3 | ✓ | ✓ | ✓ | Verify cookie-based tokens have SameSite attribute |

---

## V4: Access Control

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 4.1.1 | ✓ | ✓ | ✓ | Verify access control enforced on trusted server |
| 4.1.2 | ✓ | ✓ | ✓ | Verify access control fails securely (deny by default) |
| 4.1.3 | ✓ | ✓ | ✓ | Verify principle of least privilege enforced |
| 4.2.1 | ✓ | ✓ | ✓ | Verify users can only access their own data |
| 4.2.2 | ✓ | ✓ | ✓ | Verify sensitive data protected from unauthorized access |
| 4.3.1 |   | ✓ | ✓ | Verify admin functions have access control |
| 4.3.2 |   | ✓ | ✓ | Verify directory listing disabled |
| 4.3.3 |   |   | ✓ | Verify application has additional authorization for sensitive operations |

---

## V5: Input Validation

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 5.1.1 | ✓ | ✓ | ✓ | Verify HTTP parameter pollution defenses |
| 5.1.2 | ✓ | ✓ | ✓ | Verify framework protects against mass assignment |
| 5.1.3 | ✓ | ✓ | ✓ | Verify all input validated against allowlist |
| 5.1.4 | ✓ | ✓ | ✓ | Verify structured data strongly typed and validated |
| 5.2.1 | ✓ | ✓ | ✓ | Verify HTML form input sanitized |
| 5.2.2 | ✓ | ✓ | ✓ | Verify unstructured data sanitized |
| 5.3.1 | ✓ | ✓ | ✓ | Verify output encoding for context (HTML, JS, CSS, URL) |
| 5.3.2 | ✓ | ✓ | ✓ | Verify output encoding preserves user's character set |
| 5.4.1 | ✓ | ✓ | ✓ | Verify parameterized queries used for SQL |
| 5.4.2 | ✓ | ✓ | ✓ | Verify parameterized queries used for LDAP |
| 5.4.3 | ✓ | ✓ | ✓ | Verify OS command injection prevented |

---

## V6: Cryptography

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 6.1.1 |   | ✓ | ✓ | Verify regulated data encrypted at rest |
| 6.1.2 |   | ✓ | ✓ | Verify PII encrypted at rest |
| 6.2.1 | ✓ | ✓ | ✓ | Verify all cryptographic modules fail securely |
| 6.2.2 | ✓ | ✓ | ✓ | Verify only approved algorithms used |
| 6.2.3 |   | ✓ | ✓ | Verify random numbers from secure generator |
| 6.2.4 |   |   | ✓ | Verify cryptographic key management documented |
| 6.3.1 |   | ✓ | ✓ | Verify all random values generated using approved CSPRNG |
| 6.3.2 |   |   | ✓ | Verify GUIDs use GUID v4 or cryptographically secure generator |
| 6.4.1 | ✓ | ✓ | ✓ | Verify secrets not stored in code |
| 6.4.2 |   | ✓ | ✓ | Verify key rotation supported |

---

## V7: Error Handling and Logging

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 7.1.1 | ✓ | ✓ | ✓ | Verify generic error message shown to users |
| 7.1.2 | ✓ | ✓ | ✓ | Verify exception handling used across codebase |
| 7.1.3 |   | ✓ | ✓ | Verify security errors logged |
| 7.1.4 |   | ✓ | ✓ | Verify error handling logic is centralized |
| 7.2.1 |   | ✓ | ✓ | Verify authentication events logged |
| 7.2.2 |   | ✓ | ✓ | Verify access control failures logged |
| 7.2.3 |   | ✓ | ✓ | Verify input validation failures logged |
| 7.2.4 |   |   | ✓ | Verify all sensitive operations logged |
| 7.3.1 |   | ✓ | ✓ | Verify sensitive data not logged |
| 7.3.2 |   |   | ✓ | Verify logs contain sufficient context |
| 7.4.1 |   |   | ✓ | Verify logs protected from tampering |

---

## V8: Data Protection

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 8.1.1 | ✓ | ✓ | ✓ | Verify sensitive data identified and classified |
| 8.1.2 |   | ✓ | ✓ | Verify sensitive data minimized in collection |
| 8.2.1 | ✓ | ✓ | ✓ | Verify sensitive data not in URL parameters |
| 8.2.2 | ✓ | ✓ | ✓ | Verify sensitive data not cached in browser |
| 8.2.3 |   | ✓ | ✓ | Verify sensitive data in POST body, not GET |
| 8.3.1 | ✓ | ✓ | ✓ | Verify sensitive data masked in UI |
| 8.3.2 |   | ✓ | ✓ | Verify users can export their data |
| 8.3.3 |   | ✓ | ✓ | Verify users can request data deletion |

---

## V9: Communications

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 9.1.1 | ✓ | ✓ | ✓ | Verify TLS used for all connections |
| 9.1.2 | ✓ | ✓ | ✓ | Verify TLS 1.2 or higher used |
| 9.1.3 |   | ✓ | ✓ | Verify strong cipher suites used |
| 9.2.1 | ✓ | ✓ | ✓ | Verify connections to external systems encrypted |
| 9.2.2 |   | ✓ | ✓ | Verify certificate validation performed |
| 9.2.3 |   |   | ✓ | Verify certificate pinning used for mobile apps |

---

## V10: Malicious Code

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 10.1.1 |   |   | ✓ | Verify code analysis tools used |
| 10.2.1 | ✓ | ✓ | ✓ | Verify no undocumented features |
| 10.2.2 | ✓ | ✓ | ✓ | Verify no time bombs or logic bombs |
| 10.3.1 |   | ✓ | ✓ | Verify updates signed and verified |
| 10.3.2 |   | ✓ | ✓ | Verify integrity of third-party components |

---

## V11: Business Logic

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 11.1.1 | ✓ | ✓ | ✓ | Verify business logic flows in expected sequence |
| 11.1.2 | ✓ | ✓ | ✓ | Verify business logic limits and boundaries enforced |
| 11.1.3 |   | ✓ | ✓ | Verify business logic resistant to race conditions |
| 11.1.4 |   | ✓ | ✓ | Verify anti-automation controls exist |
| 11.1.5 |   |   | ✓ | Verify business logic monitored for anomalies |

---

## V12: Files and Resources

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 12.1.1 | ✓ | ✓ | ✓ | Verify file uploads limited in size |
| 12.1.2 | ✓ | ✓ | ✓ | Verify file type validation (not just extension) |
| 12.1.3 | ✓ | ✓ | ✓ | Verify uploaded files stored outside webroot |
| 12.2.1 | ✓ | ✓ | ✓ | Verify user-submitted filenames sanitized |
| 12.3.1 | ✓ | ✓ | ✓ | Verify path traversal prevented |
| 12.4.1 |   | ✓ | ✓ | Verify files from untrusted sources scanned |

---

## V13: API and Web Services

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 13.1.1 | ✓ | ✓ | ✓ | Verify all API endpoints require authentication |
| 13.1.2 | ✓ | ✓ | ✓ | Verify API versioning implemented |
| 13.2.1 | ✓ | ✓ | ✓ | Verify RESTful services validate Content-Type |
| 13.2.2 | ✓ | ✓ | ✓ | Verify message size limits enforced |
| 13.2.3 |   | ✓ | ✓ | Verify CORS policy is restrictive |
| 13.3.1 | ✓ | ✓ | ✓ | Verify GraphQL or SOAP input validated |
| 13.3.2 |   | ✓ | ✓ | Verify query depth limits for GraphQL |

---

## V14: Configuration

| Req | L1 | L2 | L3 | Requirement |
|-----|----|----|----|-----------|
| 14.1.1 | ✓ | ✓ | ✓ | Verify build process produces identical artifacts |
| 14.1.2 |   | ✓ | ✓ | Verify compiler flags enable all warnings |
| 14.2.1 | ✓ | ✓ | ✓ | Verify components from trusted sources |
| 14.2.2 | ✓ | ✓ | ✓ | Verify dependencies up to date |
| 14.2.3 |   | ✓ | ✓ | Verify unused dependencies removed |
| 14.3.1 | ✓ | ✓ | ✓ | Verify security headers sent |
| 14.3.2 | ✓ | ✓ | ✓ | Verify default CSP prevents inline code |
| 14.4.1 | ✓ | ✓ | ✓ | Verify HTTP responses contain correct Content-Type |
| 14.5.1 |   | ✓ | ✓ | Verify build pipeline isolated |
| 14.5.2 |   |   | ✓ | Verify build pipeline audited |

---

## Quick Level Selection Guide

**Choose L1 if:**
- Building internal tools
- Low-risk public applications
- MVP/prototype phase
- No sensitive data handled

**Choose L2 if:**
- Handling user PII
- Processing payments
- B2B applications
- Standard production applications

**Choose L3 if:**
- Financial services
- Healthcare data (HIPAA)
- Government systems
- Critical infrastructure
- High-value targets