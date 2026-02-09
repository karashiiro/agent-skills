# Authorization Cheatsheet

Secure implementation patterns for access control.

## DO / DON'T

| DO | DON'T |
|----|-------|
| Deny by default | Allow by default |
| Check authorization on every request | Check only on frontend |
| Verify ownership of resources | Trust user-provided IDs |
| Use centralized authorization logic | Scatter checks throughout code |
| Log authorization failures | Silently deny access |
| Re-verify on sensitive operations | Cache authorization decisions long-term |
| Use unpredictable resource IDs | Use sequential/guessable IDs |

## Authorization Models

### Role-Based Access Control (RBAC)

```
# Define roles and permissions
roles = {
    "admin": ["read", "write", "delete", "manage_users"],
    "editor": ["read", "write"],
    "viewer": ["read"]
}

function check_permission(user, action, resource):
    user_role = get_role(user)
    allowed_actions = roles[user_role]
    
    if action not in allowed_actions:
        log_authz_failure(user, action, resource)
        return deny()
    
    return allow()
```

### Attribute-Based Access Control (ABAC)

```
# More flexible - based on attributes
function check_permission(user, action, resource):
    # Check multiple attributes
    if action == "edit_document":
        return (
            resource.owner == user.id or
            resource.department == user.department and user.role == "editor" or
            user.role == "admin"
        )
    
    return deny()
```

### Resource Ownership

```
# ALWAYS verify ownership
function get_user_document(user_id, document_id):
    document = db.find(document_id)
    
    # CRITICAL: Check ownership
    if document.owner_id != user_id:
        log_authz_failure(user_id, "read", document_id)
        raise AuthorizationError("Access denied")
    
    return document
```

## Implementation Patterns

### Middleware Pattern

```
# Centralized authorization middleware
function authz_middleware(request, required_permission):
    user = get_current_user(request)
    resource = get_resource_from_request(request)
    
    if not has_permission(user, required_permission, resource):
        log.warn(f"AuthZ denied: user={user.id}, perm={required_permission}, resource={resource.id}")
        return response(403, "Forbidden")
    
    return continue_to_handler()

# Usage
@route("/documents/{id}", methods=["DELETE"])
@require_permission("delete_document")
function delete_document(id):
    # Authorization already verified by middleware
    ...
```

### Query-Level Authorization

```
# Filter at query level - more efficient than post-filter
function get_user_documents(user):
    # GOOD: Filter in query
    return db.query(
        "SELECT * FROM documents WHERE owner_id = ? OR shared_with @> ?",
        user.id, [user.id]
    )

# BAD: Fetch all, filter in code
function get_user_documents_INSECURE(user):
    all_docs = db.query("SELECT * FROM documents")
    return filter(doc => doc.owner_id == user.id, all_docs)
    # Problem: Loads all docs, may leak via timing/errors
```

### Indirect Reference Maps

```
# Map user-facing IDs to internal IDs
# Prevents direct object reference attacks

function create_reference_map(user, resources):
    map = {}
    for resource in resources:
        # Generate random reference per user session
        ref = generate_random_id()
        map[ref] = resource.id
    session["reference_map"] = map
    return map

function resolve_reference(user, ref):
    map = session["reference_map"]
    if ref not in map:
        raise AuthorizationError("Invalid reference")
    return map[ref]
```

## Hierarchical Authorization

```
# Organization -> Team -> Project -> Resource
function check_hierarchical_access(user, resource):
    # Check at each level
    project = get_project(resource.project_id)
    team = get_team(project.team_id)
    org = get_org(team.org_id)
    
    # User must have access at some level
    return (
        user.id in org.admins or
        user.id in team.members or
        user.id in project.members or
        user.id == resource.owner_id
    )
```

## API Authorization

```
# REST endpoint authorization

# GET /users/{id} - users can only get their own data
@route("/users/{id}")
function get_user(request, id):
    if request.user.id != id and not request.user.is_admin:
        return response(403)
    ...

# GET /users/{id}/documents - only owner or admin
@route("/users/{id}/documents")
function get_user_documents(request, id):
    if request.user.id != id and not request.user.is_admin:
        return response(403)
    ...

# POST /documents - any authenticated user
@route("/documents", methods=["POST"])
function create_document(request):
    # Set owner to current user, not from request!
    document.owner_id = request.user.id  # NOT request.body.owner_id
    ...
```

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Trusting client-sent user ID | IDOR attacks | Get user ID from session only |
| Frontend-only authorization | Bypassed via API calls | Always check server-side |
| Sequential IDs | Enumeration attacks | Use UUIDs or random IDs |
| Missing authz on nested resources | Access via parent bypass | Check at every level |
| Caching authz decisions too long | Stale permissions | Short TTL or event-based invalidation |

## Related

- `authentication.md` - Verifying user identity
- `session-management.md` - Maintaining auth state
- `input-validation.md` - Validating resource IDs