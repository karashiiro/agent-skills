# Error Handling Cheatsheet

Secure error handling and logging.

## DO / DON'T

| DO | DON'T |
|----|-------|
| Return generic error messages | Expose internal error details |
| Log detailed errors server-side | Log sensitive data |
| Use consistent error format | Return different formats per error |
| Implement global error handler | Let exceptions propagate raw |
| Clean up resources on error | Leave connections/files open |
| Use appropriate HTTP status codes | Return 200 for errors |

## User-Facing Errors

```
# WRONG: Exposes internal details
{
    "error": "SQLException: Column 'password' not found in table 'users' at line 42",
    "stack": "at com.example.UserDAO.findByEmail(UserDAO.java:42)\n at com.example..."
}

# WRONG: Helps attackers enumerate
{
    "error": "User not found"  // vs "Invalid password" - different messages!
}

# CORRECT: Generic, consistent
{
    "error": {
        "code": "AUTHENTICATION_FAILED",
        "message": "Invalid credentials",
        "request_id": "abc123"  // For support correlation
    }
}
```

## Error Response Pattern

```
# Standardized error response format

class APIError(Exception):
    def __init__(self, code, message, status_code=400, details=None):
        self.code = code
        self.message = message
        self.status_code = status_code
        self.details = details or {}
        self.request_id = get_request_id()

    def to_response(self):
        return {
            "error": {
                "code": self.code,
                "message": self.message,
                "request_id": self.request_id
            }
        }, self.status_code

# Usage
raise APIError(
    code="VALIDATION_ERROR",
    message="Invalid input",
    status_code=400,
    details={"field": "email", "reason": "Invalid format"}
)

# Common error codes
AUTHENTICATION_FAILED = 401  # Bad credentials
AUTHORIZATION_DENIED = 403   # Not allowed
RESOURCE_NOT_FOUND = 404     # Doesn't exist
VALIDATION_ERROR = 400       # Bad input
RATE_LIMITED = 429           # Too many requests
INTERNAL_ERROR = 500         # Server error (generic)
```

## Global Error Handler

```
# Catch-all to prevent raw exception exposure

def global_error_handler(error):
    request_id = get_request_id()
    
    if isinstance(error, APIError):
        # Known error - return as configured
        log.warn(f"API error: {error.code}", extra={
            "request_id": request_id,
            "code": error.code
        })
        return error.to_response()
    
    elif isinstance(error, ValidationError):
        # Framework validation error
        log.warn(f"Validation error", extra={
            "request_id": request_id,
            "details": str(error)
        })
        return generic_error("VALIDATION_ERROR", 400)
    
    else:
        # Unexpected error - log details, return generic
        log.error(f"Unexpected error: {error}", extra={
            "request_id": request_id,
            "stack": traceback.format_exc()  # Log full stack
        })
        # NEVER expose details to user
        return generic_error("INTERNAL_ERROR", 500)

def generic_error(code, status):
    return {
        "error": {
            "code": code,
            "message": "An error occurred. Please try again.",
            "request_id": get_request_id()
        }
    }, status
```

## Secure Logging

```
# GOOD: Log for debugging without sensitive data
log.info("User login attempt", extra={
    "user_id": user.id,
    "ip": request.ip,
    "user_agent": request.user_agent
})

# BAD: Logging sensitive data
log.info(f"Login: {username}:{password}")    # NEVER log passwords
log.info(f"Card: {card_number}")             # NEVER log card numbers
log.info(f"Token: {session_token}")          # NEVER log auth tokens
log.info(f"SSN: {user.ssn}")                 # NEVER log PII

# Mask sensitive data if needed in context
def mask_card(card):
    return f"****{card[-4:]}"

def mask_email(email):
    parts = email.split('@')
    return f"{parts[0][:2]}***@{parts[1]}"

log.info(f"Payment processed for card {mask_card(card_number)}")
```

## Log Injection Prevention

```
# Attack: Injecting fake log entries or breaking log format
# Input: "admin\nINFO: User admin logged in successfully"

# WRONG: Direct string interpolation
log.info(f"User {username} logged in")  # Log injection possible!

# CORRECT: Use structured logging
log.info("User logged in", extra={"username": username})
# Output: {"message": "User logged in", "username": "admin\nINFO: ..."}
# Newlines are escaped in JSON, injection fails

# If using text logs, sanitize
def sanitize_log_input(value):
    return str(value).replace('\n', '\\n').replace('\r', '\\r')
```

## Resource Cleanup

```
# CORRECT: Clean up in finally block
def process_file(path):
    file = None
    try:
        file = open(path, 'r')
        return process(file.read())
    except IOError as e:
        log.error(f"File error: {e}")
        raise APIError("FILE_ERROR", "Could not process file")
    finally:
        if file:
            file.close()  # Always close, even on error

# BETTER: Use context managers
def process_file(path):
    try:
        with open(path, 'r') as file:  # Automatic cleanup
            return process(file.read())
    except IOError as e:
        log.error(f"File error: {e}")
        raise APIError("FILE_ERROR", "Could not process file")

# Database connections
def query_user(user_id):
    connection = None
    try:
        connection = db.get_connection()
        return connection.query("SELECT * FROM users WHERE id = ?", user_id)
    finally:
        if connection:
            connection.close()  # Return to pool
```

## Timing Attack Prevention

```
# WRONG: Different response times reveal information
def login(username, password):
    user = find_user(username)
    if not user:
        return error("Invalid")  # Fast - no hash check
    
    if not verify_password(user.password_hash, password):
        return error("Invalid")  # Slow - hash verification

# CORRECT: Constant-time response
def login(username, password):
    user = find_user(username)
    
    if not user:
        # Do dummy hash to match timing
        verify_password(DUMMY_HASH, password)
        return error("Invalid credentials")
    
    if not verify_password(user.password_hash, password):
        return error("Invalid credentials")  # Same message!
    
    return success()
```

## HTTP Status Codes

```
# Use appropriate status codes

200 OK              # Success
201 Created         # Resource created
204 No Content      # Success, no body (DELETE)

400 Bad Request     # Invalid input
401 Unauthorized    # Authentication required
403 Forbidden       # Authenticated but not authorized
404 Not Found       # Resource doesn't exist
409 Conflict        # State conflict (duplicate, etc.)
422 Unprocessable   # Validation failed
429 Too Many        # Rate limited

500 Internal Error  # Server error (generic)
502 Bad Gateway     # Upstream service failed
503 Unavailable    # Temporarily unavailable
504 Gateway Timeout # Upstream timeout

# Avoid information leakage via status codes
# BAD: 403 for valid user, 404 for invalid user
# GOOD: Consistent 401/403 regardless of user existence
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Stack traces to users | Exposes internals | Generic errors + server logs |
| Different error for valid/invalid user | User enumeration | Consistent error messages |
| Logging passwords/tokens | Credential leak | Never log secrets |
| No request ID | Hard to debug | Include correlation ID |
| Empty catch blocks | Silent failures | Log or rethrow |

## Related

- `secrets-management.md` - Not exposing secrets in errors
- `authentication.md` - Login error handling
- `input-validation.md` - Validation error messages