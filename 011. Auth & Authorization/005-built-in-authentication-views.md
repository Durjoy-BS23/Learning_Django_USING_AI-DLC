# Built-in Authentication Views

## Introduction

Django provides built-in class-based views for common authentication tasks like login and logout. These views are pre-implemented, tested, and secure, saving you from writing boilerplate code. Understanding how to use and customize these views allows you to implement authentication quickly while maintaining the flexibility to adapt them to your needs.

## Concept Explanation

### Built-in Authentication Views

Django's `django.contrib.auth.views` module provides:
- **LoginView**: Handles user login
- **LogoutView**: Handles user logout
- **PasswordChangeView**: Handles password changes
- **PasswordResetView**: Handles password reset requests
- **PasswordResetDoneView**: Shows confirmation after reset request
- **PasswordResetConfirmView**: Handles password reset with token
- **PasswordResetCompleteView**: Shows confirmation after reset

This guide focuses on LoginView and LogoutView as they're the most commonly used.

### Why Use Built-in Views?

**Advantages:**
- Pre-implemented and tested by Django team
- Security best practices built-in
- Less code to write and maintain
- Easy to customize
- Consistent with Django conventions
- Handles edge cases automatically

**When to Use:**
- Standard login/logout functionality
- Projects without special authentication requirements
- When you want to reduce boilerplate code

**When Not to Use:**
- Custom authentication flows
- Additional business logic during login/logout
- Special session handling requirements

### Including Auth URLs

Django provides a URL configuration that includes all auth views:
```python
path('accounts/', include('django.contrib.auth.urls')),
```

This creates URLs for:
- `accounts/login/` → LoginView
- `accounts/logout/` → LogoutView
- `accounts/password_change/` → PasswordChangeView
- `accounts/password_reset/` → PasswordResetView
- And more...

**Note:** You can use any path prefix, not just `accounts/`.

### Template Requirements

Built-in views require specific templates with specific names:
- `registration/login.html` for LoginView
- `registration/logged_out.html` for LogoutView (optional)
- `registration/password_change_form.html` for PasswordChangeView
- And more...

Templates must be in a `registration` folder within your templates directory:
```
your_app/
    templates/
        registration/
            login.html
            logged_out.html
```

### as_view() Method

Built-in views are class-based views. To use them in URLs, you call the `as_view()` method:
```python
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(), name='login'),
]
```

The `as_view()` method:
- Converts class-based view to a function-based view
- Returns a view function that Django can call
- Allows passing configuration parameters

### Customizing Built-in Views

You can customize built-in views by passing parameters to `as_view()`:
```python
path('login/', auth_views.LoginView.as_view(
    template_name='my_custom_login.html',
    authentication_form=MyCustomForm
), name='login')
```

**Common Parameters:**
- `template_name`: Custom template path
- `authentication_form`: Custom form class
- `redirect_authenticated_user`: Redirect if already logged in
- `extra_context`: Additional template context

### Authentication Settings

Django provides settings to configure authentication behavior globally:

**LOGIN_REDIRECT_URL:**
- URL to redirect after successful login
- Default: `/accounts/profile/`
- Can be URL name or path

**LOGOUT_REDIRECT_URL:**
- URL to redirect after logout
- Default: None (shows default logged out page)
- Can be URL name or path

**settings.py:**
```python
LOGIN_REDIRECT_URL = '/home/'
LOGOUT_REDIRECT_URL = '/login/'
```

### redirect_authenticated_user Parameter

The `redirect_authenticated_user` parameter prevents already-authenticated users from accessing login page:
```python
auth_views.LoginView.as_view(
    redirect_authenticated_user=True
)
```

**Behavior:**
- If `True` and user is authenticated: redirect to LOGIN_REDIRECT_URL
- If `False` or user is anonymous: show login form
- Default: `False`

This is useful for preventing logged-in users from seeing the login form.

## Code Examples

### Including Auth URLs

**Project urls.py:**
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('django.contrib.auth.urls')),
    path('', include('myapp.urls')),
]
```

**Resulting URLs:**
- `/accounts/login/` → LoginView
- `/accounts/logout/` → LogoutView
- `/accounts/password_change/` → PasswordChangeView
- And more...

### Creating Login Template

**Create app and register:**
```bash
python manage.py startapp registration
```

**settings.py:**
```python
INSTALLED_APPS = [
    # ...
    'registration',
]
```

**registration/templates/registration/login.html:**
```html
{% extends 'base.html' %}

{% block body %}
<h1>Login</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>
{% endblock %}
```

### Using LoginView Directly in URLs

**accounts/urls.py:**
```python
from django.urls import path
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
]
```

### Customizing Template Name

**accounts/urls.py:**
```python
from django.urls import path
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(
        template_name='accounts/login.html'
    ), name='login'),
]
```

**accounts/templates/accounts/login.html:**
```html
{% extends 'base.html' %}
{% block body %}
<h1>Custom Login Page</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>
{% endblock %}
```

### Using Custom Form

**accounts/forms.py:**
```python
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

**accounts/urls.py:**
```python
from django.urls import path
from django.contrib.auth import views as auth_views
from .forms import LoginForm

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(
        template_name='accounts/login.html',
        authentication_form=LoginForm
    ), name='login'),
]
```

### Setting Redirect URLs

**settings.py:**
```python
LOGIN_REDIRECT_URL = '/'
LOGOUT_REDIRECT_URL = '/accounts/login/'
```

### Using redirect_authenticated_user

**accounts/urls.py:**
```python
from django.urls import path
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(
        redirect_authenticated_user=True
    ), name='login'),
]
```

**Behavior:**
- If user is logged in: redirect to LOGIN_REDIRECT_URL
- If user is not logged in: show login form

### Complete Custom Login Setup

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

**accounts/urls.py:**
```python
from django.urls import path
from django.contrib.auth import views as auth_views
from .forms import LoginForm

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(
        template_name='accounts/login.html',
        authentication_form=LoginForm,
        redirect_authenticated_user=True
    ), name='login'),
    path('logout/', auth_views.LogoutView.as_view(
        next_page='login'
    ), name='logout'),
]
```

**settings.py:**
```python
LOGIN_REDIRECT_URL = '/'
```

**accounts/templates/accounts/login.html:**
```html
{% extends 'base.html' %}
{% block body %}
<div class="container">
    <h1 class="text-center">Login</h1>
    <form method="post">
        {% csrf_token %}
        {% for field in form %}
            <div class="form-group">
                <label>{{ field.label }}</label>
                {{ field }}
                {% for error in field.errors %}
                    <small class="text-danger">{{ error }}</small>
                {% endfor %}
            </div>
        {% endfor %}
        <button type="submit" class="btn btn-primary">Login</button>
    </form>
</div>
{% endblock %}
```

### Logout with Custom Redirect

**accounts/urls.py:**
```python
from django.urls import path
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('logout/', auth_views.LogoutView.as_view(
        next_page='login'  # URL name
    ), name='logout'),
]
```

Or using settings:
```python
# settings.py
LOGOUT_REDIRECT_URL = '/login/'
```

### Logout with POST (Django 5.0+)

**accounts/urls.py:**
```python
from django.urls import path
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
]
```

**Template:**
```html
<form action="{% url 'logout' %}" method="post">
    {% csrf_token %}
    <button type="submit">Logout</button>
</form>
```

### Next Parameter for Redirect

The built-in LoginView supports a `next` parameter to redirect after login:
```html
<a href="{% url 'login' %}?next=/protected/">Login to access</a>
```

The user will be redirected to `/protected/` after successful login.

### Extra Context Parameter

Pass additional context to templates:
```python
path('login/', auth_views.LoginView.as_view(
    extra_context={'title': 'Please Login'}
), name='login'),
```

**Template:**
```html
<h1>{{ title }}</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>
```

## Key Takeaways

- Django provides built-in class-based views for authentication (LoginView, LogoutView)
- Use `include('django.contrib.auth.urls')` to include all auth URLs
- Built-in views require specific templates in `registration/` folder
- Use `as_view()` to use class-based views in URL patterns
- Customize built-in views by passing parameters to `as_view()`
- `template_name` parameter overrides default template path
- `authentication_form` parameter uses custom form class
- `redirect_authenticated_user` prevents logged-in users from accessing login
- `LOGIN_REDIRECT_URL` sets global redirect after successful login
- `LOGOUT_REDIRECT_URL` sets global redirect after logout
- `next_page` parameter in LogoutView sets redirect URL
- Built-in views reduce boilerplate and follow Django best practices

## Additional Context & Best Practices

### Built-in Views Best Practices

**1. Use Built-in Views When Possible**
```python
# ✅ GOOD - Use built-in views
path('login/', auth_views.LoginView.as_view(), name='login')

# ❌ AVOID - Reinventing the wheel unless necessary
def custom_login(request):
    # Custom implementation
    pass
```

**2. Set Global Redirect URLs**
```python
# ✅ GOOD - Use settings for global behavior
LOGIN_REDIRECT_URL = '/'
LOGOUT_REDIRECT_URL = '/login/'

# ✅ ALSO GOOD - Override per view if needed
path('login/', auth_views.LoginView.as_view(), name='login'),
path('logout/', auth_views.LogoutView.as_view(next_page='/custom/'), name='logout'),
```

**3. Use redirect_authenticated_user**
```python
# ✅ GOOD - Prevent logged-in users from seeing login form
path('login/', auth_views.LoginView.as_view(
    redirect_authenticated_user=True
), name='login'),
```

**4. Use Custom Forms for Styling**
```python
# ✅ GOOD - Custom form for CSS classes
path('login/', auth_views.LoginView.as_view(
    authentication_form=LoginForm
), name='login'),
```

### Common Pitfalls

**1. Wrong Template Location**
```python
# ❌ WRONG - Template in wrong location
# templates/accounts/login.html (won't be found by default)

# ✅ CORRECT - Template in registration folder
# templates/registration/login.html

# ✅ ALSO CORRECT - Override template_name
path('login/', auth_views.LoginView.as_view(
    template_name='accounts/login.html'
), name='login'),
```

**2. Forgetting as_view()**
```python
# ❌ WRONG - Missing as_view()
path('login/', auth_views.LoginView, name='login')

# ✅ CORRECT - Call as_view()
path('login/', auth_views.LoginView.as_view(), name='login')
```

**3. Not Setting Redirect URLs**
```python
# ❌ WRONG - User redirected to /accounts/profile/ (may not exist)
# No settings configured

# ✅ CORRECT - Set redirect URLs
LOGIN_REDIRECT_URL = '/'
LOGOUT_REDIRECT_URL = '/login/'
```

**4. Using GET for Logout (Django 5.0+)**
```html
<!-- ❌ WRONG - GET method deprecated -->
<a href="{% url 'logout' %}">Logout</a>

<!-- ✅ CORRECT - Use POST form -->
<form action="{% url 'logout' %}" method="post">
    {% csrf_token %}
    <button type="submit">Logout</button>
</form>
```

### Security Considerations

**1. HTTPS in Production**
```python
# settings.py
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

**2. Secure Session Settings**
```python
# settings.py
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Lax'
```

**3. Custom Logout Template (Optional)**
If you need a custom logout page:
```python
path('logout/', auth_views.LogoutView.as_view(
    template_name='accounts/logout.html'
), name='logout'),
```

**4. Validate Next Parameter**
The `next` parameter can be exploited for open redirects. Django validates it by default, but be aware:
```python
# Django automatically validates next parameter
# Only allows relative URLs or URLs from same host
```

### Performance Considerations

**1. Built-in Views Are Efficient**
- No performance penalty over custom views
- Optimized by Django team
- Use them without concern

**2. Template Loading**
- Built-in views use Django's template system
- Same performance as custom views
- Use template caching in production

**3. Session Storage**
- Built-in views use standard session framework
- Performance depends on session backend (database vs cache)
- Consider cache-backed sessions for high-traffic sites

### When to Use Custom Views

**1. Additional Business Logic**
```python
def custom_login(request):
    if request.method == 'POST':
        form = LoginForm(request, data=request.POST)
        if form.is_valid():
            user = form.get_user()
            login(request, user)
            # Custom logic: track login, send email, etc.
            track_user_login(user)
            send_login_notification(user)
            return redirect('home')
```

**2. Custom Session Handling**
```python
def custom_login(request):
    # Custom session handling
    login(request, user)
    request.session['custom_data'] = 'value'
    request.session.set_expiry(3600)  # Custom expiry
    return redirect('home')
```

**3. API Authentication**
```python
@csrf_exempt
def api_login(request):
    # API-specific authentication
    # JSON response instead of HTML
    pass
```

## Practice Exercises

### Exercise 1: Include Auth URLs

Include Django's built-in auth URLs in your project.

<details>
<summary>Solution</summary>

```python
# Project urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('django.contrib.auth.urls')),
]
```
</details>

### Exercise 2: Create Login Template

Create the required login template for the built-in LoginView.

<details>
<summary>Solution</summary>

```html
<!-- registration/templates/registration/login.html -->
{% extends 'base.html' %}
{% block body %}
<h1>Login</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Login</button>
</form>
{% endblock %}
```
</details>

### Exercise 3: Customize Template Name

Use a custom template path for LoginView.

<details>
<summary>Solution</summary>

```python
# urls.py
from django.urls import path
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(
        template_name='myapp/custom_login.html'
    ), name='login'),
]
```
</details>

### Exercise 4: Set Redirect URLs

Configure global redirect URLs in settings.

<details>
<summary>Solution</summary>

```python
# settings.py
LOGIN_REDIRECT_URL = '/'
LOGOUT_REDIRECT_URL = '/accounts/login/'
```
</details>

### Exercise 5: Use Custom Form

Use a custom AuthenticationForm with LoginView.

<details>
<summary>Solution</summary>

```python
# forms.py
from django import forms
from django.contrib.auth.forms import AuthenticationForm

class LoginForm(AuthenticationForm):
    username = forms.CharField(
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    password = forms.CharField(
        widget=forms.PasswordInput(attrs={'class': 'form-control'})
    )

# urls.py
from django.urls import path
from django.contrib.auth import views as auth_views
from .forms import LoginForm

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(
        authentication_form=LoginForm
    ), name='login'),
]
```
</details>

## Summary

You've completed all 5 guides on Django Authentication & Authorization:

1. **Authentication Fundamentals** - Understanding the authentication system, User model, and session-based authentication
2. **User Registration** - Implementing registration with UserCreationForm and custom forms
3. **Login Functionality** - Using AuthenticationForm, authenticate(), and login() functions
4. **Access Control & Logout** - Restricting access, using authentication data in templates, and implementing logout
5. **Built-in Authentication Views** - Using Django's pre-built LoginView and LogoutView with customization

You now have a comprehensive understanding of Django's authentication system and can implement secure user authentication in your Django applications. Whether you choose to build custom views or use Django's built-in views, you have the knowledge to make the right decision for your project.
