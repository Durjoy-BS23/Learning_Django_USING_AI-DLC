# Creating Custom Middlewares

## Introduction

Creating custom middlewares allows you to implement specific functionality that applies globally to your Django application. Whether you need to add custom headers, implement rate limiting, or modify requests/responses, custom middlewares provide a clean, reusable way to handle cross-cutting concerns. This guide covers both function-based and class-based middleware approaches, how to return responses from middleware, and the importance of middleware ordering.

## Concept Explanation

### Function-Based Middleware

Function-based middleware is the simpler approach to creating middleware. It's a function that takes `get_response` as a parameter and returns another function that processes the request.

**Structure:**
```python
def my_middleware(get_response):
    # One-time initialization code (runs when server starts)
    def middleware(request):
        # Code here runs BEFORE the view
        response = get_response(request)
        # Code here runs AFTER the view
        return response
    return middleware
```

**Key Points:**
- `get_response` is a callable that represents the next middleware or the view
- The outer function runs once when Django starts (initialization)
- The inner function runs for every request
- Code before `get_response(request)` executes before the view
- Code after `get_response(request)` executes after the view

### Class-Based Middleware

Class-based middleware provides more structure and is the preferred approach for complex middlewares. It uses Python classes with special methods to handle the request/response cycle.

**Structure:**
```python
class MyMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # One-time initialization code
    
    def __call__(self, request):
        # Code here runs BEFORE the view
        response = self.get_response(request)
        # Code here runs AFTER the view
        return response
```

**Key Points:**
- `__init__` runs once when Django starts (initialization)
- `__call__` runs for every request
- Store `get_response` as an instance variable
- Class-based middleware supports additional hooks (covered in next guide)

### Function vs Class-Based Middleware

| Feature | Function-Based | Class-Based |
|---------|----------------|-------------|
| Simplicity | Simpler for basic use cases | More structured |
| Initialization | Outer function | `__init__` method |
| Request handling | Inner function | `__call__` method |
| Middleware hooks | Not available | Available (process_view, etc.) |
| State management | Limited | Full class capabilities |
| Recommended for | Simple middleware | Complex middleware |

### Registering Middleware

To use a custom middleware, you must register it in `settings.py`:

```python
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    # ... other middlewares
    'myapp.middleware.my_custom_middleware',  # Your custom middleware
]
```

**Path Format:** `app_name.module_name.middleware_name`

### Returning Responses from Middleware

Middlewares can return responses directly without calling the next middleware or view. This is useful for:
- Blocking unauthorized access
- Rate limiting
- Maintenance mode
- Redirects

**Function-Based:**
```python
def blocking_middleware(get_response):
    def middleware(request):
        if some_condition:
            return HttpResponse("Blocked", status=403)
        response = get_response(request)
        return response
    return middleware
```

**Class-Based:**
```python
class BlockingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        if some_condition:
            return HttpResponse("Blocked", status=403)
        response = self.get_response(request)
        return response
```

### Middleware Ordering Importance

The order of middlewares in `MIDDLEWARE` list is critical:
- Request passes through middlewares in the order listed
- Response passes through in reverse order
- Earlier middlewares can block later ones
- Dependencies matter (e.g., Session before Authentication)

**Example:**
```python
MIDDLEWARE = [
    'middleware1',  # First to see request, last to see response
    'middleware2',  # Second to see request, second to see response
    'middleware3',  # Last to see request, first to see response
]
```

If `middleware2` returns a response, `middleware3` and the view are never called.

## Code Examples

### Function-Based Middleware

**middleware.py**
```python
def simple_middleware(get_response):
    # One-time initialization (runs when server starts)
    print("Simple middleware initialized")
    
    def middleware(request):
        # Code runs BEFORE the view
        print(f"Request to: {request.path}")
        
        # Pass request to next middleware or view
        response = get_response(request)
        
        # Code runs AFTER the view
        print(f"Response status: {response.status_code}")
        
        return response
    
    return middleware
```

**settings.py**
```python
MIDDLEWARE = [
    # ... built-in middlewares
    'myapp.middleware.simple_middleware',
]
```

### Class-Based Middleware

**middleware.py**
```python
class SimpleMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # One-time initialization (runs when server starts)
        print("Simple middleware initialized")
    
    def __call__(self, request):
        # Code runs BEFORE the view
        print(f"Request to: {request.path}")
        
        # Pass request to next middleware or view
        response = self.get_response(request)
        
        # Code runs AFTER the view
        print(f"Response status: {response.status_code}")
        
        return response
```

**settings.py**
```python
MIDDLEWARE = [
    # ... built-in middlewares
    'myapp.middleware.SimpleMiddleware',
]
```

### Path-Based Conditional Middleware

**middleware.py**
```python
class PathSpecificMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Only process /api/ endpoints
        if request.path.startswith('/api/'):
            print(f"API request: {request.path}")
            response = self.get_response(request)
            response['X-API-Version'] = '1.0'
            return response
        
        return self.get_response(request)
```

### Authentication Check Middleware

**middleware.py**
```python
from django.http import HttpResponse
from django.conf import settings

class AuthCheckMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Check authentication for specific paths
        if request.path.startswith('/protected/'):
            if not hasattr(request, 'user') or not request.user.is_authenticated:
                return HttpResponse("Authentication required", status=401)
        
        response = self.get_response(request)
        return response
```

### Rate Limiting Middleware

**middleware.py**
```python
from django.http import HttpResponse
from django.core.cache import cache
import time

class RateLimitMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Get client IP
        ip = request.META.get('REMOTE_ADDR')
        cache_key = f'rate_limit_{ip}'
        
        # Check rate limit (100 requests per minute)
        request_count = cache.get(cache_key, 0)
        
        if request_count >= 100:
            return HttpResponse("Rate limit exceeded", status=429)
        
        # Increment counter
        cache.set(cache_key, request_count + 1, 60)
        
        response = self.get_response(request)
        return response
```

### Adding Custom Headers

**middleware.py**
```python
class CustomHeaderMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        
        # Add custom headers
        response['X-Powered-By'] = 'MyApp'
        response['X-Response-Time'] = str(int(time.time()))
        
        return response
```

### Request Timing Middleware

**middleware.py**
```python
import time

class TimingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Record start time
        start_time = time.time()
        
        # Process request
        response = self.get_response(request)
        
        # Calculate duration
        duration = time.time() - start_time
        
        # Add timing header
        response['X-Process-Time'] = f"{duration:.3f}s"
        
        return response
```

### Multiple Middlewares with Ordering

**middleware.py**
```python
def first_middleware(get_response):
    def middleware(request):
        print("First: Before view")
        response = get_response(request)
        print("First: After view")
        return response
    return middleware

def second_middleware(get_response):
    def middleware(request):
        print("Second: Before view")
        response = get_response(request)
        print("Second: After view")
        return response
    return middleware
```

**settings.py**
```python
MIDDLEWARE = [
    'myapp.middleware.first_middleware',
    'myapp.middleware.second_middleware',
]
```

**Output when accessing a view:**
```
First: Before view
Second: Before view
View executed
Second: After view
First: After view
```

### Short-Circuit Middleware Example

**middleware.py**
```python
from django.http import HttpResponse
from django.conf import settings

class MaintenanceMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Check if maintenance mode is enabled
        if getattr(settings, 'MAINTENANCE_MODE', False):
            # Return maintenance page without calling view
            return HttpResponse(
                "Site under maintenance. Please try again later.",
                status=503
            )
        
        response = self.get_response(request)
        return response
```

**settings.py**
```python
MAINTENANCE_MODE = True

MIDDLEWARE = [
    'myapp.middleware.MaintenanceMiddleware',
    # ... other middlewares
]
```

## Key Takeaways

- Function-based middleware uses nested functions with `get_response` parameter
- Class-based middleware uses `__init__` and `__call__` methods
- Class-based middleware supports additional hooks (covered in next guide)
- Register middleware in `settings.py` using the full path
- Middleware order matters for request/response processing
- Middlewares can return responses directly to short-circuit the chain
- Use function-based for simple middleware, class-based for complex ones
- Code before `get_response(request)` runs before the view
- Code after `get_response(request)` runs after the view
- Initialization code runs once when the server starts

## Additional Context & Best Practices

### Middleware Best Practices

**1. Choose the Right Type**
```python
# ✅ Use function-based for simple cases
def simple_header_middleware(get_response):
    def middleware(request):
        response = get_response(request)
        response['X-Custom-Header'] = 'Value'
        return response
    return middleware

# ✅ Use class-based for complex cases
class ComplexAuthMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.rate_limiter = RateLimiter()
    
    def __call__(self, request):
        # Complex logic with instance variables
        if self.rate_limiter.is_allowed(request):
            return self.get_response(request)
        return HttpResponse("Rate limited", status=429)
```

**2. Keep Initialization Fast**
```python
# ✅ GOOD - Fast initialization
class FastMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # Simple assignment only

# ❌ BAD - Slow initialization (blocks server startup)
class SlowMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # Heavy computation or database queries
        self.cache = load_large_database()  # Slow!
```

**3. Handle Errors Gracefully**
```python
# ✅ GOOD - Handle exceptions
class RobustMiddleware:
    def __call__(self, request):
        try:
            response = self.get_response(request)
        except Exception as e:
            logger.error(f"Middleware error: {e}")
            response = HttpResponse("Internal error", status=500)
        return response

# ❌ BAD - Unhandled exceptions crash the app
class FragileMiddleware:
    def __call__(self, request):
        response = self.get_response(request)  # Might crash
        return response
```

### Common Pitfalls

**1. Forgetting to Return Response**
```python
# ❌ WRONG - No return statement
def broken_middleware(get_response):
    def middleware(request):
        response = get_response(request)
        # Forgot to return!
    return middleware

# ✅ CORRECT - Always return response
def working_middleware(get_response):
    def middleware(request):
        response = get_response(request)
        return response
    return middleware
```

**2. Incorrect Middleware Path**
```python
# ❌ WRONG - Incorrect path format
MIDDLEWARE = [
    'middleware.my_middleware',  # Missing app name
]

# ✅ CORRECT - Full path
MIDDLEWARE = [
    'myapp.middleware.my_middleware',
]
```

**3. Breaking Middleware Dependencies**
```python
# ❌ WRONG - Authentication before Session
MIDDLEWARE = [
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
]

# ✅ CORRECT - Session before Authentication
MIDDLEWARE = [
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
]
```

**4. Blocking All Requests**
```python
# ❌ DANGEROUS - Blocks everything
def block_all(get_response):
    def middleware(request):
        return HttpResponse("Blocked")  # Never calls get_response
    return middleware

# ✅ CORRECT - Conditional blocking
def block_specific(get_response):
    def middleware(request):
        if request.path.startswith('/admin/'):
            return HttpResponse("Blocked")
        return get_response(request)
    return middleware
```

### Performance Considerations

**1. Minimize Processing Time**
```python
# ✅ GOOD - Fast checks
def fast_middleware(get_response):
    def middleware(request):
        if request.method == 'POST':
            response = get_response(request)
            response['X-Post-Request'] = 'True'
            return response
        return get_response(request)
    return middleware

# ❌ BAD - Slow processing
def slow_middleware(get_response):
    def middleware(request):
        # Expensive operation on every request
        data = expensive_database_query()
        response = get_response(request)
        return response
    return middleware
```

**2. Use Caching**
```python
from django.core.cache import cache

class CachedMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.config = cache.get('middleware_config')
        if not self.config:
            self.config = load_config()
            cache.set('middleware_config', self.config, 3600)
```

**3. Conditional Execution**
```python
class ConditionalMiddleware:
    def __call__(self, request):
        # Only process specific paths
        if not request.path.startswith('/api/'):
            return self.get_response(request)
        
        # Process API requests only
        # ... middleware logic
        return response
```

### Advanced Tips

**1. Configurable Middleware**
```python
class ConfigurableMiddleware:
    def __init__(self, get_response, custom_header='X-Custom'):
        self.get_response = get_response
        self.custom_header = custom_header
    
    def __call__(self, request):
        response = self.get_response(request)
        response[self.custom_header] = 'Value'
        return response
```

**2. Middleware with State**
```python
class StatefulMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.request_count = 0
    
    def __call__(self, request):
        self.request_count += 1
        response = self.get_response(request)
        response['X-Request-Count'] = str(self.request_count)
        return response
```

**3. Debugging Middleware**
```python
class DebugMiddleware:
    def __call__(self, request):
        print(f"\n=== Request ===")
        print(f"Method: {request.method}")
        print(f"Path: {request.path}")
        print(f"User: {getattr(request, 'user', 'Anonymous')}")
        
        response = self.get_response(request)
        
        print(f"\n=== Response ===")
        print(f"Status: {response.status_code}")
        print(f"Content-Type: {response.get('Content-Type')}")
        
        return response
```

## Practice Exercises

### Exercise 1: Create Function-Based Middleware

Create a function-based middleware that prints "Before view" before the view is called and "After view" after the view is called.

<details>
<summary>Solution</summary>

```python
# middleware.py
def simple_logging_middleware(get_response):
    def middleware(request):
        print("Before view")
        response = get_response(request)
        print("After view")
        return response
    return middleware

# settings.py
MIDDLEWARE = [
    'myapp.middleware.simple_logging_middleware',
]
```
</details>

### Exercise 2: Create Class-Based Middleware

Convert the function-based middleware from Exercise 1 to class-based middleware.

<details>
<summary>Solution</summary>

```python
# middleware.py
class SimpleLoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        print("Before view")
        response = self.get_response(request)
        print("After view")
        return response

# settings.py
MIDDLEWARE = [
    'myapp.middleware.SimpleLoggingMiddleware',
]
```
</details>

### Exercise 3: Add Custom Header Middleware

Create a middleware that adds a custom header `X-Application: MyApp` to all responses.

<details>
<summary>Solution</summary>

```python
# middleware.py
class CustomHeaderMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        response['X-Application'] = 'MyApp'
        return response
```
</details>

### Exercise 4: Block Admin Access

Create a middleware that blocks access to URLs starting with `/admin/` and returns a 403 response.

<details>
<summary>Solution</summary>

```python
# middleware.py
from django.http import HttpResponse

class AdminBlockMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        if request.path.startswith('/admin/'):
            return HttpResponse("Admin access blocked", status=403)
        response = self.get_response(request)
        return response
```
</details>

### Exercise 5: Request Counter Middleware

Create a class-based middleware that counts the total number of requests processed and adds this count as a header `X-Request-Count` to responses.

<details>
<summary>Solution</summary>

```python
# middleware.py
class RequestCounterMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.request_count = 0
    
    def __call__(self, request):
        self.request_count += 1
        response = self.get_response(request)
        response['X-Request-Count'] = str(self.request_count)
        return response
```
</details>

### Exercise 6: Method-Specific Middleware

Create a middleware that only processes POST requests and adds a header `X-Post-Request: True` to responses.

<details>
<summary>Solution</summary>

```python
# middleware.py
class PostOnlyMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        if request.method == 'POST':
            response['X-Post-Request'] = 'True'
        return response
```
</details>

## Next Steps

Now that you know how to create custom middlewares, the next step is to learn about advanced middleware hooks that provide even more control over the request/response cycle.

Continue to **[003-advanced-middleware-hooks.md](003-advanced-middleware-hooks.md)** to learn:
- `process_view` hook (modify behavior before view execution)
- `process_exception` hook (handle exceptions globally)
- `process_template_response` hook (modify template context)
- When to use each hook
- Practical examples of advanced middleware hooks
