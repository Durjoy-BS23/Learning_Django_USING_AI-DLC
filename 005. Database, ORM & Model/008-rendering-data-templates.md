# Rendering Data in Templates

## Introduction

So far, you've learned how to create, retrieve, update, and delete data using Django's ORM in the Python shell. But in a real web application, you need to display this data to users through HTML templates. This guide covers how to fetch data in your views and render it in templates, including handling single objects, QuerySets, and using Django's `get_object_or_404` shortcut for better error handling.

## The View-Template Connection

In Django's MVT (Model-View-Template) architecture:
- **Model**: Defines data structure and database interactions
- **View**: Contains logic, fetches data from models, prepares context
- **Template**: Displays data to users as HTML

The view is the bridge between your models and templates - it fetches data and passes it to the template for rendering.

## Fetching Data in Views

### Step 1: Import the Model

Before you can use a model in a view, you must import it:

```python
# views.py
from .models import Post
```

The `.models` refers to the `models.py` file in the same directory as `views.py`.

### Step 2: Fetch Data in the View

```python
# views.py
from django.shortcuts import render
from .models import Post

def home(request):
    # Fetch all posts
    all_posts = Post.objects.all()

    # Pass to template
    context = {
        'all_posts': all_posts
    }

    return render(request, 'index.html', context)
```

### Understanding the Context Dictionary

The context dictionary is how you pass data from your view to your template:

```python
context = {
    'variable_name_in_template': python_variable
}
```

The key becomes the variable name in the template, and the value is the actual data.

## Rendering QuerySets in Templates

### Passing a QuerySet to Template

```python
# views.py
def home(request):
    all_posts = Post.objects.all()
    context = {'all_posts': all_posts}
    return render(request, 'index.html', context)
```

### Looping Through QuerySet in Template

Use Django's `{% for %}` template tag to loop through the QuerySet:

```html
<!-- index.html -->
{% for post in all_posts %}
    <h2>{{ post.post_title }}</h2>
    <p>{{ post.post_content }}</p>
    <p>{{ post.published_date }}</p>
{% endfor %}
```

### Complete Example: Rendering All Posts

**views.py:**
```python
from django.shortcuts import render
from .models import Post

def home(request):
    all_posts = Post.objects.all()
    context = {'all_posts': all_posts}
    return render(request, 'index.html', context)
```

**index.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Blog Home</title>
</head>
<body>
    <h1>All Posts</h1>
    {% for post in all_posts %}
        <div class="post">
            <h2>{{ post.post_title }}</h2>
            <p>{{ post.post_content }}</p>
            <small>Published: {{ post.published_date }}</small>
        </div>
        <hr>
    {% empty %}
        <p>No posts found.</p>
    {% endfor %}
</body>
</html>
```

**Key Points:**
- Use `{{ variable }}` to display data
- Use `{% for %}` to loop
- Use `{% empty %}` to handle empty QuerySets
- Access object fields with dot notation: `{{ post.field_name }}`

## Accessing Model Fields in Templates

Once you have an object in your template, access its fields using dot notation:

```html
{% for post in all_posts %}
    <h2>{{ post.post_title }}</h2>
    <p>{{ post.post_content }}</p>
    <p>Date: {{ post.published_date }}</p>
    <p>ID: {{ post.id }}</p>
{% endfor %}
```

**Important:** Use the exact field names from your model definition, not what you think they should be.

## Rendering Single Objects

### Fetching a Single Object

```python
# views.py
from django.shortcuts import render
from .models import Post

def post_detail(request, post_id):
    # Fetch single post
    post = Post.objects.get(id=post_id)
    context = {'post': post}
    return render(request, 'post.html', context)
```

### Rendering Single Object in Template

```html
<!-- post.html -->
<h1>{{ post.post_title }}</h1>
<p>{{ post.post_content }}</p>
<p>Published: {{ post.published_date }}</p>
```

### URL Configuration for Single Objects

You'll typically capture the ID from the URL:

```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('post/<int:post_id>/', views.post_detail, name='post_detail'),
]
```

## Handling DoesNotExist Exceptions

When using `get()`, Django raises an exception if the object doesn't exist:

```python
# views.py
from django.shortcuts import render
from django.http import Http404
from .models import Post

def post_detail(request, post_id):
    try:
        post = Post.objects.get(id=post_id)
    except Post.DoesNotExist:
        raise Http404("Post not found")

    context = {'post': post}
    return render(request, 'post.html', context)
```

This shows Django's 404 page instead of crashing with an error.

## The get_object_or_404 Shortcut

Django provides a shortcut that combines the try/except pattern:

### Importing the Shortcut

```python
from django.shortcuts import get_object_or_404
```

### Using get_object_or_404

```python
# views.py
from django.shortcuts import render, get_object_or_404
from .models import Post

def post_detail(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    context = {'post': post}
    return render(request, 'post.html', context)
```

### How It Works

- If the object exists: Returns it (same as `get()`)
- If the object doesn't exist: Automatically raises `Http404`
- No need for try/except blocks

### Syntax

```python
get_object_or_404(Model, field=value)
```

First argument is the model class, subsequent arguments are the lookup conditions.

## Complete Example: Blog Home and Detail Pages

### views.py

```python
from django.shortcuts import render, get_object_or_404
from .models import Post

def home(request):
    """Home page - show all posts"""
    all_posts = Post.objects.all().order_by('-published_date')
    context = {'all_posts': all_posts}
    return render(request, 'index.html', context)

def post_detail(request, post_id):
    """Detail page - show single post"""
    post = get_object_or_404(Post, id=post_id)
    context = {'post': post}
    return render(request, 'post.html', context)
```

### urls.py

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
    path('post/<int:post_id>/', views.post_detail, name='post_detail'),
]
```

### index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Blog</title>
</head>
<body>
    <h1>Welcome to My Blog</h1>

    {% for post in all_posts %}
        <article class="post">
            <h2>{{ post.post_title }}</h2>
            <p>{{ post.post_content|truncatewords:30 }}</p>
            <small>Published: {{ post.published_date|date:"F d, Y" }}</small>
            <br>
            <a href="{% url 'post_detail' post.id %}">Read More</a>
        </article>
        <hr>
    {% empty %}
        <p>No posts yet. Check back soon!</p>
    {% endfor %}
</body>
</html>
```

### post.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ post.post_title }}</title>
</head>
<body>
    <article>
        <h1>{{ post.post_title }}</h1>
        <small>Published: {{ post.published_date|date:"F d, Y" }}</small>
        <hr>
        <p>{{ post.post_content }}</p>
    </article>
    <br>
    <a href="{% url 'home' %}">Back to Home</a>
</body>
</html>
```

## Template Filters

Django provides template filters to modify data before display:

### Common Filters

```html
<!-- Truncate text -->
{{ post.post_content|truncatewords:50 }}

<!-- Date formatting -->
{{ post.published_date|date:"F d, Y" }}

<!-- Default value if empty -->
{{ post.post_title|default:"Untitled" }}

<!-- Capitalize -->
{{ post.post_title|title }}

<!-- Lowercase -->
{{ post.post_title|lower }}

<!-- Line breaks to <br> -->
{{ post.post_content|linebreaks }}
```

### Chaining Filters

You can chain multiple filters:

```html
{{ post.post_title|default:"Untitled"|title }}
```

## Common Patterns

### Pattern 1: Paginated List

```python
# views.py
from django.core.paginator import Paginator

def home(request):
    all_posts = Post.objects.all()
    paginator = Paginator(all_posts, 10)  # 10 posts per page
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    context = {'page_obj': page_obj}
    return render(request, 'index.html', context)
```

### Pattern 2: Filtered List

```python
# views.py
def search(request):
    query = request.GET.get('q', '')
    if query:
        posts = Post.objects.filter(post_title__icontains=query)
    else:
        posts = Post.objects.all()
    context = {'posts': posts, 'query': query}
    return render(request, 'search.html', context)
```

### Pattern 3: Related Objects

```python
# views.py
def post_detail(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    comments = post.comment_set.all()  # Access related objects
    context = {'post': post, 'comments': comments}
    return render(request, 'post.html', context)
```

### Pattern 4: Dynamic Context

```python
# views.py
def home(request):
    context = {
        'all_posts': Post.objects.all(),
        'recent_posts': Post.objects.all()[:5],
        'total_posts': Post.objects.count(),
        'current_year': 2024
    }
    return render(request, 'index.html', context)
```

## Common Pitfalls

### Pitfall 1: Forgetting to Import the Model

```python
# WRONG - Post is not defined
def home(request):
    all_posts = Post.objects.all()  # NameError

# CORRECT - import the model
from .models import Post

def home(request):
    all_posts = Post.objects.all()
```

### Pitfall 2: Wrong Field Names in Template

```python
# Model has post_title, not title
class Post(models.Model):
    post_title = models.CharField(max_length=60)
```

```html
<!-- WRONG - field name doesn't match model -->
<h2>{{ post.title }}</h2>

<!-- CORRECT - use exact field name -->
<h2>{{ post.post_title }}</h2>
```

### Pitfall 3: Not Passing Context to Template

```python
# WRONG - no context passed
def home(request):
    all_posts = Post.objects.all()
    return render(request, 'index.html')  # all_posts not available

# CORRECT - pass context
def home(request):
    all_posts = Post.objects.all()
    context = {'all_posts': all_posts}
    return render(request, 'index.html', context)
```

### Pitfall 4: Using get() Without Error Handling

```python
# WRONG - crashes if post doesn't exist
def post_detail(request, post_id):
    post = Post.objects.get(id=post_id)
    context = {'post': post}
    return render(request, 'post.html', context)

# CORRECT - use get_object_or_404
from django.shortcuts import get_object_or_404

def post_detail(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    context = {'post': post}
    return render(request, 'post.html', context)
```

### Pitfall 5: QuerySet in Template Not Evaluated

```html
<!-- QuerySets are evaluated when accessed in templates -->
{% for post in all_posts %}
    {{ post.post_title }}
{% endfor %}

<!-- This works, but if you need the count multiple times, use count() in view -->
```

## Key Takeaways

- **Views** fetch data from models and pass it to templates via context
- **Context** is a dictionary that passes data to templates
- **Import models** in views using `from .models import ModelName`
- **QuerySets** can be looped in templates using `{% for %}`
- **Single objects** are accessed directly in templates
- **Field access** in templates uses dot notation: `{{ object.field_name }}`
- **get_object_or_404** is a shortcut for safe single object retrieval
- **get_object_or_404** automatically raises 404 if object not found
- **Template filters** modify data before display (e.g., `|date`, `|truncatewords`)
- **{% empty %}** handles empty QuerySets in loops
- Always handle DoesNotExist when using `get()`
- Use exact field names from model definition in templates

## Additional Context & Best Practices

### Optimizing Database Queries

Use `select_related` and `prefetch_related` for related objects:

```python
# Inefficient - N+1 queries
posts = Post.objects.all()
for post in posts:
    print(post.author.name)  # Separate query for each post

# Efficient - single query
posts = Post.objects.select_related('author').all()
for post in posts:
    print(post.author.name)  # No additional queries
```

### Passing Multiple QuerySets

You can pass multiple QuerySets in the context:

```python
def home(request):
    context = {
        'all_posts': Post.objects.all(),
        'featured_posts': Post.objects.filter(is_featured=True),
        'recent_posts': Post.objects.all()[:5],
    }
    return render(request, 'index.html', context)
```

### Using Template Inheritance

Create a base template and extend it:

```html
<!-- base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Blog{% endblock %}</title>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>

<!-- index.html -->
{% extends 'base.html' %}

{% block title %}Home - My Blog{% endblock %}

{% block content %}
    <h1>Welcome</h1>
    {% for post in all_posts %}
        {{ post.post_title }}
    {% endfor %}
{% endblock %}
```

### Context Processors

For data needed on every page, use context processors:

```python
# context_processors.py
from .models import Post

def recent_posts(request):
    return {
        'recent_posts': Post.objects.all()[:5]
    }
```

Then add to `TEMPLATES` in settings.py.

## Practice Exercises

### Exercise 1: Create a View and Template

1. Create a view that fetches all posts ordered by date
2. Pass them to a template
3. Display title and date for each post

<details>
<summary>Solution</summary>

**views.py:**
```python
from django.shortcuts import render
from .models import Post

def home(request):
    all_posts = Post.objects.all().order_by('-published_date')
    context = {'all_posts': all_posts}
    return render(request, 'index.html', context)
```

**index.html:**
```html
{% for post in all_posts %}
    <h2>{{ post.post_title }}</h2>
    <small>{{ post.published_date }}</small>
    <hr>
{% endfor %}
```

</details>

### Exercise 2: Create Detail View with 404 Handling

Create a detail view that:
- Takes a post_id from URL
- Uses get_object_or_404
- Passes the post to template
- Shows all fields

<details>
<summary>Solution</summary>

**views.py:**
```python
from django.shortcuts import render, get_object_or_404
from .models import Post

def post_detail(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    context = {'post': post}
    return render(request, 'post.html', context)
```

**post.html:**
```html
<h1>{{ post.post_title }}</h1>
<p>{{ post.post_content }}</p>
<p>Published: {{ post.published_date }}</p>
```

**urls.py:**
```python
path('post/<int:post_id>/', views.post_detail, name='post_detail'),
```

</details>

### Exercise 3: Fix the Code

What's wrong with this view?

```python
def home(request):
    posts = Post.objects.all()
    return render(request, 'index.html')
```

<details>
<summary>Solution</summary>

Problem: The QuerySet is not passed to the template via context.

Fixed version:
```python
def home(request):
    posts = Post.objects.all()
    context = {'posts': posts}
    return render(request, 'index.html', context)
```

</details>

## Next Steps

Now you know how to fetch data from your database and display it to users in templates. The final guide covers **aggregation functions** - how to perform calculations on your data like finding averages, maximums, minimums, and counts. This is useful for analytics, statistics, and data summarization in your applications.
