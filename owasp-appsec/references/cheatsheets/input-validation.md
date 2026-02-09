# Input Validation Cheatsheet

Validating and sanitizing user input.

## DO / DON'T

| DO | DON'T |
|----|-------|
| Use allowlist validation | Use blocklist validation |
| Validate on server side | Trust client-side validation alone |
| Validate data type, length, format | Only check for specific bad patterns |
| Canonicalize before validating | Validate encoded/escaped input |
| Reject invalid input | Try to "fix" invalid input |
| Validate all input sources | Only validate form fields |

## Validation Strategy

```
# Order of operations:
1. Decode/canonicalize input
2. Validate against allowlist
3. Sanitize if needed for specific context
4. Use the validated data

# Never:
- Validate encoded data (decode first)
- Sanitize then validate (validate first)
- Skip validation for "internal" sources
```

## Type Validation

```
# GOOD: Strong typing with schema validation

schema = {
    "user_id": {"type": "integer", "minimum": 1},
    "email": {"type": "string", "format": "email", "maxLength": 255},
    "age": {"type": "integer", "minimum": 0, "maximum": 150},
    "role": {"type": "string", "enum": ["user", "admin", "moderator"]}
}

function validate(data, schema):
    for field, rules in schema:
        value = data[field]
        
        # Type check
        if not isinstance(value, rules["type"]):
            raise ValidationError(f"{field} must be {rules['type']}")
        
        # Range/length checks
        if "minimum" in rules and value < rules["minimum"]:
            raise ValidationError(f"{field} below minimum")
        
        # Enum check
        if "enum" in rules and value not in rules["enum"]:
            raise ValidationError(f"{field} not in allowed values")
```

## String Validation

```
# Username: alphanumeric + underscore, 3-30 chars
function validate_username(input):
    if not re.match(r'^[a-zA-Z0-9_]{3,30}$', input):
        raise ValidationError("Invalid username")
    return input

# Email: use library validator
function validate_email(input):
    if len(input) > 255:
        raise ValidationError("Email too long")
    if not email_validator.validate(input):
        raise ValidationError("Invalid email")
    return input.lower()  # Normalize

# URL: allowlist domains
function validate_url(input):
    parsed = urlparse(input)
    allowed_domains = ["example.com", "cdn.example.com"]
    
    if parsed.scheme not in ["http", "https"]:
        raise ValidationError("Invalid URL scheme")
    if parsed.hostname not in allowed_domains:
        raise ValidationError("Domain not allowed")
    
    return input
```

## Numeric Validation

```
# Integer with bounds
function validate_quantity(input):
    try:
        value = int(input)
    except:
        raise ValidationError("Must be integer")
    
    if value < 1 or value > 100:
        raise ValidationError("Quantity must be 1-100")
    
    return value

# Price/currency - avoid floating point
function validate_price(input):
    # Store as cents to avoid float issues
    match = re.match(r'^(\d+)\.(\d{2})$', input)
    if not match:
        raise ValidationError("Invalid price format")
    
    cents = int(match[1]) * 100 + int(match[2])
    if cents < 0 or cents > 100000000:  # Max $1M
        raise ValidationError("Price out of range")
    
    return cents
```

## File Upload Validation

```
function validate_file_upload(file):
    # 1. Check file size
    if file.size > 10 * 1024 * 1024:  # 10 MB
        raise ValidationError("File too large")
    
    # 2. Check MIME type (from content, not extension)
    allowed_types = ["image/jpeg", "image/png", "image/gif"]
    detected_type = magic.from_buffer(file.read(2048), mime=True)
    file.seek(0)  # Reset file pointer
    
    if detected_type not in allowed_types:
        raise ValidationError("Invalid file type")
    
    # 3. Sanitize filename
    safe_name = re.sub(r'[^a-zA-Z0-9._-]', '_', file.filename)
    safe_name = safe_name[:100]  # Limit length
    
    # 4. Don't trust extension, generate new name
    extension = {
        "image/jpeg": ".jpg",
        "image/png": ".png",
        "image/gif": ".gif"
    }[detected_type]
    
    new_name = generate_uuid() + extension
    
    return new_name, file
```

## Canonicalization

```
# Decode BEFORE validating

# URL decoding
function validate_path(input):
    # Decode first
    decoded = url_decode(input)
    
    # Then validate
    if ".." in decoded:  # Path traversal
        raise ValidationError("Invalid path")
    
    # Normalize path
    normalized = os.path.normpath(decoded)
    
    # Verify still within allowed directory
    if not normalized.startswith(ALLOWED_BASE):
        raise ValidationError("Path outside allowed area")
    
    return normalized

# Unicode normalization
function validate_text(input):
    # Normalize Unicode to catch homograph attacks
    normalized = unicodedata.normalize('NFKC', input)
    
    # Now validate
    if not re.match(r'^[\w\s]+$', normalized):
        raise ValidationError("Invalid characters")
    
    return normalized
```

## Array/List Validation

```
function validate_ids(input):
    # Validate it's actually an array
    if not isinstance(input, list):
        raise ValidationError("Must be array")
    
    # Limit array size
    if len(input) > 100:
        raise ValidationError("Too many items")
    
    # Validate each element
    validated = []
    for item in input:
        if not isinstance(item, int) or item < 1:
            raise ValidationError("Invalid ID in array")
        validated.append(item)
    
    return validated
```

## Input Sources to Validate

```
# ALL input needs validation:
- request.body         # Form data, JSON
- request.query        # URL parameters  
- request.headers      # Including cookies
- request.path         # URL path segments
- uploaded_files       # File content and metadata
- websocket_messages   # Real-time data
- database_results     # If from untrusted source
- api_responses        # Third-party API data
- environment_vars     # If user-controllable
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Client-only validation | Bypassed via API | Server-side validation required |
| Blocklist approach | Misses new attack patterns | Use allowlist |
| Checking extension only | File type spoofing | Check MIME from content |
| No length limits | DoS via large input | Set max lengths |
| Validating encoded data | Double-encoding bypass | Decode first |

## Related

- `injection-prevention.md` - Using validated input safely
- `xss-prevention.md` - Output encoding after validation