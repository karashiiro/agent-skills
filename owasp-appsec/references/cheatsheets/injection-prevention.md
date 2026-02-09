# Injection Prevention Cheatsheet

Preventing SQL, command, and other injection attacks.

## DO / DON'T

| DO | DON'T |
|----|-------|
| Use parameterized queries | Concatenate strings into queries |
| Use ORM/query builders | Build raw SQL strings |
| Allowlist input validation | Blocklist known bad patterns |
| Escape for specific context | Use generic escape function |
| Use prepared statements | Trust escaped input in queries |
| Avoid system commands | Pass user input to shell |

## SQL Injection

### Parameterized Queries (CORRECT)

```
# SECURE: Parameters are separate from query structure

# Standard SQL
query = "SELECT * FROM users WHERE id = ? AND status = ?"
result = db.execute(query, [user_id, status])

# Named parameters
query = "SELECT * FROM users WHERE id = :id AND status = :status"
result = db.execute(query, {"id": user_id, "status": status})

# ORM (always parameterized)
user = User.find_by(id: user_id, status: status)
```

### String Concatenation (WRONG)

```
# INSECURE: User input becomes part of query structure

# Direct concatenation
query = "SELECT * FROM users WHERE id = " + user_id  # VULNERABLE
query = f"SELECT * FROM users WHERE id = {user_id}"   # VULNERABLE
query = "SELECT * FROM users WHERE id = '" + user_id + "'"  # VULNERABLE

# Even with "escaping"
escaped = escape_sql(user_id)
query = f"SELECT * FROM users WHERE id = {escaped}"  # STILL RISKY
```

### Dynamic Query Construction

```
# SECURE: Build query safely with validated parts

function build_search_query(filters):
    base_query = "SELECT * FROM products WHERE 1=1"
    params = []
    
    # Allowlist columns that can be filtered
    allowed_columns = ["name", "category", "price"]
    
    for column, value in filters:
        if column not in allowed_columns:
            raise ValidationError(f"Invalid filter: {column}")
        base_query += f" AND {column} = ?"  # Column from allowlist, safe
        params.append(value)                 # Value as parameter, safe
    
    # Safe ORDER BY
    allowed_order = ["name", "price", "created_at"]
    if filters.order_by in allowed_order:
        direction = "DESC" if filters.desc else "ASC"
        base_query += f" ORDER BY {filters.order_by} {direction}"
    
    return db.execute(base_query, params)
```

## Command Injection

### Avoid Shell Commands

```
# BEST: Use language APIs instead of shell

# BAD: Shell command
os.system(f"convert {input_file} {output_file}")  # VULNERABLE

# GOOD: Use library
from PIL import Image
img = Image.open(input_file)
img.save(output_file)

# BAD: Shell for file operations
os.system(f"rm -rf {user_path}")  # VULNERABLE

# GOOD: Use filesystem APIs
import shutil
shutil.rmtree(validated_path)
```

### If Shell Required

```
# If you MUST use shell commands:

# 1. Use array form (no shell interpretation)
subprocess.run(["convert", input_file, output_file])  # Safe

# 2. Never use shell=True with user input
subprocess.run(f"convert {input}", shell=True)  # VULNERABLE

# 3. Validate against strict allowlist
allowed_formats = ["png", "jpg", "gif"]
if format not in allowed_formats:
    raise ValidationError()

# 4. Use shlex.quote for unavoidable cases
from shlex import quote
cmd = f"echo {quote(user_input)}"  # Last resort
```

## LDAP Injection

```
# INSECURE
filter = f"(&(uid={username})(password={password}))"  # VULNERABLE
# Attack: username = "*)(uid=*))(|(uid=*"

# SECURE: Escape special characters
function ldap_escape(input):
    special_chars = ['\\', '*', '(', ')', '\0', '/']
    for char in special_chars:
        input = input.replace(char, '\\' + hex(ord(char)))
    return input

filter = f"(&(uid={ldap_escape(username)})(password={ldap_escape(password)}))"
```

## XPath Injection

```
# INSECURE
query = f"//users/user[name='{username}' and password='{password}']"
# Attack: username = "' or '1'='1"

# SECURE: Use parameterized XPath (if available) or validate
function xpath_escape(input):
    if "'" in input and '"' in input:
        # Use concat() for mixed quotes
        return "concat('" + input.replace("'", "',\"'\",'") + "')"
    elif "'" in input:
        return '"' + input + '"'
    else:
        return "'" + input + "'"
```

## Template Injection

```
# INSECURE: User input in template
template = f"Hello {user_input}!"
render(template)  # If template engine processes user_input

# Attack: user_input = "{{7*7}}" or "${7*7}" or "<%= 7*7 %>"

# SECURE: Pass as variable, not template content
template = "Hello {{name}}!"
render(template, {"name": user_input})  # Safe - user_input is data, not code
```

## NoSQL Injection

```
# MongoDB INSECURE
db.users.find({"username": username, "password": password})
# Attack: password = {"$ne": ""}

# SECURE: Validate types
if not isinstance(username, str) or not isinstance(password, str):
    raise ValidationError("Invalid input type")

# Or use schema validation
from pydantic import BaseModel

class LoginRequest(BaseModel):
    username: str
    password: str

request = LoginRequest(**request_data)  # Validates types
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Escaping instead of parameterizing | Bypass via encoding tricks | Always use parameterized queries |
| Allowlisting after query built | Injection already occurred | Validate before query construction |
| shell=True with subprocess | Command injection | Use array form without shell |
| Trusting ORM completely | Raw queries still vulnerable | Review any raw SQL in ORM |
| JSON.parse without validation | NoSQL injection | Validate types after parsing |

## Related

- `input-validation.md` - Validating input before use
- `xss-prevention.md` - Output encoding for HTML context