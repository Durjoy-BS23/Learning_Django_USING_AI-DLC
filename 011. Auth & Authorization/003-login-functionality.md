# Login Functionality

## Introduction

Login functionality allows users to authenticate themselves and access protected resources in your Django application. Django provides built-in tools like `AuthenticationForm`, `authenticate()`, and `login()` functions to handle the login process securely. Understanding how these components work together is essential for implementing secure authentication in your applications.

## Concept Explanation

### AuthenticationForm

Django's `AuthenticationForm` is a built-in form that:
- Provides username and password fields
- Validates credentials using `authenticate()` function
- Shows error messages for invalid credentials
- Handles CSRF protection

**Default Fields:**
- `username`: Text field for username
- `password`: Password field for password

**Built-in Validation:**
- Checks if username exists
- Verifies password matches
- Returns user object if valid
- Returns error message if invalid

### authenticate() Function

The `authenticate()` function verifies user credentials:
- Takes `request`, `username`, and `password` as arguments
- Checks credentials against the database
- Returns User object if credentials are valid
- Returns `None` if credentials are invalid

**Signature:**
```python
authenticate(request=None, **credentials)
```

**Usage:**
```python
user = authenticate(request, username='john', password='secret')
if user is not None:
    # Credentials are valid
    pass
else:
    # Credentials are invalid
    pass
```

### login() Function

The `login()` function creates a session for authenticated users:
- Takes `request` and `user` object as arguments
- Stores user ID in session
- Stores authentication backend in session
- Sets session cookie on client's browser
- Persists authentication across requests

**Signature:**
```python
login(request, user, backend=None)
```

**What It Does:**
- Saves `auth_user_id` in session
- Saves `auth_user_backend` in session
- Generates session ID cookie
- User remains logged in until logout or session expires

### Session Data Structure

When a user logs in, Django stores this data in the session:
```python
{
    'auth_user_id': user.id,
    'auth_user_backend': 'django.contrib.auth.backends.ModelBackend',
    'auth_user_hash': 'security_hash'
}
```

**Session ID Cookie:**
- Random string sent to browser
- Links browser to session data in database
- Automatically sent with each request
- Used by AuthenticationMiddleware to retrieve user

### Login Workflow

1. User visits login page
2. Server renders AuthenticationForm
3. User enters username and password
4. Form submitted via POST
5. View creates AuthenticationForm with POST data
6. Form validates data
7. `authenticate()` checks credentials
8. If valid, `login()` creates session
9. User redirected to home page
10. Session ID cookie sent to browser
11. Subsequent requests include session ID
12. AuthenticationMiddleware retrieves user from session
13. `request.user` set to authenticated user

### UsernameField vs CharField

Django's `AuthenticationForm` uses `UsernameField` for the username:
- Inherits from `CharField`
- Validates username format
- Ensures username uniqueness
- Provides better validation than plain `CharField`

**When Customizing AuthenticationForm:**
- Use `UsernameField` for username (best practice)
- Use `CharField` only if you need custom validation
- Import from `django.contrib.auth.forms`

## Code Examples

### Basic Login Implementation

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.auth import authenticate, login
from django.urls import reverse

def auth_login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password')
            user = authenticate(request, username=username, password=password)
            if user is not None:
                login(request, user)
                return redirect(reverse('home'))
    else:
        form = AuthenticationForm()
    
    return render(request, 'accounts/login.html', {'form': form})
```

**accounts/urls.py:**
```python
from django.urls import path
from . import views

urlpatterns = [
    path('login/', views.auth_login, name='login'),
]
```

**templates/accounts/login.html:**
```html
{% extends 'base.html' %}

{% block body %}
<h1 class="text-center">Login</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit" class="btn btn-primary">Login</button>
</form>
{% endblock %}
```

### Customizing AuthenticationForm

**accounts/forms.py:**
```python
from django import forms
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.auth.forms import UsernameField

class LoginForm(AuthenticationForm):
    username = UsernameField(
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Username'
        })
    )
    password = forms.CharField(
        widget=forms.PasswordInput(attrs={
            'class': 'form-control',
            'placeholder': 'Password'
        })
    )
```

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login
from django.urls import reverse
from .forms import LoginForm

def auth_login(request):
    if request.method == 'POST':
        form = LoginForm(request, data=request.POST)
        if form.is_valid():
            user = form.get_user()
            login(request, user)
            return redirect(reverse('home'))
    else:
        form = LoginForm()
    
    return render(request, 'accounts/login.html', {'form': form})
```

### Manual Form Rendering

**templates/accounts/login.html:**
```html
{% extends 'base.html' %}

{% block body %}
<div class="container">
    <h1 class="text-center">Login</h1>
    <form method="post">
        {% csrf_token %}
        {% for field in form %}
            <div class="form-group">
                <label for="{{ field.id_for_label }}" class="form-label">
                    {{ field.label }}
                </label>
                {{ field }}
                {% if field.errors %}
                    <div class="text-danger">
                        {% for error in field.errors %}
                            <small>{{ error }}</small>
                        {% endfor %}
                    </div>
                {% endif %}
            </div>
        {% endfor %}
        {% if form.non_field_errors %}
            <div class="alert alert-danger">
                {% for error in form.non_field_errors %}
                    {{ error }}
                {% endfor %}
            </div>
        {% endif %}
        <button type="submit" class="btn btn-primary">Login</button>
    </form>
</div>
{% endblock %}
```

### Complete Login with Error Handling

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login
from django.contrib.auth.forms import AuthenticationForm
from django.urls import reverse
from django.http import HttpResponse

def auth_login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password')
            user = authenticate(request, username=username, password=password)
            if user is not None:
                login(request, user)
                return redirect(reverse('home'))
            else:
                # AuthenticationForm already shows error, but you can add custom logic
                return HttpResponse("Invalid credentials", status=401)
    else:
        form = AuthenticationForm()
    
    return render(request, 'accounts/login.html', {'form': form})
```

### Checking Session Data After Login

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login
from django.contrib.auth.forms import AuthenticationForm
from django.urls import reverse

def auth_login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password')
            user = authenticate(request, username=username, password=password)
            if user is not None:
                login(request, user)
                # Inspect session data
                print(f"Session data: {dict(request.session)}")
                return redirect(reverse('home'))
    else:
        form = AuthenticationForm()
    
    return render(request, 'accounts/login.html', {'form': form})
```

### Preventing Duplicate View Names

**accounts/views.py:**
```python
# Note: Named 'auth_login' instead of 'login' to avoid conflict
# with Django's built-in login function
def auth_login(request):
    # ... implementation
```

**accounts/urls.py:**
```python
urlpatterns = [
    path('login/', views.auth_login, name='login'),
]
```

### Login with Remember Me

**accounts/forms.py:**
```python
from django import forms
from django.contrib.auth.forms import AuthenticationForm

class LoginForm(AuthenticationForm):
    remember_me = forms.BooleanField(required=False)

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['remember_me'].label = 'Remember me'
```

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login
from django.urls import reverse
from .forms import LoginForm

def auth_login(request):
    if request.method == 'POST':
        form = LoginForm(request, data=request.POST)
        if form.is_valid():
            user = form.get_user()
            login(request, user)
            
            # Set session expiration based on remember me
            if form.cleaned_data.get('remember_me'):
                request.session.set_expiry(1209600)  # 2 weeks
            else:
                request.session.set_expiry(0)  # Browser close
            
            return redirect(reverse('home'))
    else:
        form = LoginForm()
    
    return render(request, 'accounts/login.html', {'form': form})
```

## Key Takeaways

- `AuthenticationForm` provides username and password fields with built-in validation
- `authenticate()` function verifies credentials and returns User object or None
- `login()` function creates session and stores user ID in session
- Session data includes `auth_user_id`, `auth_user_backend`, and security hash
- Session ID cookie links browser to session data in database
- AuthenticationMiddleware uses session to set `request.user`
- Use `UsernameField` for username field when customizing AuthenticationForm
- Always pass `request` to `authenticate()` for proper session handling
- Form validation happens before authentication
- `form.get_user()` returns authenticated user from AuthenticationForm
- Name views differently than built-in functions to avoid conflicts

## Additional Context & Best Practices

### Login Best Practices

**1. Always Pass Request to authenticate()**
```python
# ✅ GOOD - Pass request parameter
user = authenticate(request, username='john', password='secret')

# ❌ BAD - Missing request parameter
user = authenticate(username='john', password='secret')
```

**2. Check User is Not None**
```python
# ✅ GOOD - Explicit check
user = authenticate(request, username='john', password='secret')
if user is not None:
    login(request, user)

# ❌ BAD - Could fail silently
user = authenticate(request, username='john', password='secret')
login(request, user)  # Fails if user is None
```

**3. Use form.get_user() with AuthenticationForm**
```python
# ✅ GOOD - Use form's get_user() method
if form.is_valid():
    user = form.get_user()
    login(request, user)

# ✅ ALSO GOOD - Manual authentication
if form.is_valid():
    username = form.cleaned_data['username']
    password = form.cleaned_data['password']
    user = authenticate(request, username=username, password=password)
    if user is not None:
        login(request, user)
```

**4. Redirect After Successful Login**
```python
# ✅ GOOD - Redirect to home or next page
if user is not None:
    login(request, user)
    return redirect('home')

# ❌ BAD - No redirect, user stays on login page
if user is not None:
    login(request, user)
    return render(request, 'login.html')
```

### Common Pitfalls

**1. Not Passing Request to AuthenticationForm**
```python
# ❌ WRONG - Missing request parameter
form = AuthenticationForm(data=request.POST)

# ✅ CORRECT - Include request parameter
form = AuthenticationForm(request, data=request.POST)
```

**2. Forgetting CSRF Token**
```html
<!-- ❌ WRONG - No CSRF token -->
<form method="post">
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>

<!-- ✅ CORRECT - Include CSRF token -->
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>
```

**3. Naming Conflict with Built-in Functions**
```python
# ❌ WRONG - Shadows built-in login function
def login(request):
    from django.contrib.auth import login  # Circular import
    # ...

# ✅ CORRECT - Use different name
def auth_login(request):
    from django.contrib.auth import login as auth_login
    auth_login(request, user)
```

**4. Not Checking is_valid()**
```python
# ❌ WRONG - No validation
form = AuthenticationForm(request, data=request.POST)
user = authenticate(request, **form.cleaned_data)

# ✅ CORRECT - Validate first
form = AuthenticationForm(request, data=request.POST)
if form.is_valid():
    user = authenticate(request, **form.cleaned_data)
```

### Security Considerations

**1. Rate Limiting Login Attempts**
```python
from django.core.cache import cache

def auth_login(request):
    ip = request.META.get('REMOTE_ADDR')
    attempts = cache.get(f'login_attempts_{ip}', 0)
    
    if attempts >= 5:
        return HttpResponse("Too many attempts. Try again later.", status=429)
    
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            user = form.get_user()
            if user is None:
                cache.set(f'login_attempts_{ip}', attempts + 1, 300)
            else:
                cache.delete(f'login_attempts_{ip}')
                login(request, user)
                return redirect('home')
```

**2. HTTPS Only in Production**
```python
# settings.py
SESSION_COOKIE_SECURE = True  # Only send over HTTPS
CSRF_COOKIE_SECURE = True
```

**3. Secure Session Settings**
```python
# settings.py
SESSION_COOKIE_HTTPONLY = True  # Prevent JavaScript access
SESSION_COOKIE_SAMESITE = 'Lax'  # CSRF protection
SESSION_COOKIE_SECURE = True  # HTTPS only
```

**4. Don't Store Passwords in Session**
```python
# ❌ WRONG - Never store passwords in session
request.session['password'] = password

# ✅ CORRECT - Only store user ID (handled by login())
login(request, user)  # Django handles this securely
```

### Performance Considerations

**1. Database Queries**
```python
# ❌ WRONG - Additional query
user = authenticate(request, username=username, password=password)
user = User.objects.get(username=username)  # Unnecessary query

# ✅ CORRECT - Use authenticated user
user = authenticate(request, username=username, password=password)
# user is already loaded from database
```

**2. Session Storage**
- Database-backed sessions: Default, good for most cases
- Cache-backed sessions: Faster for high-traffic sites
- Consider session size when storing additional data

**3. Authentication Backend**
Django's default backend is efficient. Only customize if needed:
```python
# settings.py
AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
]
```

## Practice Exercises

### Exercise 1: Basic Login View

Create a basic login view using AuthenticationForm.

<details>
<summary>Solution</summary>

```python
# accounts/views.py
from django.shortcuts import render, redirect
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.auth import authenticate, login

def auth_login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            user = form.get_user()
            login(request, user)
            return redirect('home')
    else:
        form = AuthenticationForm()
    
    return render(request, 'accounts/login.html', {'form': form})
```
</details>

### Exercise 2: Manual Authentication

Use authenticate() function instead of form.get_user().

<details>
<summary>Solution</summary>

```python
# accounts/views.py
from django.shortcuts import render, redirect
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.auth import authenticate, login

def auth_login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            username = form.cleaned_data['username']
            password = form.cleaned_data['password']
            user = authenticate(request, username=username, password=password)
            if user is not None:
                login(request, user)
                return redirect('home')
    else:
        form = AuthenticationForm()
    
    return render(request, 'accounts/login.html', {'form': form})
```
</details>

### Exercise 3: Custom AuthenticationForm

Create a custom LoginForm with CSS classes for all fields.

<details>
<summary>Solution</summary>

```python
# accounts/forms.py
from django import forms
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.auth.forms import UsernameField

class LoginForm(AuthenticationForm):
    username = UsernameField(
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    password = forms.CharField(
        widget=forms.PasswordInput(attrs={'class': 'form-control'})
    )
```
</details>

### Exercise 4: Session Expiration

Set session to expire when browser closes if "Remember me" is not checked.

<details>
<summary>Solution</summary>

```python
# accounts/views.py
def auth_login(request):
    if request.method == 'POST':
        form = LoginForm(request, data=request.POST)
        if form.is_valid():
            user = form.get_user()
            login(request, user)
            
            if not form.cleaned_data.get('remember_me', False):
                request.session.set_expiry(0)  # Browser close
            
            return redirect('home')
    else:
        form = LoginForm()
    
    return render(request, 'accounts/login.html', {'form': form})
```
</details>

### Exercise 5: Inspect Session Data

Print session data after successful login to understand what's stored.

<details>
<summary>Solution</summary>

```python
# accounts/views.py
def auth_login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            user = form.get_user()
            login(request, user)
            
            # Print session data
            print(f"Session key: {request.session.session_key}")
            print(f"Session data: {dict(request.session)}")
            
            return redirect('home')
    else:
        form = AuthenticationForm()
    
    return render(request, 'accounts/login.html', {'form': form})
```
</details>

## Next Steps

Now that you understand login functionality, the next step is to learn about access control and logout.

Continue to **[004-access-control-and-logout.md](004-access-control-and-logout.md)** to learn:
- Restricting access to logged-in and anonymous users
- Using request.user.is_authenticated
- Authentication data in templates
- Implementing logout with logout() function
- Context processors for template access
- Best practices for access control
