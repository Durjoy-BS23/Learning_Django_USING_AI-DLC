# Django Pagination in Views

## Introduction

Pagination is the process of dividing large datasets into smaller, manageable chunks displayed across multiple pages. Instead of showing hundreds or thousands of records on a single page, pagination breaks them into pages with a fixed number of items per page, improving user experience and application performance. Django provides a built-in `Paginator` class that makes implementing pagination straightforward and efficient.

## Concept Explanation

### What Is Pagination?

Pagination splits a large collection of items into discrete pages. For example:
- 100 blog posts divided into pages of 10 posts each → 10 pages
- 500 products divided into pages of 20 products each → 25 pages
- User can navigate between pages using "Previous", "Next", and numbered buttons

**Why Use Pagination?**
- **Performance**: Loading all records at once is slow and resource-intensive
- **User Experience**: Long pages are overwhelming and hard to navigate
- **Bandwidth**: Transferring fewer records reduces network usage
- **Server Load**: Reduces database query time and memory usage

### Django's Paginator Class

Django's `Paginator` class handles the logic of dividing data into pages:
- Takes a list, tuple, QuerySet, or any object with `count()` and `__len__()` methods
- Splits the data into Page objects
- Provides methods to access specific pages
- Handles edge cases like invalid page numbers

**Location**: `django.core.paginator.Paginator`

**Basic Usage:**
```python
from django.core.paginator import Paginator

# Create paginator with data and items per page
paginator = Paginator(data, items_per_page)

# Get a specific page
page = paginator.page(page_number)  # Raises exception if invalid
# or
page = paginator.get_page(page_number)  # Returns first/last page if invalid
```

### page() vs get_page() Methods

**page(page_number):**
- Raises `EmptyPage` exception if page doesn't exist
- Raises `PageNotAnInteger` exception if page_number is not an integer
- You must handle exceptions manually with try/except

**get_page(page_number):**
- Returns the last page if page_number is too high
- Returns the first page if page_number is invalid (not an integer, negative, etc.)
- Never raises exceptions
- Recommended for production use

**Recommendation**: Always use `get_page()` for user-facing pagination to avoid errors.

### Ordering QuerySets

Django warns about paginating unordered QuerySets because:
- Results may be inconsistent across requests
- Without ordering, database doesn't guarantee order
- Pagination could show different items on the same page number

**Solution**: Always order your QuerySet before paginating:
```python
# ✅ CORRECT - Ordered QuerySet
posts = Post.objects.all().order_by('-id')  # Latest first
paginator = Paginator(posts, 10)

# ❌ WRONG - Unordered QuerySet (triggers warning)
posts = Post.objects.all()
paginator = Paginator(posts, 10)
```

### Page Object Properties

The Page object returned by `page()` or `get_page()` has useful properties:
- `number`: Current page number (1-indexed)
- `paginator`: Reference to the Paginator object
- `object_list`: List of items on this page
- `has_next()`: True if there's a next page
- `has_previous()`: True if there's a previous page
- `next_page_number()`: Next page number
- `previous_page_number()`: Previous page number
- `start_index()`: 1-based index of first item on this page
- `end_index()`: 1-based index of last item on this page
- `count()`: Total number of items (same as paginator.count)

## Code Examples

### Basic Pagination Setup

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    # Get all posts
    all_posts = Post.objects.all().order_by('-id')
    
    # Create paginator - 4 posts per page
    paginator = Paginator(all_posts, 4)
    
    # Get first page
    page_obj = paginator.get_page(1)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```

**home.html:**
```html
{% for post in posts %}
    <h2>{{ post.title }}</h2>
    <p>{{ post.content }}</p>
{% endfor %}
```

### Complete Pagination View

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from .models import Post

def home(request):
    # Get all posts, ordered by ID (latest first)
    all_posts = Post.objects.all().order_by('-id')
    
    # Create paginator with 4 items per page
    paginator = Paginator(all_posts, 4)
    
    # Get page number from query parameter, default to 1
    page_number = request.GET.get('page', 1)
    
    # Get the page object using get_page (handles errors gracefully)
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```

### Using page() Method with Exception Handling

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    paginator = Paginator(all_posts, 4)
    page_number = request.GET.get('page', 1)
    
    try:
        page_obj = paginator.page(page_number)
    except PageNotAnInteger:
        # If page is not an integer, deliver first page
        page_obj = paginator.page(1)
    except EmptyPage:
        # If page is out of range, deliver last page
        page_obj = paginator.page(paginator.num_pages)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```

### Inspecting Page Object Properties

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    paginator = Paginator(all_posts, 4)
    page_obj = paginator.get_page(1)
    
    # Print page object properties for debugging
    print(f"Current page number: {page_obj.number}")
    print(f"Total pages: {paginator.num_pages}")
    print(f"Total items: {paginator.count}")
    print(f"Items on this page: {len(page_obj)}")
    print(f"Has next: {page_obj.has_next()}")
    print(f"Has previous: {page_obj.has_previous()}")
    print(f"Start index: {page_obj.start_index()}")
    print(f"End index: {page_obj.end_index()}")
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```

### Dynamic Page Number from Query Parameter

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    paginator = Paginator(all_posts, 4)
    
    # Get page number from URL query parameter (e.g., ?page=2)
    page_number = request.GET.get('page', 1)
    
    # get_page handles invalid page numbers gracefully
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```

**URL Examples:**
- `/home/` → Shows page 1
- `/home/?page=2` → Shows page 2
- `/home/?page=999` → Shows last page (if 999 doesn't exist)
- `/home/?page=abc` → Shows page 1 (invalid input)

### Paginating Filtered QuerySets

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def search(request):
    query = request.GET.get('q', '')
    
    # Filter posts based on search query
    filtered_posts = Post.objects.filter(
        title__icontains=query
    ).order_by('-id')
    
    # Paginate the filtered results
    paginator = Paginator(filtered_posts, 10)
    page_number = request.GET.get('page', 1)
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj,
        'query': query
    }
    return render(request, 'search.html', context)
```

### Multiple Paginations on Same Page

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post, Comment

def dashboard(request):
    # Paginate posts
    posts = Post.objects.all().order_by('-id')
    post_paginator = Paginator(posts, 5)
    post_page = request.GET.get('post_page', 1)
    posts_page_obj = post_paginator.get_page(post_page)
    
    # Paginate comments
    comments = Comment.objects.all().order_by('-created_at')
    comment_paginator = Paginator(comments, 10)
    comment_page = request.GET.get('comment_page', 1)
    comments_page_obj = comment_paginator.get_page(comment_page)
    
    context = {
        'posts': posts_page_obj,
        'comments': comments_page_obj
    }
    return render(request, 'dashboard.html', context)
```

## Key Takeaways

- Pagination divides large datasets into smaller, manageable pages
- Django's `Paginator` class handles pagination logic automatically
- Use `get_page()` instead of `page()` for production (handles errors gracefully)
- Always order QuerySets before paginating to avoid inconsistent results
- `get_page()` returns first page for invalid input, last page for too-high page numbers
- Page objects provide properties like `number`, `has_next()`, `has_previous()`, etc.
- Query parameters (e.g., `?page=2`) control which page to display
- Pagination improves performance, user experience, and reduces server load
- Filtered QuerySets can be paginated just like regular QuerySets
- Multiple paginations can exist on the same page with different query parameter names

## Additional Context & Best Practices

### Pagination Best Practices

**1. Always Use get_page() for User-Facing Pagination**
```python
# ✅ GOOD - Handles errors gracefully
page_obj = paginator.get_page(page_number)

# ❌ AVOID - Requires manual exception handling
try:
    page_obj = paginator.page(page_number)
except (EmptyPage, PageNotAnInteger):
    page_obj = paginator.page(1)
```

**2. Always Order QuerySets**
```python
# ✅ GOOD - Ordered QuerySet
posts = Post.objects.all().order_by('-id')
paginator = Paginator(posts, 10)

# ❌ BAD - Unordered QuerySet (triggers warning, inconsistent results)
posts = Post.objects.all()
paginator = Paginator(posts, 10)
```

**3. Choose Appropriate Page Size**
```python
# ✅ GOOD - Reasonable page size (10-50 items)
paginator = Paginator(posts, 20)

# ❌ BAD - Too small (many page turns)
paginator = Paginator(posts, 2)

# ❌ BAD - Too large (defeats pagination purpose)
paginator = Paginator(posts, 1000)
```

**4. Use Meaningful Ordering**
```python
# ✅ GOOD - Latest first for blogs
posts = Post.objects.all().order_by('-created_at')

# ✅ GOOD - Alphabetical for names
users = User.objects.all().order_by('last_name', 'first_name')

# ✅ GOOD - Price for products
products = Product.objects.all().order_by('price')
```

### Common Pitfalls

**1. Forgetting to Order QuerySet**
```python
# ❌ WRONG - Unordered QuerySet
posts = Post.objects.all()
paginator = Paginator(posts, 10)
# Warning: Pagination may yield inconsistent results

# ✅ CORRECT - Ordered QuerySet
posts = Post.objects.all().order_by('-id')
paginator = Paginator(posts, 10)
```

**2. Using page() Without Exception Handling**
```python
# ❌ WRONG - Will crash on invalid page
page_obj = paginator.page(page_number)

# ✅ CORRECT - Use get_page() or handle exceptions
page_obj = paginator.get_page(page_number)
```

**3. Paginating After Slicing**
```python
# ❌ WRONG - Paginating after slicing (inefficient)
posts = Post.objects.all()[:100]  # Loads all 100
paginator = Paginator(posts, 10)

# ✅ CORRECT - Let Paginator handle slicing
posts = Post.objects.all()
paginator = Paginator(posts, 10)
```

**4. Modifying QuerySet After Pagination**
```python
# ❌ WRONG - Modifying after pagination
page_obj = paginator.get_page(1)
filtered = [p for p in page_obj if p.is_published]

# ✅ CORRECT - Filter before pagination
posts = Post.objects.filter(is_published=True)
paginator = Paginator(posts, 10)
```

### Performance Considerations

**1. QuerySet Evaluation**
```python
# ✅ GOOD - Paginator evaluates QuerySet lazily
posts = Post.objects.all()  # Not evaluated yet
paginator = Paginator(posts, 10)  # Still not evaluated
page_obj = paginator.get_page(1)  # Evaluates only needed items

# ✅ GOOD - Only fetches items for current page
# Django uses LIMIT and OFFSET efficiently
```

**2. Count Query Performance**
```python
# Paginator executes COUNT query to determine total pages
# For very large tables, this can be slow

# ✅ GOOD - Use count() if you need total count
paginator = Paginator(posts, 10)
total_pages = paginator.num_pages  # Executes COUNT query

# ✅ GOOD - If you don't need total count, avoid accessing num_pages
page_obj = paginator.get_page(page_number)
# Just display items, don't show page numbers
```

**3. Database Indexes**
```python
# Ensure ordered fields are indexed for performance
class Post(models.Model):
    title = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['-created_at']),
        ]
```

**4. Large Datasets**
```python
# For millions of records, consider cursor-based pagination
# instead of offset-based pagination (LIMIT/OFFSET)

# ✅ GOOD - Standard pagination for moderate datasets
paginator = Paginator(posts, 10)

# ✅ BETTER - Cursor-based pagination for very large datasets
# (requires custom implementation)
```

### Security Considerations

**1. Validate Page Number**
```python
# get_page() already handles this, but if using page():
page_number = int(request.GET.get('page', 1))

# ✅ GOOD - get_page() handles validation
page_obj = paginator.get_page(page_number)

# ✅ GOOD - Manual validation if using page()
if page_number < 1:
    page_number = 1
page_obj = paginator.page(page_number)
```

**2. Prevent Information Disclosure**
```python
# Don't expose total count if it reveals sensitive information
# e.g., total number of users

# ✅ GOOD - Hide total count
context = {
    'posts': page_obj
    # Don't include paginator.num_pages
}
```

## Practice Exercises

### Exercise 1: Basic Pagination

Create a view that paginates all Post objects with 5 items per page.

<details>
<summary>Solution</summary>

```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    paginator = Paginator(all_posts, 5)
    page_number = request.GET.get('page', 1)
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```
</details>

### Exercise 2: Exception Handling

Use the `page()` method with proper exception handling instead of `get_page()`.

<details>
<summary>Solution</summary>

```python
from django.shortcuts import render
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    paginator = Paginator(all_posts, 5)
    page_number = request.GET.get('page', 1)
    
    try:
        page_obj = paginator.page(page_number)
    except PageNotAnInteger:
        page_obj = paginator.page(1)
    except EmptyPage:
        page_obj = paginator.page(paginator.num_pages)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```
</details>

### Exercise 3: Inspect Page Properties

Print all available properties of the Page object for debugging.

<details>
<summary>Solution</summary>

```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    paginator = Paginator(all_posts, 5)
    page_obj = paginator.get_page(1)
    
    print(f"Page number: {page_obj.number}")
    print(f"Has next: {page_obj.has_next()}")
    print(f"Has previous: {page_obj.has_previous()}")
    print(f"Next page number: {page_obj.next_page_number()}")
    print(f"Previous page number: {page_obj.previous_page_number()}")
    print(f"Start index: {page_obj.start_index()}")
    print(f"End index: {page_obj.end_index()}")
    print(f"Total items: {paginator.count}")
    print(f"Total pages: {paginator.num_pages}")
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```
</details>

### Exercise 4: Paginate Filtered Results

Create a search view that paginates filtered Post objects.

<details>
<summary>Solution</summary>

```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def search(request):
    query = request.GET.get('q', '')
    posts = Post.objects.filter(
        title__icontains=query
    ).order_by('-id')
    
    paginator = Paginator(posts, 10)
    page_number = request.GET.get('page', 1)
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj,
        'query': query
    }
    return render(request, 'search.html', context)
```
</details>

### Exercise 5: Custom Page Size

Allow the user to specify items per page via query parameter.

<details>
<summary>Solution</summary>

```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    
    # Get items per page from query parameter, default to 10
    items_per_page = request.GET.get('per_page', 10)
    
    # Validate items_per_page
    try:
        items_per_page = int(items_per_page)
        if items_per_page < 1 or items_per_page > 100:
            items_per_page = 10
    except ValueError:
        items_per_page = 10
    
    paginator = Paginator(all_posts, items_per_page)
    page_number = request.GET.get('page', 1)
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj,
        'items_per_page': items_per_page
    }
    return render(request, 'home.html', context)
```
</details>

## Next Steps

Now that you understand how to implement pagination in views, the next step is to learn how to create dynamic pagination controls in templates.

Continue to **[002-dynamic-pagination-in-templates.md](002-dynamic-pagination-in-templates.md)** to learn:
- Capturing query parameters in views
- Creating dynamic pagination controls in templates
- Using Page object methods for navigation
- Implementing previous/next buttons
- Creating page number navigation
- Advanced Paginator arguments (orphans, allow_empty_first_page)
- Best practices for template-based pagination
