# Dynamic Pagination in Templates

## Introduction

While views handle the logic of paginating data, templates provide the user interface for navigation between pages. Dynamic pagination allows users to navigate through pages using previous/next buttons and numbered page links. This guide covers how to create interactive pagination controls in Django templates using the Page object's methods and properties.

## Concept Explanation

### Query Parameters for Page Navigation

Query parameters are the key to dynamic pagination. They allow users to specify which page to view by appending `?page=X` to the URL:
- `/home/` → Page 1 (default)
- `/home/?page=2` → Page 2
- `/home/?page=3` → Page 3

**How It Works:**
1. User clicks a pagination button with `href="?page=2"`
2. Browser requests `/home/?page=2`
3. View captures `page` parameter: `request.GET.get('page', 1)`
4. View uses `paginator.get_page(page_number)` to get the page
5. Template renders the new page

**Best Practice:** Use a descriptive query parameter name like `page` or `p`.

### Page Object Methods for Templates

The Page object provides methods and properties essential for template navigation:

**Boolean Methods:**
- `has_previous()`: Returns `True` if there's a previous page
- `has_next()`: Returns `True` if there's a next page

**Page Number Properties:**
- `number`: Current page number (1-indexed)
- `previous_page_number()`: Previous page number
- `next_page_number()`: Next page number

**Range Properties:**
- `paginator.num_pages`: Total number of pages
- `start_index()`: 1-based index of first item on this page
- `end_index()`: 1-based index of last item on this page

### Conditional Rendering

Pagination controls should only show when relevant:
- Previous button: Only show if `has_previous()` is `True`
- Next button: Only show if `has_next()` is `True`
- Page numbers: Show current context based on page position

**Why Conditional Rendering Matters:**
- Prevents users from clicking non-existent pages
- Improves user experience by hiding irrelevant controls
- Avoids errors from accessing invalid page numbers

### Pagination Control Patterns

**Previous/Next Buttons:**
Simple navigation to adjacent pages. Always include these for basic pagination.

**Page Number Buttons:**
Direct navigation to specific pages. Useful for jumping to known pages.

**Combined Approach:**
Previous/Next buttons plus a range of page numbers around the current page. Best for large datasets.

### Advanced Paginator Arguments

**orphans:**
- Minimum number of items to avoid creating a new page
- If last page has fewer than `orphans` items, merge with previous page
- Default: 0 (no orphans)
- Example: `Paginator(data, 10, orphans=2)` → If last page has ≤2 items, add to previous page

**allow_empty_first_page:**
- Whether the first page can be empty
- Default: `True` (allows empty first page)
- If `False`, raises `EmptyPage` exception if first page is empty
- Recommended: Keep `True` to avoid errors

## Code Examples

### Basic View with Query Parameter Capture

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    paginator = Paginator(all_posts, 4)
    
    # Capture page number from query parameter
    page_number = request.GET.get('page', 1)
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```

### Previous and Next Buttons

**home.html:**
```html
<div class="pagination">
    {% if posts.has_previous %}
        <a href="?page={{ posts.previous_page_number }}">Previous</a>
    {% endif %}
    
    <span>Page {{ posts.number }} of {{ posts.paginator.num_pages }}</span>
    
    {% if posts.has_next %}
        <a href="?page={{ posts.next_page_number }}">Next</a>
    {% endif %}
</div>
```

### Complete Pagination with Page Numbers

**home.html:**
```html
<div class="pagination">
    {% if posts.has_previous %}
        <a href="?page={{ posts.previous_page_number }}" class="btn">&laquo; Previous</a>
    {% else %}
        <span class="btn disabled">&laquo; Previous</span>
    {% endif %}
    
    {% for num in posts.paginator.page_range %}
        {% if posts.number == num %}
            <span class="btn active">{{ num }}</span>
        {% else %}
            <a href="?page={{ num }}" class="btn">{{ num }}</a>
        {% endif %}
    {% endfor %}
    
    {% if posts.has_next %}
        <a href="?page={{ posts.next_page_number }}" class="btn">Next &raquo;</a>
    {% else %}
        <span class="btn disabled">Next &raquo;</span>
    {% endif %}
</div>
```

### Pagination with Adjacent Page Numbers Only

For large datasets, show only page numbers around the current page:

**home.html:**
```html
<div class="pagination">
    {% if posts.has_previous %}
        <a href="?page={{ posts.previous_page_number }}">Previous</a>
    {% endif %}
    
    {% for num in posts.paginator.page_range %}
        {% if posts.number == num %}
            <span class="active">{{ num }}</span>
        {% elif num == 1 or num == posts.paginator.num_pages or num == posts.number|add:'-1' or num == posts.number|add:'1' %}
            <a href="?page={{ num }}">{{ num }}</a>
        {% elif num == posts.number|add:'-2' or num == posts.number|add:'2' %}
            <span>...</span>
        {% endif %}
    {% endfor %}
    
    {% if posts.has_next %}
        <a href="?page={{ posts.next_page_number }}">Next</a>
    {% endif %}
</div>
```

### Bootstrap-Styled Pagination

**home.html:**
```html
<nav aria-label="Page navigation">
    <ul class="pagination">
        {% if posts.has_previous %}
            <li class="page-item">
                <a class="page-link" href="?page={{ posts.previous_page_number }}" aria-label="Previous">
                    <span aria-hidden="true">&laquo;</span>
                </a>
            </li>
        {% else %}
            <li class="page-item disabled">
                <span class="page-link">&laquo;</span>
            </li>
        {% endif %}
        
        {% for num in posts.paginator.page_range %}
            {% if posts.number == num %}
                <li class="page-item active">
                    <span class="page-link">{{ num }}</span>
                </li>
            {% else %}
                <li class="page-item">
                    <a class="page-link" href="?page={{ num }}">{{ num }}</a>
                </li>
            {% endif %}
        {% endfor %}
        
        {% if posts.has_next %}
            <li class="page-item">
                <a class="page-link" href="?page={{ posts.next_page_number }}" aria-label="Next">
                    <span aria-hidden="true">&raquo;</span>
                </a>
            </li>
        {% else %}
            <li class="page-item disabled">
                <span class="page-link">&raquo;</span>
            </li>
        {% endif %}
    </ul>
</nav>
```

### Using orphans Argument

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    
    # With orphans=2, if last page has ≤2 items, merge with previous page
    paginator = Paginator(all_posts, 4, orphans=2)
    
    page_number = request.GET.get('page', 1)
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```

**Example:**
- 10 posts, 4 per page, orphans=2
- Without orphans: Pages 1 (4 items), 2 (4 items), 3 (2 items)
- With orphans=2: Pages 1 (4 items), 2 (6 items) - last 2 merged into page 2

### Complete Example with All Features

**views.py:**
```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    
    # 4 posts per page, merge small last pages
    paginator = Paginator(all_posts, 4, orphans=2)
    
    page_number = request.GET.get('page', 1)
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```

**home.html:**
```html
<div class="container">
    <!-- Posts -->
    {% for post in posts %}
        <div class="card mb-3">
            <div class="card-body">
                <h5 class="card-title">{{ post.title }}</h5>
                <p class="card-text">{{ post.content }}</p>
            </div>
        </div>
    {% endfor %}
    
    <!-- Pagination -->
    <nav aria-label="Page navigation" class="mt-4">
        <ul class="pagination justify-content-center">
            {% if posts.has_previous %}
                <li class="page-item">
                    <a class="page-link" href="?page={{ posts.previous_page_number }}">
                        Previous
                    </a>
                </li>
            {% else %}
                <li class="page-item disabled">
                    <span class="page-link">Previous</span>
                </li>
            {% endif %}
            
            {% for num in posts.paginator.page_range %}
                {% if posts.number == num %}
                    <li class="page-item active">
                        <span class="page-link">{{ num }}</span>
                    </li>
                {% else %}
                    <li class="page-item">
                        <a class="page-link" href="?page={{ num }}">{{ num }}</a>
                    </li>
                {% endif %}
            {% endfor %}
            
            {% if posts.has_next %}
                <li class="page-item">
                    <a class="page-link" href="?page={{ posts.next_page_number }}">
                        Next
                    </a>
                </li>
            {% else %}
                <li class="page-item disabled">
                    <span class="page-link">Next</span>
                </li>
            {% endif %}
        </ul>
    </nav>
    
    <!-- Page Info -->
    <p class="text-center text-muted">
        Showing {{ posts.start_index }} to {{ posts.end_index }} 
        of {{ posts.paginator.count }} posts
    </p>
</div>
```

### Pagination with URL Parameters Preservation

When you have other query parameters, preserve them when changing pages:

**home.html:**
```html
<div class="pagination">
    {% if posts.has_previous %}
        <a href="?page={{ posts.previous_page_number }}{% if request.GET.q %}&q={{ request.GET.q }}{% endif %}">
            Previous
        </a>
    {% endif %}
    
    <span>Page {{ posts.number }}</span>
    
    {% if posts.has_next %}
        <a href="?page={{ posts.next_page_number }}{% if request.GET.q %}&q={{ request.GET.q }}{% endif %}">
            Next
        </a>
    {% endif %}
</div>
```

**Better approach using template filter:**

**templatetags/pagination_tags.py:**
```python
from django import template

register = template.Library()

@register.simple_tag
def url_replace(request, field, value):
    """Replace a query parameter while preserving others."""
    dict_ = request.GET.copy()
    dict_[field] = value
    return dict_.urlencode()
```

**home.html:**
```html
{% load pagination_tags %}

<div class="pagination">
    {% if posts.has_previous %}
        <a href="?{% url_replace request 'page' posts.previous_page_number %}">
            Previous
        </a>
    {% endif %}
    
    {% for num in posts.paginator.page_range %}
        {% if posts.number == num %}
            <span>{{ num }}</span>
        {% else %}
            <a href="?{% url_replace request 'page' num %}">{{ num }}</a>
        {% endif %}
    {% endfor %}
    
    {% if posts.has_next %}
        <a href="?{% url_replace request 'page' posts.next_page_number %}">
            Next
        </a>
    {% endif %}
</div>
```

## Key Takeaways

- Query parameters (`?page=X`) control which page to display
- Capture page number in view using `request.GET.get('page', 1)`
- Use conditional rendering (`{% if %}`) to show/hide pagination controls
- `has_previous()` and `has_next()` check if navigation buttons should show
- `previous_page_number()` and `next_page_number()` provide navigation targets
- `paginator.page_range` provides all page numbers for iteration
- `number` property shows current page number
- `orphans` argument merges small last pages with previous page
- `allow_empty_first_page` controls whether empty first pages are allowed
- Preserve existing query parameters when implementing pagination
- Use descriptive query parameter names like `page` or `p`

## Additional Context & Best Practices

### Template Best Practices

**1. Always Use Conditional Rendering**
```html
<!-- ✅ GOOD - Conditional rendering -->
{% if posts.has_previous %}
    <a href="?page={{ posts.previous_page_number }}">Previous</a>
{% endif %}

<!-- ❌ BAD - No conditional, may cause errors -->
<a href="?page={{ posts.previous_page_number }}">Previous</a>
```

**2. Show Current Page Context**
```html
<!-- ✅ GOOD - Shows user where they are -->
<span>Page {{ posts.number }} of {{ posts.paginator.num_pages }}</span>

<!-- ✅ GOOD - Shows item range -->
<p>Showing {{ posts.start_index }}-{{ posts.end_index }} of {{ posts.paginator.count }}</p>
```

**3. Use Semantic HTML**
```html
<!-- ✅ GOOD - Semantic nav element -->
<nav aria-label="Page navigation">
    <ul class="pagination">
        <!-- pagination links -->
    </ul>
</nav>

<!-- ❌ BAD - Generic div -->
<div class="pagination">
    <!-- pagination links -->
</div>
```

**4. Provide Visual Feedback**
```html
<!-- ✅ GOOD - Disabled state for unavailable buttons -->
{% if not posts.has_previous %}
    <span class="disabled">Previous</span>
{% endif %}

<!-- ✅ GOOD - Active state for current page -->
{% if posts.number == num %}
    <span class="active">{{ num }}</span>
{% endif %}
```

### Common Pitfalls

**1. Forgetting Conditional Rendering**
```html
<!-- ❌ WRONG - Will raise error on first page -->
<a href="?page={{ posts.previous_page_number }}">Previous</a>

<!-- ✅ CORRECT - Check before accessing -->
{% if posts.has_previous %}
    <a href="?page={{ posts.previous_page_number }}">Previous</a>
{% endif %}
```

**2. Not Preserving Query Parameters**
```html
<!-- ❌ WRONG - Loses other query parameters -->
<a href="?page=2">Next</a>

<!-- ✅ CORRECT - Preserves existing parameters -->
<a href="?page=2&q={{ request.GET.q }}">Next</a>
```

**3. Showing Too Many Page Numbers**
```html
<!-- ❌ WRONG - Shows all pages (bad for 1000 pages) -->
{% for num in posts.paginator.page_range %}
    <a href="?page={{ num }}">{{ num }}</a>
{% endfor %}

<!-- ✅ CORRECT - Show limited range or use ellipsis -->
<!-- Show only pages around current page -->
```

**4. Hardcoding Query Parameter Name**
```html
<!-- ❌ WRONG - Hardcoded in template -->
<a href="?page=2">Next</a>

<!-- ✅ CORRECT - Use variable if needed -->
<a href="?{{ page_param }}=2">Next</a>
```

### Performance Considerations

**1. Avoid Full Page Range for Large Datasets**
```html
<!-- ❌ BAD - Renders 1000 links for 1000 pages -->
{% for num in posts.paginator.page_range %}
    <a href="?page={{ num }}">{{ num }}</a>
{% endfor %}

<!-- ✅ GOOD - Show limited range -->
{% for num in posts.paginator.page_range %}
    {% if num <= 3 or num > posts.paginator.num_pages|add:'-3' %}
        <a href="?page={{ num }}">{{ num }}</a>
    {% endif %}
{% endfor %}
```

**2. Cache Pagination Metadata**
```python
# For very large datasets, consider caching pagination metadata
from django.core.cache import cache

def home(request):
    cache_key = f"pagination_metadata_{request.GET.q}"
    metadata = cache.get(cache_key)
    
    if not metadata:
        all_posts = Post.objects.filter(title__icontains=request.GET.q)
        paginator = Paginator(all_posts, 10)
        metadata = {
            'count': paginator.count,
            'num_pages': paginator.num_pages
        }
        cache.set(cache_key, metadata, 300)
```

**3. Lazy Evaluation Benefits**
```python
# Django's Paginator uses lazy evaluation
# Only executes queries when needed
paginator = Paginator(posts, 10)  # No query yet
page_obj = paginator.get_page(1)  # Only queries for current page
```

### UX Considerations

**1. Provide Multiple Navigation Methods**
```html
<!-- ✅ GOOD - Previous/Next + page numbers -->
<div class="pagination">
    {% if posts.has_previous %}
        <a href="?page={{ posts.previous_page_number }}">Previous</a>
    {% endif %}
    
    {% for num in posts.paginator.page_range %}
        <a href="?page={{ num }}">{{ num }}</a>
    {% endfor %}
    
    {% if posts.has_next %}
        <a href="?page={{ posts.next_page_number }}">Next</a>
    {% endif %}
</div>
```

**2. Show Position Information**
```html
<!-- ✅ GOOD - User knows where they are -->
<p>Page {{ posts.number }} of {{ posts.paginator.num_pages }}</p>
<p>Showing {{ posts.start_index }}-{{ posts.end_index }} of {{ posts.paginator.count }}</p>
```

**3. Use Clear Labels**
```html
<!-- ✅ GOOD - Descriptive labels -->
<a href="?page={{ posts.previous_page_number }}">Previous Page</a>
<a href="?page={{ posts.next_page_number }}">Next Page</a>

<!-- ❌ BAD - Ambiguous labels -->
<a href="?page={{ posts.previous_page_number }}">Prev</a>
<a href="?page={{ posts.next_page_number }}">Next</a>
```

## Practice Exercises

### Exercise 1: Basic Previous/Next Buttons

Create a simple pagination with only Previous and Next buttons.

<details>
<summary>Solution</summary>

```html
<div class="pagination">
    {% if posts.has_previous %}
        <a href="?page={{ posts.previous_page_number }}">Previous</a>
    {% endif %}
    
    <span>Page {{ posts.number }}</span>
    
    {% if posts.has_next %}
        <a href="?page={{ posts.next_page_number }}">Next</a>
    {% endif %}
</div>
```
</details>

### Exercise 2: Page Number Buttons

Add numbered page buttons to show all available pages.

<details>
<summary>Solution</summary>

```html
<div class="pagination">
    {% for num in posts.paginator.page_range %}
        {% if posts.number == num %}
            <span class="active">{{ num }}</span>
        {% else %}
            <a href="?page={{ num }}">{{ num }}</a>
        {% endif %}
    {% endfor %}
</div>
```
</details>

### Exercise 3: Complete Pagination

Combine Previous, Next, and page number buttons.

<details>
<summary>Solution</summary>

```html
<div class="pagination">
    {% if posts.has_previous %}
        <a href="?page={{ posts.previous_page_number }}">Previous</a>
    {% endif %}
    
    {% for num in posts.paginator.page_range %}
        {% if posts.number == num %}
            <span class="active">{{ num }}</span>
        {% else %}
            <a href="?page={{ num }}">{{ num }}</a>
        {% endif %}
    {% endfor %}
    
    {% if posts.has_next %}
        <a href="?page={{ posts.next_page_number }}">Next</a>
    {% endif %}
</div>
```
</details>

### Exercise 4: Using orphans Argument

Modify the view to use the orphans argument to merge small last pages.

<details>
<summary>Solution</summary>

```python
from django.shortcuts import render
from django.core.paginator import Paginator
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-id')
    
    # Merge last page if it has 2 or fewer items
    paginator = Paginator(all_posts, 4, orphans=2)
    
    page_number = request.GET.get('page', 1)
    page_obj = paginator.get_page(page_number)
    
    context = {
        'posts': page_obj
    }
    return render(request, 'home.html', context)
```
</details>

### Exercise 5: Preserve Query Parameters

Create pagination that preserves a search query parameter.

<details>
<summary>Solution</summary>

```html
<div class="pagination">
    {% if posts.has_previous %}
        <a href="?page={{ posts.previous_page_number }}&q={{ request.GET.q }}">
            Previous
        </a>
    {% endif %}
    
    {% for num in posts.paginator.page_range %}
        {% if posts.number == num %}
            <span>{{ num }}</span>
        {% else %}
            <a href="?page={{ num }}&q={{ request.GET.q }}">{{ num }}</a>
        {% endif %}
    {% endfor %}
    
    {% if posts.has_next %}
        <a href="?page={{ posts.next_page_number }}&q={{ request.GET.q }}">
            Next
        </a>
    {% endif %}
</div>
```
</details>

## Summary

You've completed both guides on Django Pagination:

1. **Pagination in Views** - Learned about the Paginator class, page() vs get_page() methods, ordering QuerySets, and Page object properties
2. **Dynamic Pagination in Templates** - Learned about query parameters, conditional rendering, pagination controls, and advanced Paginator arguments

With these skills, you can now implement complete, user-friendly pagination in your Django applications. Remember to always order your QuerySets, use `get_page()` for production, and provide clear navigation controls for the best user experience.
