# Secrets Management Cheatsheet

Securely handling API keys, passwords, and credentials.

## DO / DON'T

| DO | DON'T |
|----|-------|
| Use environment variables | Hardcode secrets in code |
| Use secret managers (Vault, AWS SM) | Store secrets in config files |
| Rotate secrets regularly | Use same secret forever |
| Audit secret access | Share secrets freely |
| Use service accounts | Use personal credentials in apps |
| Encrypt secrets at rest | Store secrets in plaintext |
| Use separate secrets per environment | Share prod secrets with dev |

## Secret Types

```
# Types of secrets to protect:
- API keys (third-party services)
- Database credentials
- Encryption keys
- OAuth client secrets
- JWT signing keys
- SSH keys
- TLS certificates/keys
- Service account tokens
```

## Environment Variables

```
# GOOD: Load from environment
database_url = os.environ['DATABASE_URL']
api_key = os.environ['STRIPE_API_KEY']

# BAD: Hardcoded
database_url = "postgres://user:password@localhost/db"  # NEVER DO THIS
api_key = "sk_live_abc123"  # LEAKED IF COMMITTED

# Required vs optional with defaults
database_url = os.environ['DATABASE_URL']  # Required - fail if missing
debug_mode = os.environ.get('DEBUG', 'false')  # Optional with default
```

## Secret Managers

```
# AWS Secrets Manager
import boto3

def get_secret(secret_name):
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_name)
    return response['SecretString']

db_password = get_secret('prod/database/password')

# HashiCorp Vault
import hvac

client = hvac.Client(url='https://vault.example.com')
client.auth.kubernetes.login(role='myapp')  # K8s auth
secret = client.secrets.kv.v2.read_secret_version(path='myapp/database')
db_password = secret['data']['data']['password']

# Azure Key Vault
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

client = SecretClient(
    vault_url="https://mykeyvault.vault.azure.net/",
    credential=DefaultAzureCredential()
)
db_password = client.get_secret("database-password").value
```

## Secret Rotation

```
# Design for rotation from the start

# Support multiple active keys during rotation
api_keys = [
    os.environ['API_KEY_CURRENT'],
    os.environ.get('API_KEY_PREVIOUS')  # Optional, for rotation period
]

def validate_api_key(provided_key):
    return provided_key in api_keys and provided_key is not None

# Database credential rotation
# 1. Create new credentials in secret manager
# 2. App reads new credentials on next refresh
# 3. Old credentials disabled after grace period

# JWT key rotation
jwt_keys = {
    'current': os.environ['JWT_KEY_CURRENT'],
    'previous': os.environ.get('JWT_KEY_PREVIOUS')
}

def sign_jwt(payload):
    return jwt.sign(payload, jwt_keys['current'], kid='current')

def verify_jwt(token):
    header = jwt.get_unverified_header(token)
    key = jwt_keys.get(header['kid'])
    return jwt.verify(token, key)
```

## Configuration Files

```
# GOOD: Config with placeholders, secrets from env
# config.yaml
database:
  host: ${DATABASE_HOST}
  port: 5432
  password: ${DATABASE_PASSWORD}  # From environment

# BAD: Secrets in config file
# config.yaml
database:
  password: "actual_password_here"  # NEVER DO THIS

# .gitignore - ALWAYS ignore secret files
.env
.env.*
*.pem
*.key
secrets/
config/production.yaml
```

## Application Patterns

```
# Initialize secrets at startup
class Config:
    def __init__(self):
        self.database_url = self._get_required('DATABASE_URL')
        self.api_key = self._get_required('API_KEY')
        self.debug = self._get_optional('DEBUG', 'false') == 'true'
    
    def _get_required(self, name):
        value = os.environ.get(name)
        if not value:
            raise ConfigError(f"Missing required config: {name}")
        return value
    
    def _get_optional(self, name, default):
        return os.environ.get(name, default)

# Fail fast if secrets missing
config = Config()  # Fails at startup if missing

# Don't log secrets
logger.info(f"Connecting to {config.database_url}")  # BAD - logs password!
logger.info(f"Connecting to database")  # GOOD - no sensitive data
```

## Preventing Leaks

```
# Pre-commit hooks to catch secrets
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']

# Mask secrets in logs
class SecretString:
    def __init__(self, value):
        self._value = value
    
    def __str__(self):
        return "[REDACTED]"
    
    def __repr__(self):
        return "SecretString([REDACTED])"
    
    def get_value(self):
        return self._value

api_key = SecretString(os.environ['API_KEY'])
print(api_key)  # Prints: [REDACTED]

# Clear sensitive data from memory
def process_password(password):
    try:
        hash = hash_password(password)
        return hash
    finally:
        # Clear password from memory (best effort)
        password = '\x00' * len(password)
```

## CI/CD Secrets

```
# GitHub Actions
# Settings -> Secrets -> Actions
steps:
  - name: Deploy
    env:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
      API_KEY: ${{ secrets.API_KEY }}
    run: ./deploy.sh

# GitLab CI
# Settings -> CI/CD -> Variables (masked, protected)
deploy:
  script:
    - echo "Deploying..."
  variables:
    DATABASE_URL: $DATABASE_URL  # From CI/CD variables
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Secrets in git history | Permanent leak | Rotate and remove from history |
| Same secret in all envs | Prod leak via dev | Separate secrets per environment |
| Secrets in error messages | Logged/displayed | Never include secrets in errors |
| No rotation capability | Can't respond to leaks | Design for rotation from start |
| Secrets in URLs | Logged by proxies/servers | Use headers or body |

## Related

- `authentication.md` - Credential handling
- `cryptographic-storage.md` - Encryption keys
- `error-handling.md` - Not leaking secrets in errors