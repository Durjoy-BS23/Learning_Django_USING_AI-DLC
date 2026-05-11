# Access Control & Logout

## Introduction

Access control ensures that only authorized users can access specific parts of your application. Django provides tools to restrict access based on authentication status and to handle logout functionality securely. Understanding how to control access and implement logout is essential for building secure, user-aware applications.

## Concept Explanation

### request.user Attribute

Django's AuthenticationMiddleware adds the `request.user` attribute to every request:
- If user is logged in: `request.user` is the User object
- If user is not logged in: `request.user` is an AnonymousUser object

**AnonymousUser Properties:**
- `is_authenticated`: Always `False`
- `is_anonymous`: Always `True`
- `id`: Always `None`
- `username`: Empty string
- No access to User model fields like email, first_name, etc.

### is_authenticated Property

The `is_authenticated` property checks if a user is logged in:
- Returns `True` for authenticated users
- Returns `False` for AnonymousUser
- Read-only property (cannot be set)
- Available on both User and AnonymousUser objects

**Usage:**
```python
if request.user.is_authenticated:
    # User is logged in
    pass
else:
    # User is not logged in
    pass
```

### Restricting Access to Authenticated Users

Prevent anonymous users from accessing protected views:
```python
def protected_view(request):
    if not request.user.is_authenticated:
        return redirect('login')
    # Protected code here
    pass
```

### Restricting Access to Anonymous Users

Prevent authenticated users from accessing public-only views (like login/register):
```python
def login_view(request):
    if request.user.is_authenticated:
        return redirect('home')
    # Login code here
    pass
```

### Context Processors

Context processors add variables to the template context automatically. Django's `auth` context processor adds the `user` variable to all templates:

**settings.py (default):**
```python
TEMPLATES = [
    {
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',  # Adds 'user'
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

**Effect:**
- `user` variable available in all templates
- Same as `request.user` in views
- Can be used to show/hide elements based on authentication status

### Authentication Data in Templates

Access user data in templates using the `user` variable:
```html
{% if user.is_authenticated %}
    <p>Welcome, {{ user.username }}!</p>
{% else %}
    <p>Please <a href="{% url 'login' %}">login</a></p>
{% endif %}
```

**Available Template Variables:**
- `user.is_authenticated`: Boolean
- `user.username`: Username
- `user.email`: Email address
- `user.first_name`: First name
- `user.last_name`: Last name
- `user.get_full_name()`: Returns first_name + last_name or username

### logout() Function

The `logout()` function logs out a user:
- Takes `request` as argument
- Deletes session data from database
- Removes session ID cookie
- Sets `request.user` to AnonymousUser
- No return value (void function)

**Signature:**
```python
logout(request)
```

**What It Does:**
- Calls `request.session.flush()` - complete session deletion
- Removes session cookie
- User becomes unauthenticated
- Subsequent requests see AnonymousUser

### Logout Workflow

1. User clicks logout button/link
2. Request sent to logout view
3. View calls `logout(request)`
4. Session data deleted from database
5. Session cookie removed
6. User redirected (typically to login page)
7. Subsequent requests show AnonymousUser

## Code Examples

### Restricting Access to Authenticated Users

**views.py:**
```python
from django.shortcuts import render, redirect
from django.http import HttpResponse

def protected_view(request):
    if not request.user.is_authenticated:
        return redirect('login')
    
    return HttpResponse(f"Welcome, {request.user.username}!")
```

### Restricting Access to Anonymous Users

**views.py:**
```python
from django.shortcuts import render, redirect

def login_view(request):
    if request.user.is_authenticated:
        return redirect('home')
    
    # Show login form
    return render(request, 'accounts/login.html')
```

### Combined Access Control

**views.py:**
```python
from django.shortcuts import render, redirect

def register_view(request):
    if request.user.is_authenticated:
        return redirect('home')
    
    # Show registration form
    return render(request, 'accounts/register.html')
```

### Authentication Data in Templates

**templates/base.html:**
```html
<nav class="navbar">
    {% if user.is_authenticated %}
        <span>{{ user.username }}</span>
        <a href="{% url 'logout' %}">Logout</a>
    {% else %}
        <a href="{% url 'login' %}">Login</a>
        <a href="{% url 'register' %}">Register</a>
    {% endif %}
</nav>
```

### Displaying User Full Name

**templates/base.html:**
```html
{% if user.is_authenticated %}
    <p>Welcome, {{ user.get_full_name|default:user.username }}!</p>
{% endif %}
```

### Implementing Logout

**accounts/views.py:**
```python
from django.shortcuts import redirect
from django.contrib.auth import logout
from django.urls import reverse

def auth_logout(request):
    logout(request)
    return redirect(reverse('login'))
```

**accounts/urls.py:**
```python
from django.urls import path
from . import views

urlpatterns = [
    path('logout/', views.auth_logout, name='logout'),
]
```

### Logout with POST Request (Django 5.0+)

**views.py:**
```python
from django.shortcuts import redirect
from django.contrib.auth import logout

def auth_logout(request):
    if request.method == 'POST':
        logout(request)
        return redirect('login')
    return redirect('login')  # GET requests also redirect
```

**templates/base.html:**
```html
{% if user.is_authenticated %}
    <form action="{% url 'logout' %}" method="post">
        {% csrf_token %}
        <button type="submit">Logout</button>
    </form>
{% endif %}
```

### Conditional Navigation Based on Auth Status

**templates/base.html:**
```html
<nav class="navbar navbar-expand-lg">
    <div class="container">
        <a class="navbar-brand" href="{% url 'home' %}">MySite</a>
        
        <div class="navbar-nav ms-auto">
            {% if user.is_authenticated %}
                <span class="nav-item me-3">{{ user.username }}</span>
                <a href="{% url 'logout' %}" class="nav-link">Logout</a>
            {% else %}
                <a href="{% url 'login' %}" class="nav-link">Login</a>
                <a href="{% url 'register' %}" class="nav-link">Register</a>
            {% endif %}
        </div>
    </div>
</nav>
```

### Access Control Decorator Alternative

**views.py:**
```python
from django.contrib.auth.decorators import login_required

@login_required(login_url='/accounts/login/')
def protected_view(request):
    return HttpResponse(f"Welcome, {request.user.username}!")
```

**Note:** This is a built-in decorator that provides the same functionality as manual checks.

### Checking User Properties in Views

**views.py:**
```python
from django.shortcuts import render

def profile_view(request):
    if not request.user.is_authenticated:
        return redirect('login')
    
    context = {
        'username': request.user.username,
        'email': request.user.email,
        'full_name': request.user.get_full_name(),
        'is_staff': request.user.is_staff,
        'date_joined': request.user.date_joined,
    }
    return render(request, 'accounts/profile.html', context)
```

### Session Inspection After Logout

**views.py:**
```python
from django.shortcuts import redirect
from django.contrib.auth import logout

def auth_logout(request):
    print(f"Session before logout: {dict(request.session)}")
    logout(request)
    print(f"Session after logout: {dict(request.session)}")
    return redirect('login')
```

## Key Takeaways

- `request.user` is added by AuthenticationMiddleware to every request
- `request.user.is_authenticated` checks if user is logged in
- AnonymousUser represents unauthenticated users with `is_authenticated=False`
- Restrict access by checking `request.user.is_authenticated` in views
- Auth context processor adds `user` variable to all templates
- Use `{% if user.is_authenticated %}` in templates for conditional content
- `logout()` function deletes session and logs out user
- Django 5.0+ requires POST request for logout (security measure)
- Logout removes session data and session cookie
- Use redirects to guide users after access control checks
- `user.get_full_name()` returns first_name + last_name or username

## Additional Context & Best Practices

### Access Control Best Practices

**1. Use @login_required Decorator**
```python
# ✅ GOOD - Clean and reusable
from django.contrib.auth.decorators import login_required

@login_required(login_url='/login/')
def protected_view(request):
    pass

# ✅ ALSO GOOD - Manual check for custom logic
def protected_view(request):
    if not request.user.is_authenticated:
        return redirect('login')
```

**2. Provide Clear Redirects**
```python
# ✅ GOOD - Redirect to login with next parameter
from django.contrib.auth.decorators import login_required

@login_required(login_url='/login/?next=/protected/')
def protected_view(request):
    pass

# ✅ GOOD - Store intended URL in session
def protected_view(request):
    if not request.user.is_authenticated:
        request.session['next'] = request.path
        return redirect('login')
```

**3. Check Authentication Before Accessing User Fields**
```python
# ✅ GOOD - Check before accessing fields
if request.user.is_authenticated:
    email = request.user.email
else:
    email = None

# ❌ BAD - AttributeError for AnonymousUser
email = request.user.email  # Fails for anonymous users
```

### Logout Best Practices

**1. Use POST for Logout (Django 5.0+)**
```html
<!-- ✅ GOOD - POST request -->
<form action="{% url 'logout' %}" method="post">
    {% csrf_token %}
    <button type="submit">Logout</button>
</form>

<!-- ❌ BAD - GET request (deprecated) -->
<a href="{% url 'logout' %}">Logout</a>
```

**2. Redirect After Logout**
```python
# ✅ GOOD - Redirect to login
def auth_logout(request):
    logout(request)
    return redirect('login')

# ✅ GOOD - Redirect to home
def auth_logout(request):
    logout(request)
    return redirect('home')
```

**3. Clean Up Additional Data**
```python
# ✅ GOOD - Clean up session data
def auth_logout(request):
    # Clean up custom session data
    if 'cart' in request.session:
        del request.session['cart']
    
    logout(request)
    return redirect('login')
```

### Common Pitfalls

**1. Forgetting to Check is_authenticated**
```python
# ❌ WRONG - AttributeError for AnonymousUser
def view(request):
    return HttpResponse(f"Hello, {request.user.username}")

# ✅ CORRECT - Check first
def view(request):
    if request.user.is_authenticated:
        return HttpResponse(f"Hello, {request.user.username}")
    return HttpResponse("Hello, Guest")
```

**2. Using GET for Logout (Django 5.0+)**
```python
# ❌ WRONG - GET method deprecated
def auth_logout(request):
    logout(request)
    return redirect('login')

# ✅ CORRECT - Require POST
def auth_logout(request):
    if request.method == 'POST':
        logout(request)
    return redirect('login')
```

**3. Not Using Context Processor**
```python
# ❌ WRONG - Manually passing user to every template
def view(request):
    return render(request, 'template.html', {'user': request.user})

# ✅ CORRECT - Use context processor (automatic)
def view(request):
    return render(request, 'template.html')  # user available automatically
```

**4. Hardcoding Redirect URLs**
```python
# ❌ WRONG - Hardcoded URL
def protected_view(request):
    if not request.user.is_authenticated:
        return redirect('/accounts/login/')

# ✅ CORRECT - Use URL names
def protected_view(request):
    if not request.user.is_authenticated:
        return redirect('login')
```

### Security Considerations

**1. CSRF Protection for Logout Forms**
```html
<!-- ✅ GOOD - Include CSRF token -->
<form action="{% url 'logout' %}" method="post">
    {% csrf_token %}
    <button type="submit">Logout</button>
</form>

<!-- ❌ WRONG - No CSRF token -->
<form action="{% url 'logout' %}" method="post">
    <button type="submit">Logout</button>
</form>
```

**2. Prevent Logout via GET**
```python
# ✅ GOOD - Only allow POST
def auth_logout(request):
    if request.method != 'POST':
        return HttpResponse('Method not allowed', status=405)
    logout(request)
    return redirect('login')
```

**3. Secure Session Cookies**
```python
# settings.py
SESSION_COOKIE_HTTPONLY = True  # Prevent JavaScript access
SESSION_COOKIE_SECURE = True     # HTTPS only
SESSION_COOKIE_SAMESITE = 'Lax'  # CSRF protection
```

**4. Logout on Security Events**
```python
def security_breach_detected(request):
    # Logout user on security breach
    logout(request)
    return HttpResponse('Session terminated for security reasons')
```

### Performance Considerations

**1. Minimize Session Data**
```python
# ❌ WRONG - Storing large data in session
request.session['user_data'] = User.objects.all()

# ✅ CORRECT - Store only IDs
request.session['user_id'] = request.user.id
```

**2. Use Efficient Access Control**
```python
# ✅ GOOD - Decorator is efficient
@login_required
def view(request):
    pass

# ✅ GOOD - Manual check is also efficient
def view(request):
    if not request.user.is_authenticated:
        return redirect('login')
```

**3. Clean Up Expired Sessions**
```python
# Regular cleanup task
from django.contrib.sessions.backends.db import SessionStore

def cleanup_sessions():
    SessionStore.clear_expired()
```

## Practice Exercises

### Exercise 1: Protect a View

Create a view that only authenticated users can access.

<details>
<summary>Solution</summary>

```python
# views.py
from django.shortcuts import render, redirect

def protected_view(request):
    if not request.user.is_authenticated:
        return redirect('login')
    return render(request, 'protected.html')
```
</details>

### Exercise 2: Conditional Template Content

Show different content in a template based on authentication status.

<details>
<summary>Solution</summary>

```html
{% if user.is_authenticated %}
    <p>Welcome back, {{ user.username }}!</p>
    <a href="{% url 'logout' %}">Logout</a>
{% else %}
    <p>Please log in to continue.</p>
    <a href="{% url 'login' %}">Login</a>
{% endif %}
```
</details>

### Exercise 3: Implement Logout

Create a logout view that logs out the user and redirects to login page.

<details>
<summary>Solution</summary>

```python
# views.py
from django.shortcuts import redirect
from django.contrib.auth import logout

def auth_logout(request):
    logout(request)
    return redirect('login')
```
</details>

### Exercise 4: Prevent Login Page Access

Prevent authenticated users from accessing the login page.

<details>
<summary>Solution</summary>

```python
# views.py
def login_view(request):
    if request.user.is_authenticated:
        return redirect('home')
    # Show login form
    return render(request, 'login.html')
```
</details>

### Exercise 5: Display User Information

Display user's full name in the navigation bar, falling back to username if full name is not set.

<details>
<summary>Solution</summary>

```html
{% if user.is_authenticated %}
    <span>{{ user.get_full_name|default:user.username }}</span>
{% endif %}
```
</details>

## Next Steps

Now that you understand access control and logout, the final step is to learn about Django's built-in authentication views.

Continue to **[005-built-in-authentication-views.md](005-built-in-authentication-views.md)** to learn:
- Django's built-in LoginView and LogoutView
- Including auth URLs in your project
- Customizing built-in views with parameters
- Authentication settings (LOGIN_REDIRECT_URL, LOGOUT_REDIRECT_URL)
- redirect_authenticated_user parameter
- Using as_view() for class-based views
- Benefits of using built-in views over custom implementations
