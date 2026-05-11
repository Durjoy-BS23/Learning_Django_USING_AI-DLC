# User Registration

## Introduction

User registration is the process of creating new user accounts in your Django application. Django provides a built-in `UserCreationForm` that handles most of the complexity, including password validation, username uniqueness checks, and password hashing. Understanding how to implement and customize registration is essential for building user-centric applications.

## Concept Explanation

### UserCreationForm

Django's `UserCreationForm` is a ModelForm that:
- Inherits from the User model
- Includes username, password, and password confirmation fields
- Validates username uniqueness
- Validates password strength
- Hashes passwords before saving
- Automatically saves the user to the database

**Default Fields:**
- `username`: Text field (required)
- `password1`: Password field (required)
- `password2`: Password confirmation (required)

**Built-in Password Validation:**
- Minimum 8 characters
- Not too similar to username
- Not a common password
- Not entirely numeric
- Both passwords must match

### Why Extend UserCreationForm?

The default `UserCreationForm` only collects username and password. However, the User model has additional fields:
- `email`: Email address
- `first_name`: First name
- `last_name`: Last name

To collect this information during registration, you must:
1. Create a subclass of `UserCreationForm`
2. Add the additional fields to the form
3. Override field properties (required, widgets, etc.)

### Form Inheritance Pattern

When extending `UserCreationForm`:
- Inherit from `UserCreationForm`
- Define `Meta` class with model and fields
- Override fields to customize behavior
- Add widgets for CSS classes and attributes

**Important Notes:**
- Password fields (`password1`, `password2`) are defined in `UserCreationForm`, not the User model
- Don't add `password` field from User model - it will create duplicate fields
- Use `fields` in Meta to specify which User model fields to include

### Manual Form Rendering

For custom styling, render forms manually:
- Iterate over `form` in template
- Access `field.label` for label text
- Access `field` for the input
- Access `field.errors` for field-specific errors
- Access `form.non_field_errors` for form-wide errors

This gives you complete control over HTML structure and CSS classes.

## Code Examples

### Basic User Registration

**accounts/forms.py:**
```python
from django.contrib.auth.forms import UserCreationForm
from django import forms

class RegisterForm(UserCreationForm):
    class Meta:
        model = User
        fields = ('username',)
```

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.contrib.auth.forms import UserCreationForm
from django.urls import reverse

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect(reverse('home'))
    else:
        form = UserCreationForm()
    
    return render(request, 'accounts/register.html', {'form': form})
```

**accounts/urls.py:**
```python
from django.urls import path
from . import views

urlpatterns = [
    path('register/', views.register, name='register'),
]
```

**templates/accounts/register.html:**
```html
{% extends 'base.html' %}

{% block body %}
<h1>Register</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit" class="btn btn-primary">Register</button>
</form>
{% endblock %}
```

### Extending UserCreationForm for Additional Fields

**accounts/forms.py:**
```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    email = forms.EmailField(required=True)
    first_name = forms.CharField(required=True)
    last_name = forms.CharField(required=True)

    class Meta:
        model = User
        fields = ('username', 'email', 'first_name', 'last_name')
```

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.urls import reverse
from .forms import RegisterForm

def register(request):
    if request.method == 'POST':
        form = RegisterForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect(reverse('home'))
    else:
        form = RegisterForm()
    
    return render(request, 'accounts/register.html', {'form': form})
```

### Adding Custom Widgets and CSS Classes

**accounts/forms.py:**
```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    email = forms.EmailField(
        required=True,
        widget=forms.EmailInput(attrs={'class': 'form-control'})
    )
    first_name = forms.CharField(
        required=True,
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    last_name = forms.CharField(
        required=True,
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    
    # Override password fields for custom styling
    password1 = forms.CharField(
        widget=forms.PasswordInput(attrs={'class': 'form-control'})
    )
    password2 = forms.CharField(
        widget=forms.PasswordInput(attrs={'class': 'form-control'})
    )

    class Meta:
        model = User
        fields = ('username', 'email', 'first_name', 'last_name')
        widgets = {
            'username': forms.TextInput(attrs={'class': 'form-control'})
        }
```

### Manual Form Rendering

**templates/accounts/register.html:**
```html
{% extends 'base.html' %}

{% block body %}
<div class="container">
    <h1 class="text-center">Register</h1>
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
        <button type="submit" class="btn btn-primary">Register</button>
    </form>
</div>
{% endblock %}
```

### Complete Registration Example with Validation

**accounts/forms.py:**
```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    email = forms.EmailField(
        required=True,
        widget=forms.EmailInput(attrs={
            'class': 'form-control',
            'placeholder': 'Email address'
        })
    )
    first_name = forms.CharField(
        required=True,
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'First name'
        })
    )
    last_name = forms.CharField(
        required=True,
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Last name'
        })
    )
    password1 = forms.CharField(
        widget=forms.PasswordInput(attrs={
            'class': 'form-control',
            'placeholder': 'Password'
        })
    )
    password2 = forms.CharField(
        widget=forms.PasswordInput(attrs={
            'class': 'form-control',
            'placeholder': 'Confirm password'
        })
    )

    class Meta:
        model = User
        fields = ('username', 'email', 'first_name', 'last_name')
        widgets = {
            'username': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Username'
            })
        }
```

**accounts/views.py:**
```python
from django.shortcuts import render, redirect
from django.urls import reverse
from .forms import RegisterForm

def register(request):
    if request.method == 'POST':
        form = RegisterForm(request.POST)
        if form.is_valid():
            user = form.save()
            return redirect(reverse('login'))
    else:
        form = RegisterForm()
    
    return render(request, 'accounts/register.html', {'form': form})
```

### Preventing Duplicate Usernames

Django's `UserCreationForm` automatically checks for duplicate usernames. If you create a custom form, ensure you maintain this validation:

**accounts/forms.py:**
```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User
from django.core.exceptions import ValidationError

class RegisterForm(UserCreationForm):
    email = forms.EmailField(required=True)
    
    class Meta:
        model = User
        fields = ('username', 'email', 'first_name', 'last_name')
    
    def clean_username(self):
        username = self.cleaned_data.get('username')
        if User.objects.filter(username=username).exists():
            raise ValidationError("Username already exists")
        return username
```

Note: `UserCreationForm` already includes this validation, so you don't need to add it unless you're creating a completely custom form.

## Key Takeaways

- `UserCreationForm` provides built-in user registration with password validation
- Extend `UserCreationForm` to add additional fields (email, first name, last name)
- Use `required=True` to make optional User model fields required
- Override field widgets to add CSS classes and attributes
- Password fields (`password1`, `password2`) are defined in `UserCreationForm`, not User model
- Manual form rendering gives complete control over HTML structure
- Django automatically hashes passwords before saving
- Username uniqueness is automatically validated by `UserCreationForm`
- Use `form.is_valid()` to validate all form data
- Use `form.save()` to create the user in the database

## Additional Context & Best Practices

### Registration Best Practices

**1. Always Use UserCreationForm**
```python
# ✅ GOOD - Inherits built-in validation
class RegisterForm(UserCreationForm):
    pass

# ❌ BAD - Reinventing the wheel
class RegisterForm(forms.ModelForm):
    # You'd need to implement all validation yourself
    pass
```

**2. Collect Minimal Information**
```python
# ✅ GOOD - Only essential fields
class RegisterForm(UserCreationForm):
    email = forms.EmailField(required=True)

# ❌ BAD - Too many fields, poor UX
class RegisterForm(UserCreationForm):
    email = forms.EmailField(required=True)
    phone = forms.CharField(required=True)
    address = forms.CharField(required=True)
    # ... too many fields
```

**3. Provide Helpful Error Messages**
```python
# ✅ GOOD - Clear error messages
def clean_email(self):
    email = self.cleaned_data.get('email')
    if User.objects.filter(email=email).exists():
        raise ValidationError("Email already registered")
    return email
```

**4. Redirect After Registration**
```python
# ✅ GOOD - Redirect to login or home
def register(request):
    if form.is_valid():
        form.save()
        return redirect('login')  # or 'home'

# ❌ BAD - No redirect, user stays on form
def register(request):
    if form.is_valid():
        form.save()
        return render(request, 'register.html')
```

### Common Pitfalls

**1. Adding Password Field from User Model**
```python
# ❌ WRONG - Creates duplicate password fields
class RegisterForm(UserCreationForm):
    class Meta:
        model = User
        fields = ('username', 'password')  # Don't include password

# ✅ CORRECT - Let UserCreationForm handle passwords
class RegisterForm(UserCreationForm):
    class Meta:
        model = User
        fields = ('username',)
```

**2. Forgetting to Import User Model**
```python
# ❌ WRONG - User not imported
class RegisterForm(UserCreationForm):
    class Meta:
        model = User  # NameError

# ✅ CORRECT - Import User model
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    class Meta:
        model = User
```

**3. Not Handling Form Errors**
```python
# ❌ WRONG - No error handling
def register(request):
    if request.method == 'POST':
        form = RegisterForm(request.POST)
        form.save()  # Saves even if invalid
```

**4. Rendering Form Without CSRF Token**
```html
<!-- ❌ WRONG - No CSRF token -->
<form method="post">
    {{ form.as_p }}
    <button type="submit">Register</button>
</form>

<!-- ✅ CORRECT - Include CSRF token -->
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Register</button>
</form>
```

### Security Considerations

**1. Email Verification**
After registration, send verification email:
```python
def register(request):
    if form.is_valid():
        user = form.save(commit=False)
        user.is_active = False  # Inactive until email verified
        user.save()
        send_verification_email(user)
        return redirect('check_email')
```

**2. Password Strength**
Django's default password validators are good, but you can add custom ones:
```python
# settings.py
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 9,
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

**3. Rate Limiting**
Prevent brute force attacks on registration:
```python
from django.core.cache import cache

def register(request):
    # Check rate limit
    ip_address = request.META.get('REMOTE_ADDR')
    if cache.get(f'register_{ip_address}'):
        return HttpResponse("Too many registration attempts")
    
    # Set rate limit
    cache.set(f'register_{ip_address}', True, 300)  # 5 minutes
    
    # Process registration
    # ...
```

### Performance Considerations

**1. Database Indexes**
Ensure username and email are indexed (default in Django's User model):
```python
# Django's User model already has indexes on username
# Email is not indexed by default, but you can add it if needed
```

**2. Form Validation**
Django's built-in validation is efficient. Don't add unnecessary custom validation:
```python
# ❌ BAD - Unnecessary custom validation
def clean_username(self):
    username = self.cleaned_data['username']
    if len(username) < 3:
        raise ValidationError("Username too short")
    return username

# ✅ GOOD - Use Django's built-in validation
# User model already has min_length validator
```

**3. Async Operations**
For heavy operations (like sending emails), use background tasks:
```python
from django.core.mail import send_mail
from celery import shared_task

@shared_task
def send_welcome_email(user_id):
    user = User.objects.get(id=user_id)
    send_mail('Welcome', 'Thanks for registering', ...)
```

## Practice Exercises

### Exercise 1: Basic Registration Form

Create a basic registration view using UserCreationForm.

<details>
<summary>Solution</summary>

```python
# accounts/views.py
from django.shortcuts import render, redirect
from django.contrib.auth.forms import UserCreationForm

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('home')
    else:
        form = UserCreationForm()
    
    return render(request, 'accounts/register.html', {'form': form})
```
</details>

### Exercise 2: Add Email Field

Extend UserCreationForm to include an email field.

<details>
<summary>Solution</summary>

```python
# accounts/forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    email = forms.EmailField(required=True)

    class Meta:
        model = User
        fields = ('username', 'email')
```
</details>

### Exercise 3: Custom Widget Classes

Add CSS classes to all form fields using widgets.

<details>
<summary>Solution</summary>

```python
# accounts/forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    email = forms.EmailField(
        required=True,
        widget=forms.EmailInput(attrs={'class': 'form-control'})
    )
    password1 = forms.CharField(
        widget=forms.PasswordInput(attrs={'class': 'form-control'})
    )
    password2 = forms.CharField(
        widget=forms.PasswordInput(attrs={'class': 'form-control'})
    )

    class Meta:
        model = User
        fields = ('username', 'email')
        widgets = {
            'username': forms.TextInput(attrs={'class': 'form-control'})
        }
```
</details>

### Exercise 4: Manual Form Rendering

Create a template that manually renders the form fields.

<details>
<summary>Solution</summary>

```html
<!-- templates/accounts/register.html -->
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
    <button type="submit">Register</button>
</form>
```
</details>

### Exercise 5: Required Fields

Make first_name and last_name required fields in the registration form.

<details>
<summary>Solution</summary>

```python
# accounts/forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class RegisterForm(UserCreationForm):
    email = forms.EmailField(required=True)
    first_name = forms.CharField(required=True)
    last_name = forms.CharField(required=True)

    class Meta:
        model = User
        fields = ('username', 'email', 'first_name', 'last_name')
```
</details>

## Next Steps

Now that you understand user registration, the next step is to learn how to implement login functionality.

Continue to **[003-login-functionality.md](003-login-functionality.md)** to learn:
- Using AuthenticationForm for login
- authenticate() function for credential verification
- login() function for session creation
- Customizing AuthenticationForm
- Understanding session data structure
- Best practices for login implementation
