# Advanced Session Methods

## Introduction

While basic session operations cover most use cases, Django provides advanced methods for managing session lifecycle, expiration, and complex data structures. Understanding these methods allows you to implement sophisticated session management, handle expired sessions efficiently, and work with nested data structures safely. This guide covers flush method, session expiration methods, and the session modified flag.

## Concept Explanation

### flush() Method

The `flush()` method completely deletes the session:
- Deletes all session data from the database
- Deletes the session ID cookie from the client's browser
- Removes the entire session row from `django_session` table

**When to Use:**
- User logout (complete session cleanup)
- Session invalidation (security concerns)
- Starting fresh session

**Difference from `del`:**
- `del request.session['key']` - Deletes only specific data, session and cookie remain
- `request.session.flush()` - Deletes entire session including cookie and database row

**Example:**
```python
def logout(request):
    request.session.flush()  # Complete session deletion
    return HttpResponse("Logged out")
```

### set_expiry() Method

The `set_expiry()` method sets custom expiration time for the session:

**Possible Values:**
- **Integer**: Seconds until expiration (e.g., `set_expiry(300)` for 5 minutes)
- **datetime object**: Specific expiration date/time
- **0**: Session expires when browser closes
- **None**: Uses default expiration (2 weeks from settings)

**Examples:**
```python
# Expire after 1 hour
request.session.set_expiry(3600)

# Expire on specific date
from datetime import datetime
request.session.set_expiry(datetime(2024, 12, 31))

# Expire when browser closes
request.session.set_expiry(0)

# Use default expiration
request.session.set_expiry(None)
```

**Important Notes:**
- Affects current session only
- Overrides `SESSION_COOKIE_AGE` setting
- Must be called after setting session data

### clear_expired() Method

The `clear_expired()` method removes all expired sessions from the database:

**Why It's Needed:**
- Expired sessions remain in database until cleanup
- Session cookie expires, but database row persists
- Accumulates over time, causing database bloat
- Manual cleanup required for performance

**When to Call:**
- Periodically (e.g., daily cron job)
- After session operations
- During maintenance tasks

**Example:**
```python
def cleanup_sessions(request):
    from django.contrib.sessions.backends.db import SessionStore
    SessionStore.clear_expired()
    return HttpResponse("Expired sessions cleaned")
```

**Note:** This is a class method, not called on `request.session`.

### get_expiry_age() Method

The `get_expiry_age()` method returns the number of seconds until session expiration:

**Returns:**
- Seconds remaining until expiration
- Default expiration time if not set
- Useful for displaying session timeout to users

**Example:**
```python
def check_expiry(request):
    seconds_left = request.session.get_expiry_age()
    return HttpResponse(f"Session expires in {seconds_left} seconds")
```

### get_expiry_date() Method

The `get_expiry_date()` method returns the datetime when the session will expire:

**Returns:**
- `datetime` object of expiration
- `None` if session has no expiration (browser close)

**Example:**
```python
def expiry_info(request):
    expiry_date = request.session.get_expiry_date()
    return HttpResponse(f"Session expires at: {expiry_date}")
```

### session.modified Flag

The `session.modified` flag tells Django that session data has been modified:

**When Needed:**
- Modifying nested data structures (lists, dicts)
- Changing mutable objects stored in session
- Django doesn't detect nested changes automatically

**How It Works:**
- Django tracks changes to `request.session` dictionary keys
- Doesn't track changes to objects stored in session
- Set `request.session.modified = True` to force save

**Example:**
```python
def update_cart(request):
    # ❌ WON'T SAVE - Django doesn't detect nested change
    request.session['cart'].append(item)
    
    # ✅ WILL SAVE - Tell Django data changed
    request.session['cart'].append(item)
    request.session.modified = True
```

**When NOT Needed:**
- Setting new keys: `request.session['key'] = value`
- Replacing keys: `request.session['key'] = new_value`
- Deleting keys: `del request.session['key']`

## Code Examples

### Complete Session Deletion with flush()

**views.py**
```python
from django.http import HttpResponse

def logout(request):
    # Delete complete session
    request.session.flush()
    return HttpResponse("Logged out - session completely deleted")

def check_session(request):
    if 'user_id' in request.session:
        return HttpResponse("User is logged in")
    return HttpResponse("No active session")
```

**urls.py**
```python
urlpatterns = [
    path('logout/', views.logout, name='logout'),
    path('check/', views.check_session, name='check_session'),
]
```

### Custom Session Expiration

**views.py**
```python
from datetime import datetime, timedelta

def short_lived_session(request):
    # Set session data
    request.session['temp_data'] = 'temporary'
    
    # Expire after 5 minutes (300 seconds)
    request.session.set_expiry(300)
    
    return HttpResponse("Session set with 5-minute expiration")

def browser_close_session(request):
    # Set session data
    request.session['data'] = 'value'
    
    # Expire when browser closes
    request.session.set_expiry(0)
    
    return HttpResponse("Session expires on browser close")

def specific_date_session(request):
    # Set session data
    request.session['data'] = 'value'
    
    # Expire on specific date
    expiry_date = datetime.now() + timedelta(days=7)
    request.session.set_expiry(expiry_date)
    
    return HttpResponse(f"Session expires on {expiry_date}")

def default_expiration(request):
    # Set session data
    request.session['data'] = 'value'
    
    # Use default expiration (from settings)
    request.session.set_expiry(None)
    
    return HttpResponse("Session uses default expiration")
```

### Checking Session Expiration

**views.py**
```python
def session_info(request):
    # Get seconds until expiration
    seconds_left = request.session.get_expiry_age()
    
    # Get expiration date
    expiry_date = request.session.get_expiry_date()
    
    return HttpResponse(
        f"Expires in: {seconds_left} seconds<br>"
        f"Expiration date: {expiry_date}"
    )
```

### Cleaning Up Expired Sessions

**views.py**
```python
from django.contrib.sessions.backends.db import SessionStore

def cleanup_expired_sessions(request):
    # Remove all expired sessions from database
    SessionStore.clear_expired()
    return HttpResponse("Expired sessions cleaned up")

def cleanup_on_logout(request):
    # Clean expired sessions when user logs out
    request.session.flush()
    SessionStore.clear_expired()
    return HttpResponse("Logged out and cleaned expired sessions")
```

### Session Modified Flag for Nested Data

**views.py**
```python
def update_cart_without_flag(request):
    # Initialize cart if not exists
    if 'cart' not in request.session:
        request.session['cart'] = []
    
    # ❌ WON'T SAVE - Django doesn't detect list modification
    request.session['cart'].append({'product': 'Item', 'price': 10.99})
    
    return HttpResponse("Item added (but won't persist)")

def update_cart_with_flag(request):
    # Initialize cart if not exists
    if 'cart' not in request.session:
        request.session['cart'] = []
    
    # ✅ WILL SAVE - Tell Django data changed
    request.session['cart'].append({'product': 'Item', 'price': 10.99})
    request.session.modified = True
    
    return HttpResponse("Item added and saved")

def update_nested_dict(request):
    # Initialize preferences
    if 'preferences' not in request.session:
        request.session['preferences'] = {'theme': 'light', 'language': 'en'}
    
    # Modify nested dictionary
    request.session['preferences']['theme'] = 'dark'
    request.session.modified = True  # Required for nested changes
    
    return HttpResponse("Preferences updated")
```

### Combining flush() and clear_expired()

**views.py**
```python
from django.contrib.sessions.backends.db import SessionStore

def secure_logout(request):
    # Delete current session
    request.session.flush()
    
    # Clean up all expired sessions
    SessionStore.clear_expired()
    
    return HttpResponse("Secure logout complete")

def maintenance_cleanup(request):
    # Clean expired sessions without affecting current session
    SessionStore.clear_expired()
    return HttpResponse("Expired sessions cleaned")
```

### Session Expiration Warning

**views.py**
```python
def check_session_timeout(request):
    if 'user_id' not in request.session:
        return HttpResponse("Not logged in")
    
    seconds_left = request.session.get_expiry_age()
    
    if seconds_left < 300:  # Less than 5 minutes
        return HttpResponse(f"Warning: Session expires in {seconds_left} seconds")
    
    return HttpResponse(f"Session active: {seconds_left} seconds remaining")
```

### Dynamic Session Expiration

**views.py**
```python
def extended_session(request):
    # Check if user is premium
    is_premium = request.session.get('is_premium', False)
    
    # Set longer expiration for premium users
    if is_premium:
        request.session.set_expiry(86400)  # 24 hours
    else:
        request.session.set_expiry(3600)   # 1 hour
    
    return HttpResponse("Session expiration set based on user type")
```

### Session Reset with flush()

**views.py**
```python
def reset_session(request):
    # Save current session data before flush
    saved_data = dict(request.session)
    
    # Flush complete session
    request.session.flush()
    
    # Restore specific data
    request.session['user_id'] = saved_data.get('user_id')
    
    return HttpResponse("Session reset with preserved data")
```

### Batch Session Operations

**views.py**
```python
def batch_session_update(request):
    # Initialize session data
    request.session['cart'] = []
    request.session['preferences'] = {}
    request.session['history'] = []
    
    # Modify all nested structures
    request.session['cart'].append({'id': 1, 'name': 'Item'})
    request.session['preferences']['theme'] = 'dark'
    request.session['history'].append({'action': 'login', 'time': 'now'})
    
    # Mark as modified once
    request.session.modified = True
    
    return HttpResponse("Batch session update complete")
```

## Key Takeaways

- `flush()` deletes complete session including cookie and database row
- `del` only deletes specific session data, session persists
- `set_expiry()` sets custom expiration time for current session
- `clear_expired()` removes expired sessions from database
- `get_expiry_age()` returns seconds until session expiration
- `get_expiry_date()` returns datetime of session expiration
- `session.modified` flag required for nested data changes
- Django doesn't automatically detect changes to mutable objects in session
- Use `flush()` for logout and complete session cleanup
- Use `clear_expired()` periodically to prevent database bloat

## Additional Context & Best Practices

### Session Expiration Best Practices

**1. Set Appropriate Expiration Times**
```python
# ✅ GOOD - Short expiration for sensitive operations
def sensitive_operation(request):
    request.session.set_expiry(300)  # 5 minutes
    # ... perform operation

# ❌ BAD - Too long for sensitive data
def sensitive_operation(request):
    request.session.set_expiry(86400)  # 24 hours
```

**2. Use Browser-Close for Temporary Data**
```python
# ✅ GOOD - Expire on browser close for temporary data
def temp_form_data(request):
    request.session['form_data'] = data
    request.session.set_expiry(0)
```

**3. Extend Expiration for Active Users**
```python
def refresh_session(request):
    # Extend session on user activity
    request.session.set_expiry(3600)  # Reset to 1 hour
    return HttpResponse("Session refreshed")
```

### Cleanup Best Practices

**1. Regular Cleanup Schedule**
```python
# ✅ GOOD - Scheduled cleanup (e.g., cron job)
# Run daily: python manage.py shell -c "from django.contrib.sessions.backends.db import SessionStore; SessionStore.clear_expired()"

# ✅ GOOD - Cleanup on logout
def logout(request):
    request.session.flush()
    SessionStore.clear_expired()
    return redirect('login')
```

**2. Monitor Session Table Size**
```python
# Check session table size
from django.contrib.sessions.models import Session
session_count = Session.objects.count()
print(f"Total sessions: {session_count}")
```

### Session Modified Best Practices

**1. Always Set Modified for Nested Changes**
```python
# ✅ GOOD - Always set modified flag
def update_nested(request):
    request.session['data']['key'] = 'value'
    request.session.modified = True

# ❌ BAD - Forgetting the flag
def update_nested(request):
    request.session['data']['key'] = 'value'  # Won't save
```

**2. Use setdefault for Initialization**
```python
# ✅ GOOD - Initialize and modify in one step
def update_cart(request):
    cart = request.session.setdefault('cart', [])
    cart.append(item)
    request.session.modified = True
```

**3. Consider Replacing Instead of Modifying**
```python
# Alternative to modified flag
def update_preferences(request):
    # Replace entire object instead of modifying
    preferences = request.session.get('preferences', {})
    preferences['theme'] = 'dark'
    request.session['preferences'] = preferences  # No modified flag needed
```

### Common Pitfalls

**1. Forgetting flush() on Logout**
```python
# ❌ WRONG - Only deletes data, session persists
def logout(request):
    del request.session['user_id']
    del request.session['logged_in']

# ✅ CORRECT - Complete session deletion
def logout(request):
    request.session.flush()
```

**2. Not Cleaning Expired Sessions**
```python
# ❌ WRONG - Never cleaning expired sessions
# Database grows indefinitely

# ✅ CORRECT - Regular cleanup
def logout(request):
    request.session.flush()
    SessionStore.clear_expired()
```

**3. Ignoring Nested Data Changes**
```python
# ❌ WRONG - Changes won't persist
def add_to_cart(request):
    request.session['cart'].append(item)

# ✅ CORRECT - Mark as modified
def add_to_cart(request):
    request.session['cart'].append(item)
    request.session.modified = True
```

**4. Setting Expiration After Session Data**
```python
# ❌ WRONG - Order matters
def set_session(request):
    request.session.set_expiry(3600)
    request.session['data'] = 'value'

# ✅ CORRECT - Set data first
def set_session(request):
    request.session['data'] = 'value'
    request.session.set_expiry(3600)
```

### Performance Considerations

**1. Batch Session Operations**
```python
# ✅ GOOD - Set modified once after multiple changes
def batch_update(request):
    request.session['cart'].append(item1)
    request.session['cart'].append(item2)
    request.session['preferences']['theme'] = 'dark'
    request.session.modified = True  # Set once

# ❌ BAD - Set modified multiple times
def batch_update(request):
    request.session['cart'].append(item1)
    request.session.modified = True
    request.session['cart'].append(item2)
    request.session.modified = True
```

**2. Use Appropriate Session Backend**
```python
# For high-traffic sites
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

# For most applications
SESSION_ENGINE = 'django.contrib.sessions.backends.db'
```

**3. Minimize Session Data Size**
```python
# ✅ GOOD - Store minimal data
request.session['user_id'] = user.id

# ❌ BAD - Store large objects
request.session['user'] = user  # Includes all fields
```

### Security Considerations

**1. Flush on Security Events**
```python
def security_breach_detected(request):
    # Complete session invalidation
    request.session.flush()
    return HttpResponse("Session invalidated for security")
```

**2. Short Expiration for Sensitive Data**
```python
def access_sensitive_data(request):
    # Verify user
    if not request.user.is_staff:
        return HttpResponse("Access denied", status=403)
    
    # Set short expiration
    request.session.set_expiry(300)  # 5 minutes
    return sensitive_data
```

**3. Regenerate Session on Privilege Change**
```python
def upgrade_privileges(request):
    # Change user role
    request.user.is_staff = True
    request.user.save()
    
    # Flush old session to prevent session fixation
    request.session.flush()
    
    # Create new session
    request.session['logged_in'] = True
    return HttpResponse("Privileges upgraded")
```

## Practice Exercises

### Exercise 1: Complete Session Deletion

Create a logout view that completely deletes the session using `flush()`.

<details>
<summary>Solution</summary>

```python
# views.py
def logout(request):
    request.session.flush()
    return HttpResponse("Logged out - session deleted")

# urls.py
urlpatterns = [
    path('logout/', views.logout, name='logout'),
]
```
</details>

### Exercise 2: Custom Session Expiration

Create a view that sets session data with a 10-minute expiration.

<details>
<summary>Solution</summary>

```python
# views.py
def set_short_session(request):
    request.session['data'] = 'temporary data'
    request.session.set_expiry(600)  # 10 minutes
    return HttpResponse("Session set with 10-minute expiration")
```
</details>

### Exercise 3: Browser-Close Session

Create a view that sets session data that expires when the browser closes.

<details>
<summary>Solution</summary>

```python
# views.py
def set_temp_session(request):
    request.session['temp'] = 'data'
    request.session.set_expiry(0)  # Expires on browser close
    return HttpResponse("Session expires on browser close")
```
</details>

### Exercise 4: Check Session Expiration

Create a view that displays how many seconds remain until session expiration.

<details>
<summary>Solution</summary>

```python
# views.py
def check_expiry(request):
    seconds_left = request.session.get_expiry_age()
    return HttpResponse(f"Session expires in {seconds_left} seconds")
```
</details>

### Exercise 5: Update Nested Session Data

Create a view that adds an item to a shopping cart stored in session, properly using the modified flag.

<details>
<summary>Solution</summary>

```python
# views.py
def add_to_cart(request):
    cart = request.session.setdefault('cart', [])
    cart.append({'product': 'Item', 'price': 10.99})
    request.session.modified = True
    return HttpResponse("Item added to cart")
```
</details>

### Exercise 6: Clean Expired Sessions

Create a view that cleans up all expired sessions from the database.

<details>
<summary>Solution</summary>

```python
# views.py
from django.contrib.sessions.backends.db import SessionStore

def cleanup_sessions(request):
    SessionStore.clear_expired()
    return HttpResponse("Expired sessions cleaned up")
```
</details>

### Exercise 7: Session Expiration Warning

Create a view that warns the user if their session expires in less than 5 minutes.

<details>
<summary>Solution</summary>

```python
# views.py
def check_session_warning(request):
    seconds_left = request.session.get_expiry_age()
    
    if seconds_left < 300:
        return HttpResponse(f"Warning: Session expires in {seconds_left} seconds")
    
    return HttpResponse(f"Session active: {seconds_left} seconds remaining")
```
</details>

## Next Steps

Now that you understand advanced session methods, the next step is to learn about session settings for configuring session behavior globally.

Continue to **[003-session-settings.md](003-session-settings.md)** to learn:
- `SESSION_COOKIE_AGE` setting for default expiration
- `SESSION_COOKIE_NAME` for custom cookie names
- `SESSION_COOKIE_HTTPONLY` for security
- `SESSION_EXPIRE_AT_BROWSER_CLOSE` for browser-close sessions
- Other important session configuration options
- How to configure session backends
