# Session Settings

## Introduction

Django provides various settings to configure session behavior globally in your `settings.py` file. These settings control session expiration, cookie attributes, storage backends, and security options. Understanding these settings allows you to customize session behavior to match your application's security requirements and performance needs without modifying individual views.

## Concept Explanation

### SESSION_COOKIE_AGE

Controls the default session expiration time in seconds.

**Default Value:** `1209600` (2 weeks in seconds)

**Purpose:**
- Sets how long sessions remain valid
- Applies to all sessions unless overridden with `set_expiry()`
- Affects the `sessionid` cookie's expiration

**Example:**
```python
# settings.py
SESSION_COOKIE_AGE = 3600  # 1 hour
SESSION_COOKIE_AGE = 86400  # 1 day
SESSION_COOKIE_AGE = 604800  # 1 week
```

**When to Change:**
- Shorter expiration for sensitive applications
- Longer expiration for user convenience
- Compliance with security policies

### SESSION_COOKIE_NAME

Customizes the name of the session ID cookie.

**Default Value:** `'sessionid'`

**Purpose:**
- Change cookie name for branding or security
- Avoid conflicts with other applications
- Obfuscate session cookie name (security by obscurity)

**Example:**
```python
# settings.py
SESSION_COOKIE_NAME = 'myapp_session'
SESSION_COOKIE_NAME = 'sid'
SESSION_COOKIE_NAME = 'app_session_id'
```

**Note:**
- Changing this affects all sessions
- Existing sessions with old name become invalid
- Choose a name and stick with it

### SESSION_COOKIE_HTTPONLY

Controls whether the session cookie is accessible via JavaScript.

**Default Value:** `True`

**Purpose:**
- Prevents JavaScript from accessing session cookie
- Protects against XSS (Cross-Site Scripting) attacks
- Security best practice

**When True (Recommended):**
- JavaScript cannot read `document.cookie` for session
- Protects session from malicious scripts
- Industry standard for security

**When False (Not Recommended):**
- JavaScript can access session cookie
- Vulnerable to XSS attacks
- Only for specific legacy requirements

**Example:**
```python
# settings.py
SESSION_COOKIE_HTTPONLY = True  # Recommended
SESSION_COOKIE_HTTPONLY = False  # Not recommended
```

### SESSION_COOKIE_SECURE

Controls whether session cookie is only sent over HTTPS.

**Default Value:** `False`

**Purpose:**
- Ensures session cookie only transmitted over secure connections
- Prevents cookie interception over HTTP
- Essential for production security

**When True (Production):**
- Cookie only sent over HTTPS
- Prevents man-in-the-middle attacks
- Required for secure applications

**When False (Development):**
- Cookie sent over HTTP and HTTPS
- Acceptable for local development
- Never use in production

**Example:**
```python
# settings.py
import os

SESSION_COOKIE_SECURE = os.environ.get('DJANGO_ENV') == 'production'
```

### SESSION_COOKIE_SAMESITE

Controls the SameSite attribute for session cookie.

**Default Value:** `'Lax'`

**Purpose:**
- Prevents CSRF (Cross-Site Request Forgery) attacks
- Controls when cookies are sent with cross-site requests

**Values:**
- `'Strict'`: Cookie only sent for same-site requests
- `'Lax'`: Cookie sent for same-site and top-level navigations
- `'None'`: Cookie sent with all requests (requires `Secure`)

**Example:**
```python
# settings.py
SESSION_COOKIE_SAMESITE = 'Strict'  # Most secure
SESSION_COOKIE_SAMESITE = 'Lax'     # Default
SESSION_COOKIE_SAMESITE = 'None'    # Requires Secure=True
```

### SESSION_EXPIRE_AT_BROWSER_CLOSE

Controls whether sessions expire when the browser closes.

**Default Value:** `False`

**Purpose:**
- Alternative to time-based expiration
- Useful for temporary sessions
- Enhances security for sensitive applications

**When True:**
- Session expires when browser closes
- Session cookie has no expiration date
- Overrides `SESSION_COOKIE_AGE`

**When False (Default):**
- Session expires after `SESSION_COOKIE_AGE` seconds
- Session persists across browser restarts
- Standard behavior for most applications

**Example:**
```python
# settings.py
SESSION_EXPIRE_AT_BROWSER_CLOSE = True  # Expire on browser close
SESSION_EXPIRE_AT_BROWSER_CLOSE = False  # Use time-based expiration
```

### SESSION_ENGINE

Controls the session storage backend.

**Default Value:** `'django.contrib.sessions.backends.db'`

**Purpose:**
- Choose where session data is stored
- Affects performance and scalability
- Different backends for different use cases

**Available Backends:**
- `'django.contrib.sessions.backends.db'` - Database (default)
- `'django.contrib.sessions.backends.cache'` - Cache
- `'django.contrib.sessions.backends.file'` - File system
- `'django.contrib.sessions.backends.signed_cookies'` - Signed cookies

**Example:**
```python
# settings.py
# Database-backed (default)
SESSION_ENGINE = 'django.contrib.sessions.backends.db'

# Cache-backed (faster)
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

# File-backed (not recommended for production)
SESSION_ENGINE = 'django.contrib.sessions.backends.file'
SESSION_FILE_PATH = '/var/tmp/django_sessions'
```

### SESSION_SAVE_EVERY_REQUEST

Controls whether session is saved on every request.

**Default Value:** `False`

**Purpose:**
- Force session save even if not modified
- Ensures session expiration is updated on each request
- Useful for sliding session expiration

**When True:**
- Session saved on every request
- Session expiration refreshed on each request
- Performance overhead on every request

**When False (Default):**
- Session saved only when modified
- Better performance
- Standard behavior

**Example:**
```python
# settings.py
SESSION_SAVE_EVERY_REQUEST = True  # Save on every request
SESSION_SAVE_EVERY_REQUEST = False  # Save only when modified
```

## Code Examples

### Basic Session Configuration

**settings.py**
```python
# Session cookie settings
SESSION_COOKIE_AGE = 3600  # 1 hour expiration
SESSION_COOKIE_NAME = 'myapp_session'
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SECURE = False  # Set to True in production
SESSION_COOKIE_SAMESITE = 'Lax'
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
```

### Production Session Configuration

**settings.py**
```python
import os

# Production session settings
SESSION_COOKIE_AGE = 86400  # 1 day
SESSION_COOKIE_NAME = 'app_session_id'
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SECURE = os.environ.get('DJANGO_ENV') == 'production'
SESSION_COOKIE_SAMESITE = 'Strict'
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
SESSION_SAVE_EVERY_REQUEST = False
```

### Browser-Close Session Configuration

**settings.py**
```python
# Expire sessions when browser closes
SESSION_EXPIRE_AT_BROWSER_CLOSE = True
SESSION_COOKIE_AGE = None  # Not used when browser-close is True
```

### Cache-Backed Session Configuration

**settings.py**
```python
# Cache configuration
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',
    }
}

# Use cache for sessions
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_COOKIE_AGE = 3600
```

### Development vs Production Settings

**settings.py**
```python
import os

# Development settings
DEBUG = True

if DEBUG:
    # Development session settings
    SESSION_COOKIE_SECURE = False
    SESSION_COOKIE_SAMESITE = 'Lax'
    SESSION_EXPIRE_AT_BROWSER_CLOSE = False
else:
    # Production session settings
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_SAMESITE = 'Strict'
    SESSION_EXPIRE_AT_BROWSER_CLOSE = False
    SESSION_COOKIE_AGE = 3600  # 1 hour in production
```

### Short-Lived Session Configuration

**settings.py**
```python
# Short-lived sessions for sensitive applications
SESSION_COOKIE_AGE = 1800  # 30 minutes
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_SAMESITE = 'Strict'
SESSION_SAVE_EVERY_REQUEST = True  # Refresh on each request
```

### Custom Cookie Name Configuration

**settings.py**
```python
# Custom session cookie name for branding
SESSION_COOKIE_NAME = 'myapp_sid'
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SECURE = True
```

### Sliding Session Expiration

**settings.py**
```python
# Sliding expiration - refresh on each request
SESSION_COOKIE_AGE = 3600  # 1 hour
SESSION_SAVE_EVERY_REQUEST = True  # Refreshes expiration
```

### File-Based Session Configuration

**settings.py**
```python
# File-based session storage (not recommended for production)
SESSION_ENGINE = 'django.contrib.sessions.backends.file'
SESSION_FILE_PATH = '/var/tmp/django_sessions'
SESSION_COOKIE_AGE = 3600
```

### Signed Cookie Session Configuration

**settings.py**
```python
# Store session data in signed cookies (client-side)
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'
SESSION_COOKIE_AGE = 3600
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SECURE = True

# Note: Limited to 4KB due to cookie size limit
```

## Key Takeaways

- `SESSION_COOKIE_AGE` sets default session expiration in seconds
- `SESSION_COOKIE_NAME` customizes the session cookie name
- `SESSION_COOKIE_HTTPONLY` prevents JavaScript access (security)
- `SESSION_COOKIE_SECURE` ensures HTTPS-only transmission
- `SESSION_COOKIE_SAMESITE` controls CSRF protection
- `SESSION_EXPIRE_AT_BROWSER_CLOSE` expires sessions on browser close
- `SESSION_ENGINE` chooses session storage backend
- `SESSION_SAVE_EVERY_REQUEST` forces session save on each request
- Always use `HTTPONLY=True` for security
- Use `SECURE=True` in production environments

## Additional Context & Best Practices

### Security Best Practices

**1. Always Use HTTPOnly**
```python
# ✅ CORRECT - Prevents XSS attacks
SESSION_COOKIE_HTTPONLY = True

# ❌ WRONG - Vulnerable to XSS
SESSION_COOKIE_HTTPONLY = False
```

**2. Use Secure in Production**
```python
# ✅ CORRECT - Environment-based configuration
import os
SESSION_COOKIE_SECURE = os.environ.get('DJANGO_ENV') == 'production'

# ❌ WRONG - Always insecure
SESSION_COOKIE_SECURE = False
```

**3. Use SameSite for CSRF Protection**
```python
# ✅ CORRECT - Strict SameSite for sensitive apps
SESSION_COOKIE_SAMESITE = 'Strict'

# ✅ GOOD - Lax SameSite for most apps
SESSION_COOKIE_SAMESITE = 'Lax'
```

**4. Set Appropriate Expiration**
```python
# ✅ GOOD - Shorter expiration for sensitive data
SESSION_COOKIE_AGE = 3600  # 1 hour

# ❌ BAD - Too long for sensitive applications
SESSION_COOKIE_AGE = 31536000  # 1 year
```

### Performance Considerations

**1. Choose Right Backend**
```python
# ✅ GOOD - Cache-backed for high traffic
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

# ✅ GOOD - Database-backed for most applications
SESSION_ENGINE = 'django.contrib.sessions.backends.db'

# ❌ BAD - File-backed for production
SESSION_ENGINE = 'django.contrib.sessions.backends.file'
```

**2. Avoid Unnecessary Saves**
```python
# ✅ GOOD - Save only when modified
SESSION_SAVE_EVERY_REQUEST = False

# ❌ BAD - Save on every request (performance overhead)
SESSION_SAVE_EVERY_REQUEST = True
```

**3. Monitor Session Table Size**
```python
# Regular cleanup of expired sessions
from django.contrib.sessions.backends.db import SessionStore
SessionStore.clear_expired()
```

### Common Pitfalls

**1. Forgetting Secure in Production**
```python
# ❌ WRONG - Insecure in production
SESSION_COOKIE_SECURE = False

# ✅ CORRECT - Environment-based
SESSION_COOKIE_SECURE = DEBUG is False
```

**2. Changing Cookie Name After Deployment**
```python
# ❌ WRONG - Changes break existing sessions
SESSION_COOKIE_NAME = 'new_name'  # After deployment

# ✅ CORRECT - Choose name and stick with it
SESSION_COOKIE_NAME = 'app_session'  # Set once, never change
```

**3. Using File Backend in Production**
```python
# ❌ WRONG - Not scalable for production
SESSION_ENGINE = 'django.contrib.sessions.backends.file'

# ✅ CORRECT - Use database or cache
SESSION_ENGINE = 'django.contrib.sessions.backends.db'
```

**4. Setting Both Browser Close and Age**
```python
# ❌ WRONG - Conflicting settings
SESSION_EXPIRE_AT_BROWSER_CLOSE = True
SESSION_COOKIE_AGE = 3600  # Ignored when browser-close is True

# ✅ CORRECT - Choose one approach
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
SESSION_COOKIE_AGE = 3600
```

### Advanced Configuration

**1. Multiple Session Engines**
```python
# Use different engines for different purposes
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

# Fallback to database if cache fails
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}
```

**2. Custom Session Cookie Path**
```python
# Limit session cookie to specific path
SESSION_COOKIE_PATH = '/app/'
```

**3. Custom Session Cookie Domain**
```python
# Share session across subdomains
SESSION_COOKIE_DOMAIN = '.example.com'
```

**4. Session Serialization**
```python
# Ensure session data is JSON-serializable
SESSION_SERIALIZER = 'django.contrib.sessions.serializers.JSONSerializer'
```

## Practice Exercises

### Exercise 1: Set Short Session Expiration

Configure sessions to expire after 30 minutes.

<details>
<summary>Solution</summary>

```python
# settings.py
SESSION_COOKIE_AGE = 1800  # 30 minutes in seconds
```
</details>

### Exercise 2: Customize Session Cookie Name

Change the session cookie name to `myapp_sid`.

<details>
<summary>Solution</summary>

```python
# settings.py
SESSION_COOKIE_NAME = 'myapp_sid'
```
</details>

### Exercise 3: Enable HTTPS-Only Sessions

Configure sessions to only be sent over HTTPS.

<details>
<summary>Solution</summary>

```python
# settings.py
SESSION_COOKIE_SECURE = True
```
</details>

### Exercise 4: Browser-Close Sessions

Configure sessions to expire when the browser closes.

<details>
<summary>Solution</summary>

```python
# settings.py
SESSION_EXPIRE_AT_BROWSER_CLOSE = True
```
</details>

### Exercise 5: Strict CSRF Protection

Configure sessions with strict SameSite policy.

<details>
<summary>Solution</summary>

```python
# settings.py
SESSION_COOKIE_SAMESITE = 'Strict'
```
</details>

### Exercise 6: Cache-Backed Sessions

Configure Django to use cache-backed sessions.

<details>
<summary>Solution</summary>

```python
# settings.py
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    }
}
```
</details>

### Exercise 7: Production Security Configuration

Configure sessions for production with maximum security.

<details>
<summary>Solution</summary>

```python
# settings.py
SESSION_COOKIE_AGE = 3600  # 1 hour
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_SAMESITE = 'Strict'
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
```
</details>

## Summary and Next Steps

You've now completed the Django Sessions learning series! Here's what you've learned:

### Completed Guides

1. **[001-django-sessions-fundamentals.md](001-django-sessions-fundamentals.md)**
   - What sessions are and how they work
   - Sessions vs cookies (server-side vs client-side)
   - Session ID and database storage
   - Setting, getting, updating, and deleting session data
   - SessionMiddleware and django_session table

2. **[002-advanced-session-methods.md](002-advanced-session-methods.md)**
   - `flush()` method for complete session deletion
   - `set_expiry()` for custom expiration times
   - `clear_expired()` for cleanup
   - `get_expiry_age()` and `get_expiry_date()` for checking expiration
   - `session.modified` flag for nested data changes

3. **[003-session-settings.md](003-session-settings.md)** (This Guide)
   - `SESSION_COOKIE_AGE` for default expiration
   - `SESSION_COOKIE_NAME` for custom cookie names
   - `SESSION_COOKIE_HTTPONLY` for security
   - `SESSION_COOKIE_SECURE` for HTTPS-only
   - `SESSION_COOKIE_SAMESITE` for CSRF protection
   - `SESSION_EXPIRE_AT_BROWSER_CLOSE` for browser-close sessions
   - `SESSION_ENGINE` for storage backend selection
   - `SESSION_SAVE_EVERY_REQUEST` for sliding expiration

### Key Skills Acquired

- ✅ Understand sessions vs cookies and when to use each
- ✅ Set, get, update, and delete session data
- ✅ Use session methods for advanced operations
- ✅ Configure session settings globally
- ✅ Implement secure session practices
- ✅ Handle session expiration and cleanup
- ✅ Choose appropriate session backends
- ✅ Configure session security settings
- ✅ Manage nested data in sessions
- ✅ Implement session-based features (login, cart, etc.)

### Further Learning

To deepen your understanding, consider exploring:

- **Django Authentication**: Built on sessions for user authentication
- **Django User Model**: Storing user information
- **Third-Party Authentication**: OAuth, OpenID Connect
- **Session Backends**: Redis, Memcached for production
- **CSRF Protection**: Cross-Site Request Forgery prevention
- **Django Signals**: Session lifecycle events

### Official Documentation

- Django Sessions Documentation: https://docs.djangoproject.com/en/stable/topics/http/sessions/
- Django Settings Reference: https://docs.djangoproject.com/en/stable/ref/settings/#sessions
- Django Session Backends: https://docs.djangoproject.com/en/stable/topics/http/sessions/#using-sessions-in-views

Congratulations on completing this comprehensive Django Sessions learning series! You now have the skills to effectively use sessions in your Django applications, from basic CRUD operations to advanced configuration and security best practices. Sessions are a fundamental building block for implementing authentication, shopping carts, and personalized user experiences in Django.
