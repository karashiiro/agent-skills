# Security Review Report

**Application**: [Application Name]
**Version**: [Version/Commit]
**Review Date**: [Date]
**Reviewer**: [Name/ID]
**Request ID**: [Request ID for tracking]

---

## Executive Summary

[1-2 paragraph summary of findings and overall security posture]

**Risk Level**: [Critical / High / Medium / Low]

**Key Findings**:
- [Finding 1 - brief]
- [Finding 2 - brief]
- [Finding 3 - brief]

---

## Findings

### Finding 1: [Title]

| Field | Value |
|-------|-------|
| **Severity** | Critical / High / Medium / Low / Info |
| **OWASP Category** | A01:2025 Broken Access Control |
| **CWE** | CWE-XXX |
| **Status** | Open / Fixed / Accepted Risk |

**Description**:
[Detailed description of the vulnerability]

**Affected Code**:
```
File: src/controllers/user.js:42

// Vulnerable code snippet
const user = await User.findById(req.params.id);  // No ownership check
```

**Attack Scenario**:
[How an attacker would exploit this vulnerability]

**Remediation**:
[Specific steps to fix the issue]

```
// Recommended fix
const user = await User.findById(req.params.id);
if (user.ownerId !== req.user.id) {
    throw new ForbiddenError('Access denied');
}
```

**References**:
- OWASP: [Link]
- CWE: [Link]

---

### Finding 2: [Title]

| Field | Value |
|-------|-------|
| **Severity** | Critical / High / Medium / Low / Info |
| **OWASP Category** | [Category] |
| **CWE** | CWE-XXX |
| **Status** | Open / Fixed / Accepted Risk |

**Description**:
[Detailed description]

**Affected Code**:
```
File: [path:line]

[Code snippet]
```

**Attack Scenario**:
[Exploitation details]

**Remediation**:
[Fix steps]

---

## Severity Definitions

| Severity | Impact | Examples |
|----------|--------|----------|
| **Critical** | Immediate exploitation possible, severe impact | RCE, SQL injection with data access, auth bypass |
| **High** | Significant risk, likely to be exploited | Stored XSS, IDOR, privilege escalation |
| **Medium** | Moderate risk, requires specific conditions | Reflected XSS, CSRF, information disclosure |
| **Low** | Minor risk, limited impact | Missing headers, verbose errors |
| **Info** | Best practice recommendations | Code quality, hardening suggestions |

---

## Scope

**Components Reviewed**:
- [ ] Authentication flows
- [ ] Authorization checks
- [ ] Input validation
- [ ] Output encoding
- [ ] Cryptographic implementation
- [ ] Session management
- [ ] Error handling
- [ ] API security
- [ ] Configuration

**Out of Scope**:
- [Components not reviewed]

---

## Recommendations Summary

| Priority | Finding | Effort | Recommendation |
|----------|---------|--------|----------------|
| 1 | [Critical finding] | Low | [Fix immediately] |
| 2 | [High finding] | Medium | [Fix this sprint] |
| 3 | [Medium finding] | High | [Plan for next release] |

---

## Appendix

### Tools Used
- [Tool 1]
- [Tool 2]

### Test Accounts
- User: [test account used]
- Admin: [admin account used]

### Environment
- [Environment details]

---

*Report generated using OWASP AppSec skill*