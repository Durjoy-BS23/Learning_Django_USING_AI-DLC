# Django Cookies Fundamentals

## Introduction

Cookies are a fundamental mechanism for maintaining state in HTTP, which is inherently stateless. They allow web servers to store small pieces of data on the client's browser, which are then sent back to the server with subsequent requests. This guide covers what cookies are, how they work, their security implications, and how to implement them in Django for practical use cases like user preferences, tracking, and authentication.

## Concept Explanation

### What Are Cookies?

Cookies, also known as HTTP cookies, are small pieces of text data stored by a web server on the user's browser. They consist of name-value pairs, similar to Python dictionaries, and are sent back to the server that created them with each subsequent HTTP request.

**Key Characteristics:**
- **Small text data**: Typically limited to 4KB per cookie
- **Stored on client**: Reside in the user's browser
- **Sent automatically**: Browser sends cookies to the originating server with each request
- **Name-value pairs**: Data stored as key-value pairs
- **Domain-specific**: Cookies set by amazon.com are only sent to amazon.com

### How Cookies Work

The cookie workflow follows a simple pattern:

1. **Initial Request**: User's browser sends a request to the web server
2. **Server Response**: Server processes the request and includes `Set-Cookie` header in the response
3. **Cookie Storage**: Browser receives the response and stores the cookie on the client's machine
4. **Subsequent Requests**: Browser automatically sends the cookie back to the server with each request to the same domain
5. **Server Processing**: Server can read, modify, or delete cookies based on the request

**Example Flow:**
```
User → Server: GET /homepage
Server → User: HTTP 200 OK, Set-Cookie: theme=dark
User → Server: GET /profile (Cookie: theme=dark)
Server → User: HTTP 200 OK (reads theme=dark, applies dark theme)
```

### Same-Origin Policy

Cookies follow the same-origin policy for security:
- Cookies set by `example.com` are only sent to `example.com`
- They are NOT sent to `other-site.com` or subdomains unless configured
- This prevents other websites from accessing your site's cookies

**Security Benefit**: Prevents cross-site cookie theft and unauthorized access.

### Cookie Use Cases

**1. User Behavior Tracking**
- Track which pages users visit most
- Monitor time spent on specific pages
- Analyze product views and purchases
- Build user profiles for personalization

**2. Authentication**
- Store session identifiers
- Remember login state across requests
- Implement "remember me" functionality
- Maintain user sessions

**3. Personalization**
- Theme preferences (dark/light mode)
- Language settings
- Display preferences
- Custom layouts

**4. Shopping Carts**
- Store cart items
- Remember product selections
- Maintain cart state across sessions

### Setting Cookies in Django

Django provides the `set_cookie()` method on `HttpResponse` objects:

```python
from django.http import HttpResponse

def set_cookie_view(request):
    response = HttpResponse("Cookie set!")
    response.set_cookie('theme', 'dark')
    return response
```

**Important**: You must store the `HttpResponse` object in a variable to call methods on it before returning.

### Getting Cookies in Django

Django provides cookies through the `request.COOKIES` dictionary:

```python
def get_cookie_view(request):
    theme = request.COOKIES.get('theme', 'light')
    return HttpResponse(f"Current theme: {theme}")
```

**Note**: `request.COOKIES` is a dictionary, so use `.get()` to avoid `KeyError` if the cookie doesn't exist.

### Updating Cookies

To update a cookie, simply call `set_cookie()` again with the same name and a new value:

```python
def update_cookie_view(request):
    response = HttpResponse("Cookie updated!")
    response.set_cookie('theme', 'light')  # Updates existing 'theme' cookie
    return response
```

### Deleting Cookies

Use the `delete_cookie()` method to remove a cookie:

```python
def delete_cookie_view(request):
    response = HttpResponse("Cookie deleted!")
    response.delete_cookie('theme')
    return response
```

**Note**: This instructs the browser to delete the cookie. The cookie remains until the browser processes the response.

### Cookie Parameters

Django's `set_cookie()` method accepts several parameters:

| Parameter | Purpose | Default |
|-----------|---------|---------|
| `key` | Cookie name | Required |
| `value` | Cookie value | Required |
| `max_age` | Lifespan in seconds | Browser session |
| `expires` | Expiration date | Browser session |
| `path` | URL path for cookie | `/` |
| `domain` | Domain for cookie | Current domain |
| `secure` | HTTPS-only transmission | `False` |
| `httponly` | Prevent JavaScript access | `False` |
| `samesite` | CSRF protection | `Lax` |

**max_age**: Sets cookie lifespan in seconds. If not specified, cookie lasts until browser closes.

**secure**: When `True`, cookie only sent over HTTPS connections. Essential for production security.

**httponly**: When `True`, JavaScript cannot access the cookie via `document.cookie`. Prevents XSS attacks.

## Code Examples

### Basic Cookie Operations

**views.py**
```python
from django.http import HttpResponse

def set_theme(request):
    """Set a theme cookie"""
    response = HttpResponse("Theme set to dark")
    response.set_cookie('theme', 'dark')
    return response

def get_theme(request):
    """Get the theme cookie"""
    theme = request.COOKIES.get('theme', 'light')
    return HttpResponse(f"Current theme: {theme}")

def update_theme(request):
    """Update the theme cookie"""
    response = HttpResponse("Theme updated to light")
    response.set_cookie('theme', 'light')
    return response

def delete_theme(request):
    """Delete the theme cookie"""
    response = HttpResponse("Theme cookie deleted")
    response.delete_cookie('theme')
    return response
```

**urls.py**
```python
from django.urls import path
from . import views

urlpatterns = [
    path('set/', views.set_theme, name='set_theme'),
    path('get/', views.get_theme, name='get_theme'),
    path('update/', views.update_theme, name='update_theme'),
    path('delete/', views.delete_theme, name='delete_theme'),
]
```

### Cookie with Expiration

```python
def set_persistent_cookie(request):
    """Set a cookie that lasts for 1 hour"""
    response = HttpResponse("Persistent cookie set")
    response.set_cookie('user_preference', 'value', max_age=3600)  # 1 hour
    return response
```

### Secure and HttpOnly Cookies

```python
def set_secure_cookie(request):
    """Set a secure, HTTP-only cookie for authentication"""
    response = HttpResponse("Secure cookie set")
    response.set_cookie(
        'session_id',
        'abc123',
        max_age=3600,
        secure=True,      # Only sent over HTTPS
        httponly=True,    # JavaScript cannot access
        samesite='Strict'  # CSRF protection
    )
    return response
```

### Multiple Cookies

```python
def set_multiple_cookies(request):
    """Set multiple cookies at once"""
    response = HttpResponse("Multiple cookies set")
    response.set_cookie('theme', 'dark')
    response.set_cookie('language', 'en')
    response.set_cookie('font_size', '16px')
    return response
```

### Cookie-Based Theme Switcher

**views.py**
```python
def home(request):
    """Home page with theme based on cookie"""
    theme = request.COOKIES.get('theme', 'light')
    return render(request, 'home.html', {'theme': theme})

def set_theme(request, theme_name):
    """Set theme cookie and redirect"""
    response = redirect('home')
    response.set_cookie('theme', theme_name, max_age=86400)  # 24 hours
    return response
```

**urls.py**
```python
urlpatterns = [
    path('', views.home, name='home'),
    path('set-theme/<str:theme_name>/', views.set_theme, name='set_theme'),
]
```

**home.html**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Home</title>
    <style>
        body.light { background: white; color: black; }
        body.dark { background: #333; color: white; }
    </style>
</head>
<body class="{{ theme }}">
    <h1>Welcome</h1>
    <p>Current theme: {{ theme }}</p>
    <a href="{% url 'set_theme' 'light' %}">Light Theme</a>
    <a href="{% url 'set_theme' 'dark' %}">Dark Theme</a>
</body>
</html>
```

## Key Takeaways

- Cookies are small text data stored on the client's browser and sent back to the server
- Cookies follow same-origin policy for security
- Use `response.set_cookie()` to set cookies
- Use `request.COOKIES` dictionary to read cookies
- Use `response.delete_cookie()` to remove cookies
- Update cookies by calling `set_cookie()` with the same name
- `max_age` parameter controls cookie lifespan in seconds
- `secure=True` ensures cookies only sent over HTTPS
- `httponly=True` prevents JavaScript access (XSS protection)
- Common use cases: tracking, authentication, personalization
- Never store sensitive data in cookies (see next guide)

## Additional Context & Best Practices

### Cookie Security Best Practices

**1. Always Use HttpOnly for Sensitive Cookies**
```python
# ✅ CORRECT - Prevents XSS attacks
response.set_cookie('session_id', 'abc123', httponly=True)

# ❌ WRONG - JavaScript can steal the cookie
response.set_cookie('session_id', 'abc123')
```

**2. Use Secure Flag in Production**
```python
# ✅ CORRECT - Only sent over HTTPS
response.set_cookie('session_id', 'abc123', secure=True)

# ❌ WRONG - Sent over HTTP (insecure)
response.set_cookie('session_id', 'abc123')
```

**3. Set Appropriate Expiration**
```python
# ✅ GOOD - Sessions expire after reasonable time
response.set_cookie('session_id', 'abc123', max_age=3600)

# ❌ BAD - Sessions never expire (security risk)
response.set_cookie('session_id', 'abc123', max_age=31536000)  # 1 year
```

**4. Use SameSite for CSRF Protection**
```python
# ✅ GOOD - CSRF protection
response.set_cookie('session_id', 'abc123', samesite='Strict')

# ❌ BAD - Vulnerable to CSRF
response.set_cookie('session_id', 'abc123')
```

### Common Pitfalls

**1. Not Storing HttpResponse Before Setting Cookie**
```python
# ❌ WRONG - Can't call method on returned object
return HttpResponse("Set").set_cookie('theme', 'dark')

# ✅ CORRECT - Store in variable first
response = HttpResponse("Set")
response.set_cookie('theme', 'dark')
return response
```

**2. Assuming Cookie Exists**
```python
# ❌ WRONG - Raises KeyError if cookie doesn't exist
theme = request.COOKIES['theme']

# ✅ CORRECT - Use .get() with default
theme = request.COOKIES.get('theme', 'light')
```

**3. Forgetting max_age for Persistent Cookies**
```python
# ❌ WRONG - Cookie deleted when browser closes
response.set_cookie('preference', 'value')

# ✅ CORRECT - Cookie persists
response.set_cookie('preference', 'value', max_age=86400)
```

**4. Storing Large Data in Cookies**
```python
# ❌ WRONG - Cookies have 4KB limit
response.set_cookie('large_data', huge_string)

# ✅ CORRECT - Use database or session for large data
# Store ID in cookie, retrieve data from database
```

### Performance Considerations

**1. Cookie Size**
- Keep cookies small (under 4KB)
- Large cookies slow down every request
- Use database or sessions for large data

**2. Cookie Count**
- Limit number of cookies per domain
- Browser limits total cookies (typically 50-300)
- Consolidate related data into single cookies

**3. Cookie Transmission**
- Every cookie is sent with each request to the domain
- Unnecessary cookies increase bandwidth
- Use `path` parameter to limit cookie scope

### Advanced Tips

**1. Cookie Path Scope**
```python
# Cookie only sent to /admin/ URLs
response.set_cookie('admin_pref', 'value', path='/admin/')
```

**2. Cookie Domain Scope**
```python
# Cookie shared across subdomains
response.set_cookie('shared', 'value', domain='.example.com')
```

**3. Signed Cookies (for tamper protection)**
```python
from django.core.signing import Signer

signer = Signer()
signed_value = signer.sign('my_value')
response.set_cookie('data', signed_value)

# Verify when reading
original = signer.unsign(request.COOKIES.get('data'))
```

**4. Cookie Encoding**
```python
import json

# Store complex data as JSON
data = {'theme': 'dark', 'language': 'en'}
response.set_cookie('preferences', json.dumps(data))

# Read and decode
preferences = json.loads(request.COOKIES.get('preferences', '{}'))
```

## Practice Exercises

### Exercise 1: Set and Read a Cookie

Create a view that:
- Sets a cookie named `username` with value `john_doe`
- Returns a message confirming the cookie was set
- Create another view to read and display the username cookie

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse

def set_username(request):
    response = HttpResponse("Username cookie set!")
    response.set_cookie('username', 'john_doe')
    return response

def get_username(request):
    username = request.COOKIES.get('username', 'Guest')
    return HttpResponse(f"Username: {username}")

# urls.py
urlpatterns = [
    path('set-username/', views.set_username, name='set_username'),
    path('get-username/', views.get_username, name='get_username'),
]
```
</details>

### Exercise 2: Persistent Cookie

Create a view that sets a `last_visit` cookie that:
- Stores the current date/time
- Persists for 7 days
- Display the last visit time on subsequent visits

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse
from datetime import datetime

def track_visit(request):
    response = HttpResponse("Visit tracked!")
    now = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    response.set_cookie('last_visit', now, max_age=604800)  # 7 days
    return response

def show_last_visit(request):
    last_visit = request.COOKIES.get('last_visit', 'First visit')
    return HttpResponse(f"Last visit: {last_visit}")

# urls.py
urlpatterns = [
    path('track/', views.track_visit, name='track_visit'),
    path('last-visit/', views.show_last_visit, name='show_last_visit'),
]
```
</details>

### Exercise 3: Secure Cookie

Create a view that sets a secure authentication cookie with:
- Name: `auth_token`
- Value: `secret123`
- HTTPS only
- HTTP only
- 1 hour expiration

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse

def set_auth_cookie(request):
    response = HttpResponse("Auth cookie set!")
    response.set_cookie(
        'auth_token',
        'secret123',
        max_age=3600,
        secure=True,
        httponly=True,
        samesite='Strict'
    )
    return response
```
</details>

### Exercise 4: Cookie Counter

Create a view that:
- Increments a `visit_count` cookie on each visit
- Displays the current visit count
- Initialize to 1 on first visit

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse

def visit_counter(request):
    count = int(request.COOKIES.get('visit_count', 0)) + 1
    response = HttpResponse(f"Visit count: {count}")
    response.set_cookie('visit_count', str(count), max_age=86400)
    return response
```
</details>

### Exercise 5: Delete Cookie

Create a view that:
- Deletes the `visit_count` cookie
- Returns a message confirming deletion
- Handles case where cookie doesn't exist

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse

def reset_counter(request):
    response = HttpResponse("Counter reset!")
    response.delete_cookie('visit_count')
    return response
```
</details>

## Next Steps

Now that you understand the fundamentals of Django cookies, the next step is to learn about advanced cookie usage, their limitations, and security considerations.

Continue to **[002-advanced-cookies-limitations.md](002-advanced-cookies-limitations.md)** to learn:
- How to use cookies with the render function
- Cookie size and number limitations
- Security issues with client-side cookie modification
- Best practices for secure cookie usage
- When to use cookies vs sessions
- Alternatives to cookies for sensitive data
