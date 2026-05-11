# Django Sessions Fundamentals

## Introduction

Sessions are a fundamental mechanism for maintaining user state across HTTP requests in Django. Unlike cookies, which store data on the client's browser, sessions store data on the server, making them more secure and suitable for sensitive information like authentication data. Understanding sessions is essential for implementing user authentication, shopping carts, and personalized user experiences in Django applications.

## Concept Explanation

### What Are Sessions?

Sessions are a way to store information about a user across multiple requests. They work similarly to cookies but with a crucial difference: the actual data is stored on the server, not on the client's browser.

**Key Characteristics:**
- **Server-side storage**: Data stored in database or cache
- **Client-side identifier**: Browser stores a session ID cookie
- **Secure**: Actual data never exposed to client
- **Flexible**: Can store complex data structures
- **Temporary**: Sessions expire after a set time (default 2 weeks)

### Sessions vs Cookies

| Aspect | Cookies | Sessions |
|--------|---------|----------|
| Storage Location | Client browser | Server database/cache |
| Data Size | Limited (~4KB) | No strict limit |
| Security | Less secure (client can modify) | More secure (server-side) |
| Data Type | Strings only | Any serializable data |
| Expiration | Set per cookie | Global or per-session |
| Use Case | Preferences, tracking | Authentication, sensitive data |

### How Sessions Work

**Session Workflow:**

1. **First Request**: Client makes request to server
2. **Session Creation**: Server creates session data with unique session key
3. **Database Storage**: Server stores session data in `django_session` table
4. **Cookie Sent**: Server sends session ID cookie to client
5. **Subsequent Requests**: Client sends session ID cookie with each request
6. **Data Retrieval**: Server uses session ID to retrieve session data from database

**Visual Representation:**
```
Client Request → Server → Create Session Data → Store in Database
                     ↓
              Send sessionid Cookie to Client

Client Request (with sessionid) → Server → Retrieve Data from Database
                                          ↓
                                   Use Session Data
```

### Session ID and Session Key

- **Session ID**: The value stored in the client's cookie
- **Session Key**: The unique identifier in the database table
- **They are the same**: The cookie value matches the database session key
- **Purpose**: Links the client's cookie to the server's session data

### Django Session Table

Django stores session data in the `django_session` table with these columns:
- `session_key`: Unique identifier (matches cookie value)
- `session_data`: Encrypted session data (pickled and signed)
- `expire_date`: When the session expires

**Note**: Session data is encrypted, not plain text, for security.

### SessionMiddleware

Django provides `SessionMiddleware` which:
- Adds `request.session` attribute to every request
- Automatically handles session ID cookie management
- Saves session data when modified
- Loads session data from database on each request

**Required Settings:**
```python
INSTALLED_APPS = [
    # ...
    'django.contrib.sessions',
]

MIDDLEWARE = [
    # ...
    'django.contrib.sessions.middleware.SessionMiddleware',
]
```

### Setting Session Data

Use `request.session` like a dictionary to set session data:

```python
def set_session(request):
    request.session['name'] = 'John'
    request.session['user_id'] = 123
    return HttpResponse("Session set")
```

**Important:**
- Django automatically saves session when you set values
- Session is created only when you set data
- Session ID cookie is sent to client on first session data set

### Getting Session Data

Access session data using dictionary syntax:

```python
def get_session(request):
    name = request.session.get('name', 'Guest')
    return HttpResponse(f"Hello, {name}")
```

**Best Practice:** Use `.get()` with default to avoid `KeyError`.

### Updating Session Data

Simply assign a new value to update:

```python
def update_session(request):
    request.session['name'] = 'Jane'  # Updates existing value
    return HttpResponse("Session updated")
```

### Deleting Session Data

Use the `del` keyword to delete specific session data:

```python
def delete_session_data(request):
    if 'name' in request.session:
        del request.session['name']
    return HttpResponse("Session data deleted")
```

**Note:** This only deletes the data, not the complete session.

## Code Examples

### Basic Session Setup

**settings.py** (default Django project already has this):
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',  # Required for sessions
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',  # Required
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

### Setting Session Data

**views.py**
```python
from django.http import HttpResponse

def set_session(request):
    # Set session data
    request.session['user_name'] = 'John Doe'
    request.session['user_id'] = 123
    request.session['is_logged_in'] = True
    
    return HttpResponse("Session data set successfully")
```

**urls.py**
```python
from django.urls import path
from . import views

urlpatterns = [
    path('set/', views.set_session, name='set_session'),
]
```

### Getting Session Data

**views.py**
```python
def get_session(request):
    # Get session data with default values
    user_name = request.session.get('user_name', 'Guest')
    user_id = request.session.get('user_id', None)
    is_logged_in = request.session.get('is_logged_in', False)
    
    return HttpResponse(
        f"Name: {user_name}, ID: {user_id}, Logged in: {is_logged_in}"
    )
```

**urls.py**
```python
urlpatterns = [
    path('set/', views.set_session, name='set_session'),
    path('get/', views.get_session, name='get_session'),
]
```

### Updating Session Data

**views.py**
```python
def update_session(request):
    # Update existing session data
    request.session['user_name'] = 'Jane Smith'
    
    # Add new session data
    request.session['last_login'] = '2024-01-15'
    
    return HttpResponse("Session data updated")
```

**urls.py**
```python
urlpatterns = [
    path('set/', views.set_session, name='set_session'),
    path('get/', views.get_session, name='get_session'),
    path('update/', views.update_session, name='update_session'),
]
```

### Deleting Specific Session Data

**views.py**
```python
def delete_session_data(request):
    # Delete specific session key
    if 'user_name' in request.session:
        del request.session['user_name']
        return HttpResponse("User name deleted from session")
    
    return HttpResponse("User name not found in session")
```

### Checking Session Data Existence

**views.py**
```python
def check_session(request):
    # Check if session data exists
    if 'user_id' in request.session:
        return HttpResponse("User is logged in")
    else:
        return HttpResponse("User is not logged in")
```

### Session with Complex Data

**views.py**
```python
def set_complex_session(request):
    # Store complex data in session
    cart_items = [
        {'id': 1, 'name': 'Product 1', 'price': 10.99},
        {'id': 2, 'name': 'Product 2', 'price': 20.99},
    ]
    
    user_preferences = {
        'theme': 'dark',
        'language': 'en',
        'notifications': True
    }
    
    request.session['cart'] = cart_items
    request.session['preferences'] = user_preferences
    
    return HttpResponse("Complex session data set")

def get_complex_session(request):
    cart = request.session.get('cart', [])
    preferences = request.session.get('preferences', {})
    
    return HttpResponse(f"Cart: {len(cart)} items, Theme: {preferences.get('theme')}")
```

### Session-Based Counter

**views.py**
```python
def visit_counter(request):
    # Increment visit counter
    visits = request.session.get('visits', 0) + 1
    request.session['visits'] = visits
    
    return HttpResponse(f"Visit count: {visits}")
```

### User Login Session Example

**views.py**
```python
def login_view(request):
    # Simulate login
    user_data = {
        'id': 123,
        'name': 'John Doe',
        'email': 'john@example.com',
    }
    
    request.session['user'] = user_data
    request.session['logged_in'] = True
    request.session['login_time'] = str(datetime.now())
    
    return HttpResponse("Logged in successfully")

def profile_view(request):
    if not request.session.get('logged_in'):
        return HttpResponse("Please login first", status=401)
    
    user = request.session.get('user')
    return HttpResponse(f"Welcome, {user['name']}")
```

### Shopping Cart Session

**views.py**
```python
def add_to_cart(request, product_id):
    # Get cart or create new
    cart = request.session.get('cart', [])
    
    # Add product
    cart.append({
        'id': product_id,
        'quantity': 1,
    })
    
    # Save cart
    request.session['cart'] = cart
    
    return HttpResponse(f"Product {product_id} added to cart")

def view_cart(request):
    cart = request.session.get('cart', [])
    total_items = len(cart)
    
    return HttpResponse(f"Cart has {total_items} items")

def clear_cart(request):
    if 'cart' in request.session:
        del request.session['cart']
    
    return HttpResponse("Cart cleared")
```

## Key Takeaways

- Sessions store data on the server, not on the client
- Session ID cookie links client to server-side session data
- Session data is stored in `django_session` table
- Use `request.session` like a dictionary to work with sessions
- SessionMiddleware automatically handles session management
- Sessions are more secure than cookies for sensitive data
- Use `.get()` with default to avoid `KeyError`
- Use `del` to delete specific session data
- Django automatically saves session when you modify it
- Sessions expire after a set time (default 2 weeks)

## Additional Context & Best Practices

### Session Best Practices

**1. Store Minimal Data**
```python
# ✅ GOOD - Store only IDs, retrieve data from database
request.session['user_id'] = user.id

# ❌ BAD - Store entire user object
request.session['user'] = user  # Wasteful and can cause issues
```

**2. Use Safe Default Values**
```python
# ✅ GOOD - Use .get() with default
user_id = request.session.get('user_id', None)

# ❌ BAD - Direct access can raise KeyError
user_id = request.session['user_id']  # KeyError if not set
```

**3. Check Before Deleting**
```python
# ✅ GOOD - Check existence before deleting
if 'key' in request.session:
    del request.session['key']

# ❌ BAD - Can raise KeyError
del request.session['key']
```

**4. Don't Store Sensitive Data Unencrypted**
```python
# ✅ GOOD - Store references or encrypted data
request.session['user_id'] = user.id

# ❌ BAD - Store passwords or sensitive data
request.session['password'] = password  # Never do this
```

### Common Pitfalls

**1. Forgetting SessionMiddleware**
```python
# ❌ WRONG - SessionMiddleware missing
MIDDLEWARE = [
    'django.middleware.common.CommonMiddleware',
    # Missing SessionMiddleware
]

# ✅ CORRECT - Include SessionMiddleware
MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
]
```

**2. Modifying Nested Data Without Flag**
```python
# ❌ WRONG - Changes to nested data not saved
request.session['cart'].append(item)  # Won't save

# ✅ CORRECT - Use modified flag (covered in next guide)
request.session['cart'].append(item)
request.session.modified = True
```

**3. Assuming Session Exists**
```python
# ❌ WRONG - Assumes session exists
user_id = request.session['user_id']

# ✅ CORRECT - Check or use default
user_id = request.session.get('user_id', None)
```

**4. Storing Large Objects**
```python
# ❌ WRONG - Storing large querysets
request.session['all_users'] = User.objects.all()

# ✅ CORRECT - Store IDs or use pagination
request.session['user_ids'] = list(User.objects.values_list('id', flat=True)[:100])
```

### Performance Considerations

**1. Session Backend Choice**
- **Database-backed**: Default, good for most cases
- **Cache-backed**: Faster, suitable for high-traffic sites
- **File-based**: Not recommended for production
- **Cookie-based**: Limited size, not secure for sensitive data

**2. Session Size**
- Keep session data minimal
- Store IDs, not full objects
- Large sessions slow down requests
- Database storage can become a bottleneck

**3. Session Cleanup**
- Expired sessions accumulate in database
- Use `clear_expired()` method (covered in next guide)
- Set appropriate expiration times
- Consider scheduled cleanup jobs

### Security Considerations

**1. Session Cookie Security**
```python
# settings.py
SESSION_COOKIE_HTTPONLY = True  # Prevent JavaScript access
SESSION_COOKIE_SECURE = True     # Only send over HTTPS
SESSION_COOKIE_SAMESITE = 'Lax'  # CSRF protection
```

**2. Session Expiration**
- Set appropriate expiration times
- Shorter expiration for sensitive operations
- Use `set_expiry()` for custom expiration (covered in next guide)
- Implement logout functionality

**3. Session Fixation**
- Regenerate session ID on login
- Use Django's built-in authentication
- Don't accept session IDs from URLs
- Validate session on each request

### Advanced Tips

**1. Session Backends**
```python
# settings.py
# Database-backed (default)
SESSION_ENGINE = 'django.contrib.sessions.backends.db'

# Cache-backed (faster)
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
    }
}

# File-based (not recommended for production)
SESSION_ENGINE = 'django.contrib.sessions.backends.file'
SESSION_FILE_PATH = '/tmp/django_sessions'
```

**2. Session Serialization**
```python
# Only JSON-serializable data can be stored
request.session['data'] = {'key': 'value'}  # ✅ Works
request.session['date'] = datetime.now()   # ❌ Won't work directly

# Convert to string if needed
request.session['date'] = str(datetime.now())
```

**3. Session Keys Validation**
```python
def validate_session(request):
    # Check if session is valid
    if not request.session.exists(request.session.session_key):
        return HttpResponse("Invalid session", status=400)
    return HttpResponse("Valid session")
```

## Practice Exercises

### Exercise 1: Set and Get Session

Create a view that sets a session variable `username` to your name, and another view that retrieves and displays it.

<details>
<summary>Solution</summary>

```python
# views.py
from django.http import HttpResponse

def set_username(request):
    request.session['username'] = 'John Doe'
    return HttpResponse("Username set in session")

def get_username(request):
    username = request.session.get('username', 'Guest')
    return HttpResponse(f"Username: {username}")

# urls.py
urlpatterns = [
    path('set-username/', views.set_username, name='set_username'),
    path('get-username/', views.get_username, name='get_username'),
]
```
</details>

### Exercise 2: Visit Counter

Create a view that increments a visit counter in the session and displays the total visits.

<details>
<summary>Solution</summary>

```python
# views.py
def visit_counter(request):
    visits = request.session.get('visits', 0) + 1
    request.session['visits'] = visits
    return HttpResponse(f"Total visits: {visits}")
```
</details>

### Exercise 3: Shopping Cart

Create views to:
- Add a product to a cart (stored in session)
- View the cart contents
- Clear the cart

<details>
<summary>Solution</summary>

```python
# views.py
def add_to_cart(request):
    cart = request.session.get('cart', [])
    cart.append({'product': 'Item', 'price': 10.99})
    request.session['cart'] = cart
    return HttpResponse("Item added to cart")

def view_cart(request):
    cart = request.session.get('cart', [])
    return HttpResponse(f"Cart has {len(cart)} items")

def clear_cart(request):
    if 'cart' in request.session:
        del request.session['cart']
    return HttpResponse("Cart cleared")

# urls.py
urlpatterns = [
    path('add-to-cart/', views.add_to_cart, name='add_to_cart'),
    path('view-cart/', views.view_cart, name='view_cart'),
    path('clear-cart/', views.clear_cart, name='clear_cart'),
]
```
</details>

### Exercise 4: User Preferences

Create views to:
- Set user preferences (theme, language) in session
- Get and display preferences
- Update a specific preference

<details>
<summary>Solution</summary>

```python
# views.py
def set_preferences(request):
    request.session['preferences'] = {
        'theme': 'dark',
        'language': 'en',
    }
    return HttpResponse("Preferences set")

def get_preferences(request):
    preferences = request.session.get('preferences', {})
    theme = preferences.get('theme', 'light')
    language = preferences.get('language', 'en')
    return HttpResponse(f"Theme: {theme}, Language: {language}")

def update_theme(request, new_theme):
    preferences = request.session.get('preferences', {})
    preferences['theme'] = new_theme
    request.session['preferences'] = preferences
    return HttpResponse(f"Theme updated to {new_theme}")
```
</details>

### Exercise 5: Login Simulation

Create a simple login system using sessions:
- Login view that sets session data
- Profile view that checks if user is logged in
- Logout view that clears session data

<details>
<summary>Solution</summary>

```python
# views.py
def login(request):
    request.session['logged_in'] = True
    request.session['username'] = 'John'
    return HttpResponse("Logged in")

def profile(request):
    if not request.session.get('logged_in'):
        return HttpResponse("Please login first", status=401)
    username = request.session.get('username')
    return HttpResponse(f"Welcome, {username}")

def logout(request):
    if 'logged_in' in request.session:
        del request.session['logged_in']
    if 'username' in request.session:
        del request.session['username']
    return HttpResponse("Logged out")

# urls.py
urlpatterns = [
    path('login/', views.login, name='login'),
    path('profile/', views.profile, name='profile'),
    path('logout/', views.logout, name='logout'),
]
```
</details>

## Next Steps

Now that you understand the fundamentals of Django sessions, the next step is to learn about advanced session methods for managing session expiration, cleanup, and nested data modification.

Continue to **[002-advanced-session-methods.md](002-advanced-session-methods.md)** to learn:
- Using `flush()` to delete complete sessions
- Setting custom expiration with `set_expiry()`
- Cleaning up expired sessions with `clear_expired()`
- Checking expiration with `get_expiry_age()`
- Using `session.modified` for nested data changes
- When to use `del` vs `flush()`
