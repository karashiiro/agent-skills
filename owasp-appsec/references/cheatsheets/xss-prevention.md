# XSS Prevention Cheatsheet

Preventing Cross-Site Scripting attacks.

## DO / DON'T

| DO | DON'T |
|----|-------|
| Encode output for context | Insert user input raw |
| Use framework auto-escaping | Disable escaping for convenience |
| Implement Content Security Policy | Rely on encoding alone |
| Use textContent for DOM | Use innerHTML with user data |
| Sanitize HTML if needed | Allow any HTML tags |
| HttpOnly cookies for sessions | Store tokens in localStorage |

## XSS Types

```
# Reflected XSS: Input echoed immediately
GET /search?q=<script>alert(1)</script>
Response: Results for: <script>alert(1)</script>

# Stored XSS: Input saved, shown to others
POST /comment {"text": "<script>steal(cookies)</script>"}
Later: All users see the malicious script

# DOM XSS: Client-side code uses untrusted data
element.innerHTML = location.hash.substr(1)  // Vulnerable
```

## Context-Specific Encoding

```
# Different contexts require different encoding!

# HTML Body
<div>USER_INPUT</div>
Encode: < > & " '
Result: &lt;script&gt; becomes harmless text

# HTML Attribute
<input value="USER_INPUT">
Encode: " & (and use quotes!)
Risk: Breaking out of attribute

# JavaScript String
<script>var x = "USER_INPUT";</script>
Encode: \ " ' newlines
Risk: Breaking out of string

# URL Parameter
<a href="/search?q=USER_INPUT">
Encode: URL-encode special characters
Risk: javascript: URLs, breaking URL structure

# CSS Value
<div style="background: USER_INPUT">
Encode: CSS escape or avoid entirely
Risk: expression(), url() attacks
```

## HTML Context Encoding

```
function html_encode(input):
    return input
        .replace('&', '&amp;')
        .replace('<', '&lt;')
        .replace('>', '&gt;')
        .replace('"', '&quot;')
        .replace("'", '&#x27;')

# Usage in templates
<p>Hello, {html_encode(username)}!</p>

# Most frameworks do this automatically:
# React: {username} - auto-escaped
# Vue: {{ username }} - auto-escaped
# Jinja2: {{ username }} - auto-escaped by default
# Django: {{ username }} - auto-escaped by default
```

## JavaScript Context

```
# DANGEROUS: Embedding user data in JavaScript

# WRONG
<script>
var userData = "{user_input}";  // Can break out!
</script>

# SAFER: JSON encode
<script>
var userData = JSON.parse('{json_encode(user_data)}');
</script>

# BEST: Use data attributes
<div id="container" data-user="{html_encode(json_encode(user_data))}">
</div>
<script>
var userData = JSON.parse(document.getElementById('container').dataset.user);
</script>
```

## DOM-Based XSS Prevention

```
# DANGEROUS: These sink functions execute code
element.innerHTML = userInput      // Executes script tags
element.outerHTML = userInput      // Executes script tags
document.write(userInput)          // Executes scripts
element.insertAdjacentHTML(userInput)
eval(userInput)
setTimeout(userInput)
setInterval(userInput)
new Function(userInput)

# SAFE: These don't execute code
element.textContent = userInput    // Safe - text only
element.innerText = userInput      // Safe - text only
element.setAttribute('data-x', userInput)  // Safe if not event handler

# For HTML content, use sanitizer
const clean = DOMPurify.sanitize(userInput);
element.innerHTML = clean;
```

## URL Context

```
# DANGEROUS: User-controlled URLs
<a href="{user_url}">Click</a>

# Attack: javascript:alert(document.cookie)

# SAFE: Validate URL scheme
function safe_url(url):
    parsed = urlparse(url)
    if parsed.scheme not in ['http', 'https']:
        return '#'  # Invalid, use safe default
    return url_encode_path(url)

# For same-origin paths
function safe_path(path):
    if path.startswith('/') and not path.startswith('//'):
        return path
    return '/'  # Safe default
```

## Content Security Policy (CSP)

```
# Defense in depth - blocks inline scripts even if XSS exists

# Strict CSP (recommended)
Content-Security-Policy: 
    default-src 'self';
    script-src 'self' 'nonce-{random}';
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: https:;
    font-src 'self';
    connect-src 'self' api.example.com;
    frame-ancestors 'none';
    form-action 'self';
    base-uri 'self';

# Using nonces
<script nonce="{random_nonce}">  // Allowed
    // Your code here
</script>
<script>alert('xss')</script>     // Blocked - no nonce

# Report violations
Content-Security-Policy-Report-Only: ...; report-uri /csp-report
```

## HTML Sanitization

```
# When you need to allow SOME HTML (rich text editors)

# Use a proven sanitizer library
allowed_tags = ['p', 'br', 'b', 'i', 'u', 'a', 'ul', 'ol', 'li']
allowed_attrs = {
    'a': ['href', 'title'],  # Only allow safe attributes
}

clean_html = sanitizer.clean(
    user_html,
    tags=allowed_tags,
    attributes=allowed_attrs,
    strip=True
)

# DOMPurify (JavaScript)
var clean = DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['p', 'br', 'b', 'i', 'a'],
    ALLOWED_ATTR: ['href']
});
```

## Framework-Specific

```
# React - safe by default
<div>{userInput}</div>  // Auto-escaped

# DANGER: dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{__html: userInput}} />  // XSS risk!
# Only use with DOMPurify.sanitize()

# Vue - safe by default
<div>{{ userInput }}</div>  // Auto-escaped

# DANGER: v-html
<div v-html="userInput"></div>  // XSS risk!

# Angular - safe by default
<div>{{ userInput }}</div>  // Auto-escaped

# DANGER: [innerHTML]
<div [innerHTML]="userInput"></div>  // Sanitized but risky
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Disabling auto-escape | XSS in templates | Keep escaping enabled |
| innerHTML with user data | DOM XSS | Use textContent or sanitize |
| No CSP | No defense in depth | Implement strict CSP |
| Trusting URL schemes | javascript: XSS | Validate http/https only |
| LocalStorage for tokens | XSS steals tokens | Use httpOnly cookies |

## Related

- `input-validation.md` - Validating input before use
- `session-management.md` - Secure token storage