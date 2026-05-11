# Django Authentication Fundamentals

## Introduction

Authentication is the process of verifying the identity of a user attempting to access a system. In Django, authentication is a built-in system that handles user registration, login, logout, and session management. Understanding authentication is essential for building secure web applications with user accounts, personalized experiences, and protected resources.

## Concept Explanation

### What Is Authentication?

Authentication answers the question "Who are you?" by verifying user credentials (typically username and password) against stored user data. Once authenticated, the system knows the user's identity and can grant access to protected resources.

**Key Concepts:**
- **Authentication**: Verifying user identity (username/password)
- **Authorization**: Determining what an authenticated user can do (permissions)
- **Session**: Maintaining authentication state across requests
- **User Model**: Django's default model for storing user information

### Django's Authentication System

Django provides a complete authentication system out of the box:
- **User model**: Stores user data (username, password, email, etc.)
- **Authentication middleware**: Adds `request.user` to every request
- **Session framework**: Maintains login state across requests
- **Built-in views and forms**: Common authentication functionality

### Required Components

**Installed Apps (settings.py):**
```python
INSTALLED_APPS = [
    # ...
    'django.contrib.auth',      # Authentication system
    'django.contrib.contenttypes',  # Required for auth
    # ...
]
```

**Required Middleware (settings.py):**
```python
MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',  # Must be first
    'django.contrib.auth.middleware.AuthenticationMiddleware',  # Must be after SessionMiddleware
    # ...
]
```

**Note**: SessionMiddleware must come before AuthenticationMiddleware because authentication depends on sessions.

### User Model

Django stores all users in the `auth_user` table (from `django.contrib.auth.models.User`). This single table stores:
- Regular users
- Staff users (can access admin)
- Superusers (full administrative access)

**User Model Fields:**
- `id`: Primary key
- `username`: Unique username (required)
- `password`: Encrypted password (required)
- `email`: Email address (optional)
- `first_name`: First name (optional)
- `last_name`: Last name (optional)
- `is_staff`: Can access admin (default: False)
- `is_superuser`: All permissions (default: False)
- `is_active`: Account status (default: True)
- `last_login`: Last login timestamp
- `date_joined`: Account creation timestamp

### How Authentication Works

**Registration Flow:**
1. User fills registration form with username, password, etc.
2. Server validates data (checks if username exists, password strength)
3. If valid, create user in `auth_user` table
4. Password is hashed before storage (never stored as plain text)

**Login Flow:**
1. User fills login form with username and password
2. Server validates credentials using `authenticate()` function
3. If valid, `login()` function creates a session
4. Session stores user ID and authentication backend
5. Session ID cookie sent to browser
6. Subsequent requests include session ID cookie
7. AuthenticationMiddleware retrieves user from session
8. `request.user` set to authenticated user or AnonymousUser

**Logout Flow:**
1. User requests logout
2. `logout()` function deletes session data
3. Session ID cookie removed or expired
4. `request.user` becomes AnonymousUser

### Session-Based Authentication

Django uses sessions to maintain authentication state:
- **Session storage**: Database (default), cache, or file
- **Session ID**: Random string sent as cookie
- **Session data**: Encrypted dictionary containing:
  - `auth_user_id`: The user's ID
  - `auth_user_backend`: Authentication backend used
  - `auth_user_hash`: Security hash

**Security Benefits:**
- Passwords never sent to client
- Session data encrypted
- Session ID randomly generated
- Sessions can be expired

### AuthenticationMiddleware

This middleware adds the `request.user` attribute to every request:
- If user is logged in: `request.user` is the User object
- If user is not logged in: `request.user` is an AnonymousUser object

**AnonymousUser Properties:**
- `is_authenticated`: Always False
- `is_anonymous`: Always True
- `id`: Always None
- `username`: Empty string

## Code Examples

### Basic Authentication Setup

**settings.py** (default Django project already has this):
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

### Creating Authentication App

**Terminal:**
```bash
python manage.py startapp accounts
```

**settings.py:**
```python
INSTALLED_APPS = [
    # ...
    'accounts',
]
```

**Project urls.py:**
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('accounts.urls')),
]
```

**accounts/urls.py:**
```python
from django.urls import path
from . import views

urlpatterns = [
    # URLs will be added here
]
```

### Checking User Authentication Status

**views.py:**
```python
from django.http import HttpResponse

def check_auth(request):
    if request.user.is_authenticated:
        return HttpResponse(f"Logged in as: {request.user.username}")
    else:
        return HttpResponse("Not logged in")
```

### Accessing User Information

**views.py:**
```python
from django.http import HttpResponse

def user_info(request):
    if request.user.is_authenticated:
        user = request.user
        info = {
            'username': user.username,
            'email': user.email,
            'first_name': user.first_name,
            'last_name': user.last_name,
            'is_staff': user.is_staff,
            'is_superuser': user.is_superuser,
        }
        return HttpResponse(f"User info: {info}")
    return HttpResponse("Not logged in")
```

### Understanding AnonymousUser

**views.py:**
```python
from django.http import HttpResponse
from django.contrib.auth.models import AnonymousUser

def check_user_type(request):
    if isinstance(request.user, AnonymousUser):
        return HttpResponse("This is an AnonymousUser")
    return HttpResponse(f"This is a User: {request.user.username}")
```

### Session Data Inspection

**views.py:**
```python
from django.http import HttpResponse

def session_info(request):
    if request.user.is_authenticated:
        session_data = request.session.items()
        return HttpResponse(f"Session data: {dict(session_data)}")
    return HttpResponse("No active session")
```

### Creating Superuser

**Terminal:**
```bash
python manage.py createsuperuser
```

**Example:**
```
Username: admin
Email address: admin@example.com
Password: ********
Password (again): ********
Superuser created successfully.
```

### Checking User in Database

**Python shell:**
```python
from django.contrib.auth.models import User

# Get all users
users = User.objects.all()
print(f"Total users: {users.count()}")

# Get specific user
user = User.objects.get(username='admin')
print(f"User: {user.username}")
print(f"Email: {user.email}")
print(f"Is staff: {user.is_staff}")
print(f"Is superuser: {user.is_superuser}")
```

## Key Takeaways

- Django provides a complete authentication system out of the box
- Authentication requires `django.contrib.auth` and `django.contrib.contenttypes` apps
- SessionMiddleware must come before AuthenticationMiddleware
- All users stored in single `auth_user` table (including superusers)
- AuthenticationMiddleware adds `request.user` to every request
- `request.user.is_authenticated` checks if user is logged in
- Sessions maintain authentication state across requests
- Passwords are hashed, never stored as plain text
- AnonymousUser represents unauthenticated users
- Built-in authentication system handles registration, login, and logout

## Additional Context & Best Practices

### Authentication Best Practices

**1. Never Store Plain Text Passwords**
```python
# ✅ CORRECT - Django handles password hashing automatically
user = User.objects.create_user(username='john', password='secure123')

# ❌ WRONG - Never set password directly
user.password = 'secure123'  # This won't work and is insecure
```

**2. Use create_user() for Regular Users**
```python
# ✅ CORRECT - Password is hashed
user = User.objects.create_user(username='john', password='secure123')

# ❌ WRONG - Password not hashed
user = User.objects.create(username='john', password='secure123')
```

**3. Use create_superuser() for Admins**
```python
# ✅ CORRECT - Sets is_staff=True and is_superuser=True
user = User.objects.create_superuser(
    username='admin',
    email='admin@example.com',
    password='admin123'
)

# ❌ WRONG - Must set is_staff and is_superuser manually
user = User.objects.create_user(username='admin', password='admin123')
user.is_staff = True
user.is_superuser = True
user.save()
```

**4. Middleware Order Matters**
```python
# ✅ CORRECT - SessionMiddleware before AuthenticationMiddleware
MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
]

# ❌ WRONG - Order reversed, won't work
MIDDLEWARE = [
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
]
```

### Common Pitfalls

**1. Forgetting to Register Apps**
```python
# ❌ WRONG - Apps not registered
INSTALLED_APPS = [
    # Missing 'django.contrib.auth'
]

# ✅ CORRECT - Include required apps
INSTALLED_APPS = [
    'django.contrib.auth',
    'django.contrib.contenttypes',
]
```

**2. Assuming request.user Always Exists**
```python
# ❌ WRONG - Can cause AttributeError
username = request.user.username

# ✅ CORRECT - Check if authenticated first
if request.user.is_authenticated:
    username = request.user.username
else:
    username = "Guest"
```

**3. Not Understanding AnonymousUser**
```python
# ❌ WRONG - AnonymousUser doesn't have all User attributes
if request.user.email:  # Raises AttributeError for AnonymousUser
    pass

# ✅ CORRECT - Check authentication first
if request.user.is_authenticated and request.user.email:
    pass
```

**4. Confusing Authentication with Authorization**
```python
# Authentication: Who are you?
if request.user.is_authenticated:
    # User is logged in
    pass

# Authorization: What can you do?
if request.user.has_perm('app.can_edit'):
    # User has permission
    pass
```

### Security Considerations

**1. Password Strength**
Django's default UserCreationForm enforces password strength:
- Minimum 8 characters
- Not too similar to personal information
- Not a common password
- Not entirely numeric

**2. Password Hashing**
Django uses PBKDF2 with SHA256 by default:
- Passwords are never stored in plain text
- Hashing is one-way (cannot be reversed)
- Salt is added for security
- Configurable hashers available

**3. Session Security**
- Session data is encrypted
- Session ID is random
- Sessions expire after inactivity
- Use HTTPS in production

**4. CSRF Protection**
Always include CSRF token in forms:
```html
<form method="post">
    {% csrf_token %}
    <!-- form fields -->
</form>
```

### Performance Considerations

**1. Database Queries**
```python
# ❌ WRONG - Queries database on every request
def view(request):
    if request.user.is_authenticated:
        user = User.objects.get(username=request.user.username)

# ✅ CORRECT - request.user is already loaded
def view(request):
    if request.user.is_authenticated:
        user = request.user  # No additional query
```

**2. Session Storage**
- Database-backed sessions: Default, good for most cases
- Cache-backed sessions: Faster for high-traffic sites
- File-based sessions: Not recommended for production

**3. Session Cleanup**
Expired sessions accumulate in database. Clean them regularly:
```python
from django.contrib.sessions.backends.db import SessionStore
SessionStore.clear_expired()
```

## Practice Exercises

### Exercise 1: Check Authentication Status

Create a view that returns different messages based on whether the user is authenticated or not.

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse

def auth_status(request):
    if request.user.is_authenticated:
        return HttpResponse(f"Welcome, {request.user.username}!")
    return HttpResponse("Please log in to continue.")
```
</details>

### Exercise 2: Display User Information

Create a view that displays the authenticated user's username, email, and account creation date.

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse

def user_details(request):
    if request.user.is_authenticated:
        user = request.user
        details = (
            f"Username: {user.username}\n"
            f"Email: {user.email}\n"
            f"Joined: {user.date_joined}"
        )
        return HttpResponse(details)
    return HttpResponse("Not logged in")
```
</details>

### Exercise 3: Check User Type

Create a view that checks if the authenticated user is a staff member or superuser.

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse

def check_privileges(request):
    if not request.user.is_authenticated:
        return HttpResponse("Not logged in")
    
    if request.user.is_superuser:
        return HttpResponse("Superuser - Full access")
    elif request.user.is_staff:
        return HttpResponse("Staff member - Limited admin access")
    return HttpResponse("Regular user - No admin access")
```
</details>

### Exercise 4: Session Information

Create a view that displays session information for authenticated users.

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse

def session_details(request):
    if request.user.is_authenticated:
        session_key = request.session.session_key
        return HttpResponse(f"Session key: {session_key}")
    return HttpResponse("No active session")
```
</details>

### Exercise 5: Create User Programmatically

Create a view that creates a new user (for demonstration purposes only).

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse
from django.contrib.auth.models import User

def create_user(request):
    try:
        user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        return HttpResponse(f"User created: {user.username}")
    except Exception as e:
        return HttpResponse(f"Error: {str(e)}")
```
</details>

## Next Steps

Now that you understand the fundamentals of Django authentication, the next step is to learn how to implement user registration.

Continue to **[002-user-registration.md](002-user-registration.md)** to learn:
- Using UserCreationForm for registration
- Customizing UserCreationForm for additional fields
- Form validation and error handling
- Manual form rendering for custom styling
- Best practices for user registration
