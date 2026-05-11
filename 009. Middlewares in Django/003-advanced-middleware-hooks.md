# Advanced Middleware Hooks

## Introduction

Class-based middleware in Django supports additional hooks that provide fine-grained control over the request/response cycle beyond the basic `__call__` method. These hooks allow you to execute code at specific points: just before the view is called, when exceptions occur, and after template responses are generated. Understanding these hooks enables you to implement sophisticated behavior like view-level authorization, global exception handling, and dynamic context injection.

## Concept Explanation

### What Are Middleware Hooks?

Middleware hooks are special methods that you can add to class-based middleware to execute code at specific points in the request/response lifecycle. Unlike the basic `__call__` method, hooks provide access to more context and allow you to intercept the process at critical moments.

**Key Points:**
- Hooks are only available in class-based middleware (not function-based)
- Each hook serves a specific purpose in the request/response cycle
- Hooks can return `None` to continue normal processing or an `HttpResponse` to short-circuit
- Hooks are called in a specific order during the middleware chain

### The Three Middleware Hooks

| Hook | When Called | Purpose |
|------|-------------|---------|
| `process_view` | Just before the view is called | Modify request, inspect view, or short-circuit |
| `process_exception` | When a view raises an exception | Handle exceptions globally |
| `process_template_response` | After view returns a TemplateResponse | Modify template context |

### process_view Hook

The `process_view` hook is called just before Django calls the view. It gives you access to the view function that will be called and its arguments.

**Signature:**
```python
def process_view(self, request, view_func, view_args, view_kwargs):
    # Code here
    return None  # or HttpResponse
```

**Parameters:**
- `request`: The HttpRequest object
- `view_func`: The view function Django will call
- `view_args`: List of positional arguments passed to the view
- `view_kwargs`: Dictionary of keyword arguments passed to the view

**Return Value:**
- `None`: Continue normal processing (call the view)
- `HttpResponse`: Short-circuit and return this response instead of calling the view

**Use Cases:**
- Inspect which view will be called
- Add request attributes based on the view
- Implement view-specific authorization
- Log which views are being accessed
- Modify request before it reaches the view

### process_exception Hook

The `process_exception` hook is called when a view raises an exception. It allows you to handle exceptions globally across all views.

**Signature:**
```python
def process_exception(self, request, exception):
    # Code here
    return None  # or HttpResponse
```

**Parameters:**
- `request`: The HttpRequest object
- `exception`: The Exception object raised by the view

**Return Value:**
- `None`: Continue normal exception handling (Django's default error pages)
- `HttpResponse`: Return this response instead of the error page

**Use Cases:**
- Custom error pages
- Exception logging
- Graceful error handling
- Send error notifications
- API error responses

### process_template_response Hook

The `process_template_response` hook is called after the view has finished executing, but only if the view returns a `TemplateResponse` (not an `HttpResponse`). This allows you to modify the template context before rendering.

**Signature:**
```python
def process_template_response(self, request, response):
    # Modify response.context_data
    return response
```

**Parameters:**
- `request`: The HttpRequest object
- `response`: The TemplateResponse object

**Return Value:**
- Must return the (possibly modified) TemplateResponse

**Important Notes:**
- Only called for `TemplateResponse`, not `HttpResponse`
- Can modify `response.context_data` to add/modify context variables
- Runs after the view but before the template is rendered
- Must return the response object

**Use Cases:**
- Add global context variables to all templates
- Modify context based on request
- Inject user-specific data
- Add debugging information

### Hook Execution Order

The hooks execute in this order during a request:

```
1. __call__ (before view)
2. process_view (just before view)
3. View executes
4. process_template_response (if TemplateResponse)
5. __call__ (after view)

If view raises exception:
3. View raises exception
4. process_exception
5. __call__ (after view)
```

### Why Class-Based Middleware?

The presence of hooks is a major advantage of class-based middleware over function-based middleware. This is why class-based middleware is the preferred approach for complex use cases.

**Class-Based Advantages:**
- Access to all three hooks
- Better structure and organization
- Instance variables for state management
- Easier to test and maintain

**Function-Based Limitations:**
- Only `__call__` method available
- Cannot use hooks
- Limited to before/after view logic

## Code Examples

### process_view Hook Example

**middleware.py**
```python
class ViewInspectorMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        # Print which view will be called
        view_name = view_func.__name__
        print(f"View to be called: {view_name}")
        print(f"Arguments: {view_args}")
        print(f"Keyword arguments: {view_kwargs}")
        
        # Continue normal processing
        return None
```

### process_view with Short-Circuit

**middleware.py**
```python
from django.http import HttpResponse

class AdminProtectionMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        # Block access to specific views
        view_name = view_func.__name__
        
        if view_name == 'admin_view':
            if not request.user.is_staff:
                return HttpResponse("Access denied", status=403)
        
        return None
```

### process_exception Hook Example

**middleware.py**
```python
import logging

logger = logging.getLogger(__name__)

class ExceptionLoggerMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_exception(self, request, exception):
        # Log the exception
        logger.error(
            f"Exception in {request.path}: {str(exception)}",
            exc_info=True
        )
        
        # Let Django handle the error page
        return None
```

### process_exception with Custom Response

**middleware.py**
```python
from django.http import JsonResponse

class APIExceptionHandlerMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_exception(self, request, exception):
        # Return JSON error for API requests
        if request.path.startswith('/api/'):
            return JsonResponse({
                'error': str(exception),
                'status': 'error'
            }, status=500)
        
        return None
```

### process_template_response Hook Example

**middleware.py**
```python
class GlobalContextMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_template_response(self, request, response):
        # Add global context to all templates
        response.context_data['current_year'] = 2024
        response.context_data['site_name'] = 'My Site'
        
        # Add user information if available
        if hasattr(request, 'user'):
            response.context_data['current_user'] = request.user
        
        return response
```

### View Returning TemplateResponse

**views.py**
```python
from django.template.response import TemplateResponse

def home(request):
    # Must return TemplateResponse, not HttpResponse
    return TemplateResponse(request, 'home.html', {'title': 'Home'})
```

### Combined Hooks Example

**middleware.py**
```python
import logging
from django.http import HttpResponse

logger = logging.getLogger(__name__)

class ComprehensiveMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        print(f"__call__: Request to {request.path}")
        response = self.get_response(request)
        print(f"__call__: Response status {response.status_code}")
        return response
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        view_name = view_func.__name__
        print(f"process_view: About to call {view_name}")
        return None
    
    def process_exception(self, request, exception):
        print(f"process_exception: Exception occurred: {str(exception)}")
        logger.error(f"Exception in {request.path}", exc_info=True)
        return None
    
    def process_template_response(self, request, response):
        print("process_template_response: Modifying context")
        response.context_data['global_message'] = 'Hello from middleware'
        return response
```

### View-Specific Authorization with process_view

**middleware.py**
```python
from django.http import HttpResponse

class ViewAuthMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        # Define restricted views
        restricted_views = ['delete_view', 'update_view']
        view_name = view_func.__name__
        
        # Check if view is restricted
        if view_name in restricted_views:
            if not request.user.is_authenticated:
                return HttpResponse("Authentication required", status=401)
            
            if not request.user.is_staff:
                return HttpResponse("Admin access required", status=403)
        
        return None
```

### Custom Error Pages with process_exception

**middleware.py**
```python
from django.http import HttpResponse
from django.template import loader

class CustomErrorMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_exception(self, request, exception):
        # Return custom error page
        template = loader.get_template('error.html')
        context = {'error': str(exception)}
        
        return HttpResponse(
            template.render(context, request),
            status=500
        )
```

### Dynamic Context Injection with process_template_response

**middleware.py**
```python
from django.utils import timezone

class DynamicContextMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_template_response(self, request, response):
        # Add dynamic context
        response.context_data.update({
            'now': timezone.now(),
            'request': request,
            'user': getattr(request, 'user', None),
            'debug': getattr(request, 'DEBUG', False),
        })
        
        return response
```

### Conditional Hook Execution

**middleware.py**
```python
class ConditionalHookMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        # Only process specific views
        view_name = view_func.__name__
        
        if view_name.startswith('api_'):
            print(f"API view: {view_name}")
            # Add API-specific logic
        
        return None
```

## Key Takeaways

- Middleware hooks are only available in class-based middleware
- `process_view` runs just before the view is called
- `process_exception` runs when a view raises an exception
- `process_template_response` runs after view returns TemplateResponse
- Hooks can return `None` to continue or `HttpResponse` to short-circuit
- `process_view` provides access to view function and its arguments
- `process_exception` enables global exception handling
- `process_template_response` allows modifying template context
- Only `TemplateResponse` triggers `process_template_response`, not `HttpResponse`
- Hooks provide more control than the basic `__call__` method
- Class-based middleware is preferred for complex use cases due to hooks

## Additional Context & Best Practices

### Hook Best Practices

**1. Use Hooks for Their Intended Purpose**
```python
# ✅ GOOD - Use process_view for view-specific logic
def process_view(self, request, view_func, view_args, view_kwargs):
    if view_func.__name__ == 'sensitive_view':
        if not request.user.is_authenticated:
            return redirect('login')
    return None

# ❌ BAD - Use __call__ for view-specific logic
def __call__(self, request):
    # Hard to know which view will be called
    response = self.get_response(request)
    return response
```

**2. Return None When Not Handling**
```python
# ✅ GOOD - Return None to continue normal processing
def process_exception(self, request, exception):
    if isinstance(exception, SpecificException):
        return custom_error_page()
    return None  # Let Django handle other exceptions

# ❌ BAD - Not returning anything (implicitly returns None)
def process_exception(self, request, exception):
    if isinstance(exception, SpecificException):
        return custom_error_page()
    # Missing return None
```

**3. Keep Hooks Fast**
```python
# ✅ GOOD - Fast checks
def process_view(self, request, view_func, view_args, view_kwargs):
    view_name = view_func.__name__
    if view_name in restricted_views:
        return check_permission(request)
    return None

# ❌ BAD - Slow operations
def process_view(self, request, view_func, view_args, view_kwargs):
    # Expensive database query on every request
    data = expensive_query()
    return None
```

### Common Pitfalls

**1. Forgetting to Return Response in process_template_response**
```python
# ❌ WRONG - Not returning response
def process_template_response(self, request, response):
    response.context_data['key'] = 'value'
    # Forgot return statement

# ✅ CORRECT - Always return response
def process_template_response(self, request, response):
    response.context_data['key'] = 'value'
    return response
```

**2. Using process_template_response with HttpResponse**
```python
# ❌ WRONG - Hook won't be called
def home(request):
    return HttpResponse("Hello")  # Not TemplateResponse

# ✅ CORRECT - Use TemplateResponse
from django.template.response import TemplateResponse

def home(request):
    return TemplateResponse(request, 'home.html', {'title': 'Home'})
```

**3. Modifying response in process_exception When Not Handling**
```python
# ❌ WRONG - Modifying response when returning None
def process_exception(self, request, exception):
    # This won't work if returning None
    response = self.get_response(request)
    response['X-Error'] = 'True'
    return None  # Django will ignore the modified response

# ✅ CORRECT - Return the response if modifying it
def process_exception(self, request, exception):
    response = custom_error_page()
    response['X-Error'] = 'True'
    return response
```

### Performance Considerations

**1. Minimize Hook Processing**
```python
# ✅ GOOD - Conditional execution
def process_view(self, request, view_func, view_args, view_kwargs):
    # Only process specific views
    if request.path.startswith('/admin/'):
        return check_admin_permission(request)
    return None

# ❌ BAD - Process all views
def process_view(self, request, view_func, view_args, view_kwargs):
    # Expensive check on every view
    result = expensive_check(request)
    return None
```

**2. Use Caching in Hooks**
```python
from django.core.cache import cache

class CachedHookMiddleware:
    def process_view(self, request, view_func, view_args, view_kwargs):
        view_name = view_func.__name__
        cache_key = f'view_perms_{view_name}'
        
        permissions = cache.get(cache_key)
        if not permissions:
            permissions = load_permissions(view_name)
            cache.set(cache_key, permissions, 3600)
        
        return None
```

### Advanced Tips

**1. Combining Multiple Hooks**
```python
class MultiHookMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.view_count = 0
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        self.view_count += 1
        print(f"View #{self.view_count}: {view_func.__name__}")
        return None
    
    def process_exception(self, request, exception):
        print(f"Exception: {exception}")
        return None
    
    def process_template_response(self, request, response):
        response.context_data['view_count'] = self.view_count
        return response
```

**2. View Function Inspection**
```python
class ViewInspectorMiddleware:
    def process_view(self, request, view_func, view_args, view_kwargs):
        # Get view module and name
        module = view_func.__module__
        name = view_func.__name__
        
        print(f"Module: {module}")
        print(f"Function: {name}")
        
        # Check if view has specific attributes
        if hasattr(view_func, 'requires_auth'):
            print("View requires authentication")
        
        return None
```

**3. Context Merging**
```python
class ContextMergerMiddleware:
    def process_template_response(self, request, response):
        # Merge global context with existing context
        global_context = {
            'year': 2024,
            'site_name': 'My Site',
        }
        
        # Update without overwriting existing keys
        for key, value in global_context.items():
            response.context_data.setdefault(key, value)
        
        return response
```

**4. Exception Type Handling**
```python
class TypedExceptionHandlerMiddleware:
    def process_exception(self, request, exception):
        # Handle different exception types
        if isinstance(exception, ValueError):
            return HttpResponse("Invalid input", status=400)
        elif isinstance(exception, PermissionError):
            return HttpResponse("Permission denied", status=403)
        elif isinstance(exception, DatabaseError):
            return HttpResponse("Database error", status=500)
        
        return None
```

## Practice Exercises

### Exercise 1: Create process_view Hook

Create a middleware with a `process_view` hook that prints the name of the view being called.

<details>
<summary>Solution</summary>

```python
# middleware.py
class ViewNameLoggerMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        view_name = view_func.__name__
        print(f"View called: {view_name}")
        return None
```
</details>

### Exercise 2: Block Specific Views

Create a middleware with `process_view` that blocks access to views with names containing 'delete'.

<details>
<summary>Solution</summary>

```python
# middleware.py
from django.http import HttpResponse

class DeleteBlockMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        view_name = view_func.__name__
        
        if 'delete' in view_name.lower():
            return HttpResponse("Delete operations are blocked", status=403)
        
        return None
```
</details>

### Exercise 3: Log Exceptions

Create a middleware with `process_exception` that logs all exceptions to a file.

<details>
<summary>Solution</summary>

```python
# middleware.py
import logging

logger = logging.getLogger(__name__)

class ExceptionLoggerMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_exception(self, request, exception):
        logger.error(
            f"Exception in {request.path}: {str(exception)}",
            exc_info=True
        )
        return None
```
</details>

### Exercise 4: Custom Error Response

Create a middleware with `process_exception` that returns a JSON error response for API requests (paths starting with '/api/').

<details>
<summary>Solution</summary>

```python
# middleware.py
from django.http import JsonResponse

class APIExceptionHandlerMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_exception(self, request, exception):
        if request.path.startswith('/api/'):
            return JsonResponse({
                'error': str(exception),
                'status': 'error'
            }, status=500)
        
        return None
```
</details>

### Exercise 5: Add Global Context

Create a middleware with `process_template_response` that adds a `current_year` variable to all template contexts.

<details>
<summary>Solution</summary>

```python
# middleware.py
from datetime import datetime

class GlobalContextMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_template_response(self, request, response):
        response.context_data['current_year'] = datetime.now().year
        return response
```
</details>

### Exercise 6: View Argument Inspection

Create a middleware with `process_view` that prints the arguments passed to the view.

<details>
<summary>Solution</summary>

```python
# middleware.py
class ArgumentInspectorMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        return response
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        print(f"View args: {view_args}")
        print(f"View kwargs: {view_kwargs}")
        return None
```
</details>

## Summary and Next Steps

You've now completed the Django Middleware learning series! Here's what you've learned:

### Completed Guides

1. **[001-django-middleware-fundamentals.md](001-django-middleware-fundamentals.md)**
   - What middlewares are and how they work
   - The onion layer model (request/response flow)
   - Middleware ordering and importance
   - Built-in Django middlewares
   - When to use middlewares vs decorators

2. **[002-creating-custom-middlewares.md](002-creating-custom-middlewares.md)**
   - Function-based middleware creation
   - Class-based middleware creation
   - Returning responses from middleware
   - Middleware registration in settings.py
   - Practical examples of custom middlewares

3. **[003-advanced-middleware-hooks.md](003-advanced-middleware-hooks.md)** (This Guide)
   - `process_view` hook for view-level control
   - `process_exception` hook for global exception handling
   - `process_template_response` hook for context modification
   - Hook execution order and parameters
   - When to use each hook

### Key Skills Acquired

- ✅ Understand how middlewares fit into Django's request/response cycle
- ✅ Create both function-based and class-based middleware
- ✅ Register middleware in settings.py
- ✅ Implement middleware ordering correctly
- ✅ Return responses from middleware to short-circuit
- ✅ Use `process_view` for view-specific logic
- ✅ Use `process_exception` for global error handling
- ✅ Use `process_template_response` for context injection
- ✅ Choose between function and class-based middleware
- ✅ Implement practical middleware use cases

### Further Learning

To deepen your understanding, consider exploring:

- **Django Sessions**: Built on middleware for user session management
- **Django Authentication**: Built on middleware for user authentication
- **Third-Party Middlewares**: Django REST Framework, CORS middleware, etc.
- **Middleware Testing**: How to test middleware functionality
- **Django Signals**: Alternative to middleware for certain use cases
- **Decorator Patterns**: For view-specific vs global behavior

### Official Documentation

- Django Middleware Documentation: https://docs.djangoproject.com/en/stable/topics/http/middleware/
- Django Built-in Middlewares: https://docs.djangoproject.com/en/stable/ref/middleware/
- Django Request/Response: https://docs.djangoproject.com/en/stable/ref/request-response/

Congratulations on completing this comprehensive Django Middleware learning series! You now have the skills to effectively use and create middlewares in your Django applications, from basic request/response processing to advanced hook-based control.
