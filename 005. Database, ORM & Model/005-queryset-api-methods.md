# QuerySet API Methods

## Introduction

Django provides a rich set of QuerySet API methods that let you retrieve exactly the data you need from your database. These methods are the building blocks of database queries in Django. This guide covers the most commonly used methods: `get()`, `filter()`, `exclude()`, `order_by()`, `reverse()`, `values()`, `count()`, `contains()`, `first()`, and `last()`.

## The get() Method

The `get()` method retrieves a single object from the database based on the provided lookup parameters.

### Syntax

```python
Model.objects.get(field=value)
```

### Example

```python
from post.models import Post

# Get the post with ID 1
post = Post.objects.get(id=1)

print(post.post_title)
```

### Important Characteristics

- **Returns a single object**, not a QuerySet
- **Raises an exception** if no object matches (`DoesNotExist`)
- **Raises an exception** if multiple objects match (`MultipleObjectsReturned`)

### When to Use get()

Use `get()` when:
- You expect exactly one result
- You're querying by a unique field (like primary key)
- You need the actual object, not a QuerySet

### Exception Handling

```python
from post.models import Post
from django.core.exceptions import ObjectDoesNotExist

try:
    post = Post.objects.get(id=1)
    print(post.post_title)
except ObjectDoesNotExist:
    print("Post not found")
except Post.MultipleObjectsReturned:
    print("Multiple posts found")
```

### Common Pitfall: Non-Unique Lookups

```python
# WRONG - if multiple posts have the same title
post = Post.objects.get(post_title="Django")  # May raise MultipleObjectsReturned

# CORRECT - use unique identifier
post = Post.objects.get(id=1)
```

## The filter() Method

The `filter()` method returns a QuerySet containing objects that match the given lookup parameters.

### Syntax

```python
Model.objects.filter(field=value)
```

### Example

```python
from post.models import Post

# Get all posts with "Django" in the title
django_posts = Post.objects.filter(post_title__icontains="django")

for post in django_posts:
    print(post.post_title)
```

### Important Characteristics

- **Returns a QuerySet** (can be empty, can have multiple objects)
- **Never raises exceptions** for no results (returns empty QuerySet)
- **Chainable** - can be combined with other QuerySet methods

### When to Use filter()

Use `filter()` when:
- You expect zero, one, or multiple results
- You want to work with a collection of objects
- You need to chain additional operations

### Example: Multiple Conditions

```python
from post.models import Post

# Filter by multiple conditions
posts = Post.objects.filter(
    post_title__icontains="django",
    id__gt=5
)
```

## The exclude() Method

The `exclude()` method returns a QuerySet containing objects that **do not** match the given lookup parameters.

### Syntax

```python
Model.objects.exclude(field=value)
```

### Example

```python
from post.models import Post

# Get all posts that DON'T have "Test" in the title
non_test_posts = Post.objects.exclude(post_title__icontains="test")

for post in non_test_posts:
    print(post.post_title)
```

### Important Characteristics

- **Returns a QuerySet**
- **Opposite of filter()** - excludes matching objects
- **Chainable**

### When to Use exclude()

Use `exclude()` when:
- You want everything except certain records
- It's more readable than a complex filter condition
- You need to exclude specific criteria

### Example: filter() vs exclude()

```python
from post.models import Post

# Using filter (more complex)
posts = Post.objects.exclude(post_title__icontains="test")

# Equivalent using filter
posts = Post.objects.filter(post_title__icontains="test")
```

## The order_by() Method

The `order_by()` method changes the ordering of the QuerySet results.

### Syntax

```python
Model.objects.order_by('field_name')
```

### Example

```python
from post.models import Post

# Order by title (ascending by default)
posts = Post.objects.all().order_by('post_title')

# Order by title (descending)
posts = Post.objects.all().order_by('-post_title')

# Order by multiple fields
posts = Post.objects.all().order_by('published_date', 'post_title')
```

### Important Characteristics

- **Returns a new QuerySet**
- **Ascending order** by default
- **Prefix with `-`** for descending order
- **Chainable**

### When to Use order_by()

Use `order_by()` when:
- You need results in a specific order
- You want the most recent items first
- You're preparing data for display

### Example: Common Patterns

```python
from post.models import Post

# Most recent first
latest_posts = Post.objects.all().order_by('-published_date')

# Alphabetical by title
alphabetical = Post.objects.all().order_by('post_title')

# Multiple ordering (by date, then by title)
sorted_posts = Post.objects.all().order_by('-published_date', 'post_title')
```

## The reverse() Method

The `reverse()` method reverses the current ordering of a QuerySet.

### Syntax

```python
queryset.reverse()
```

### Example

```python
from post.models import Post

# Order ascending, then reverse
posts = Post.objects.all().order_by('published_date')
reversed_posts = posts.reverse()
```

### Important Characteristics

- **Returns a new QuerySet**
- **Reverses current ordering**
- **Requires existing ordering** (no effect on unordered QuerySets)

### When to Use reverse()

Use `reverse()` when:
- You want to flip the current order
- You're working with an already-ordered QuerySet
- It's more readable than using negative signs

### Example: order_by() vs reverse()

```python
from post.models import Post

# Using order_by with negative
posts_desc = Post.objects.all().order_by('-published_date')

# Using reverse
posts_asc = Post.objects.all().order_by('published_date')
posts_desc = posts_asc.reverse()
```

## The values() Method

The `values()` method returns a QuerySet of dictionaries instead of model objects.

### Syntax

```python
Model.objects.values('field1', 'field2', ...)
```

### Example

```python
from post.models import Post

# Get dictionaries instead of objects
posts = Post.objects.values('post_title', 'published_date')

for post in posts:
    print(post)  # {'post_title': '...', 'published_date': '...'}
    print(post['post_title'])
```

### Important Characteristics

- **Returns QuerySet of dictionaries**, not objects
- **Specify which fields** to include
- **More efficient** for large datasets (fewer data loaded)
- **Cannot access model methods** (no dot notation for methods)

### When to Use values()

Use `values()` when:
- You only need specific fields
- You're building APIs or JSON responses
- You want better performance
- You don't need model methods

### Example: values() vs all()

```python
from post.models import Post

# Using all() - returns objects
posts = Post.objects.all()
for post in posts:
    print(post.post_title)  # Works
    # post.some_method()  # Can call methods

# Using values() - returns dictionaries
posts = Post.objects.values('post_title')
for post in posts:
    print(post['post_title'])  # Works
    # post.some_method()  # ERROR - no methods
```

## The count() Method

The `count()` method returns the number of objects in the QuerySet.

### Syntax

```python
queryset.count()
```

### Example

```python
from post.models import Post

# Count all posts
total_posts = Post.objects.all().count()
print(f"Total posts: {total_posts}")

# Count filtered posts
django_posts = Post.objects.filter(post_title__icontains="django").count()
print(f"Django posts: {django_posts}")
```

### Important Characteristics

- **Returns an integer**, not a QuerySet
- **Efficient** - uses SQL COUNT, doesn't load all objects
- **Can be used on any QuerySet**

### When to Use count()

Use `count()` when:
- You only need the number of records
- You want efficient counting (no data loading)
- You're displaying counts in UI

### Example: count() vs len()

```python
from post.models import Post

# Using count() - efficient, SQL COUNT
count = Post.objects.all().count()  # Fast

# Using len() - loads all objects into memory
count = len(Post.objects.all())  # Slower for large datasets
```

## The contains() Method

The `contains()` method checks if a specific object is in the QuerySet.

### Syntax

```python
queryset.contains(object)
```

### Example

```python
from post.models import Post

# Get a specific post
post = Post.objects.get(id=1)

# Get all posts
all_posts = Post.objects.all()

# Check if post is in the QuerySet
if all_posts.contains(post):
    print("Post is in the QuerySet")
```

### Important Characteristics

- **Returns True or False**
- **Requires the actual object**, not just an ID
- **Evaluates the QuerySet** if not already evaluated

### When to Use contains()

Use `contains()` when:
- You need to check membership
- You're working with specific objects
- You need conditional logic based on presence

## The first() and last() Methods

The `first()` and `last()` methods return the first and last objects from a QuerySet.

### Syntax

```python
queryset.first()
queryset.last()
```

### Example

```python
from post.models import Post

# Get first post
first_post = Post.objects.all().first()
print(f"First post: {first_post.post_title}")

# Get last post
last_post = Post.objects.all().last()
print(f"Last post: {last_post.post_title}")
```

### Important Characteristics

- **Returns a single object** or `None` if empty
- **Returns None** (no exception) if QuerySet is empty
- **Respects current ordering** (if any)

### When to Use first() and last()

Use `first()` and `last()` when:
- You want the most recent/oldest item
- You need a single item from a collection
- You want to avoid exceptions on empty results

### Example: first() vs indexing

```python
from post.models import Post

# Using first() - safe, returns None if empty
first_post = Post.objects.all().first()
if first_post:
    print(first_post.post_title)

# Using indexing - raises error if empty
posts = Post.objects.all()
if posts:
    first_post = posts[0]
    print(first_post.post_title)
```

## Complete Example: Combining Methods

Let's combine multiple methods for a realistic scenario.

```python
from post.models import Post

# Scenario: Get the 5 most recent posts about Django
django_posts = (
    Post.objects
    .filter(post_title__icontains="django")  # Filter
    .order_by('-published_date')  # Order by date descending
    [:5]  # Slice to get first 5
)

for post in django_posts:
    print(f"{post.published_date}: {post.post_title}")

# Scenario: Count posts, excluding drafts
non_draft_count = (
    Post.objects
    .exclude(post_title__startswith="DRAFT")
    .count()
)
print(f"Non-draft posts: {non_draft_count}")

# Scenario: Get just titles and dates
post_info = (
    Post.objects
    .all()
    .values('post_title', 'published_date')
    .order_by('-published_date')
)
for info in post_info:
    print(f"{info['published_date']}: {info['post_title']}")
```

## QuerySet API Reference

Django provides many more QuerySet methods. You can find the complete reference at:
- [Django QuerySet API Reference](https://docs.djangoproject.com/en/stable/ref/models/querysets/)

### Categories of Methods

**Methods that return new QuerySets:**
- `filter()`, `exclude()`, `all()`
- `order_by()`, `reverse()`
- `distinct()`, `select_related()`, `prefetch_related()`
- And many more...

**Methods that don't return QuerySets:**
- `get()`, `create()`, `get_or_create()`
- `first()`, `last()`
- `count()`, `exists()`
- `delete()`, `update()`

## Common Patterns

### Pattern 1: Get or Create

```python
from post.models import Post

post, created = Post.objects.get_or_create(
    post_title="Unique Title",
    defaults={'post_content': 'Default content'}
)
if created:
    print("Created new post")
else:
    print("Post already exists")
```

### Pattern 2: Filter and Count

```python
from post.models import Post

count = Post.objects.filter(
    post_title__icontains="django"
).count()
```

### Pattern 3: Order and Limit

```python
from post.models import Post

recent_posts = (
    Post.objects
    .all()
    .order_by('-published_date')
    [:10]  # Last 10 posts
)
```

### Pattern 4: Get Specific Fields

```python
from post.models import Post

titles = Post.objects.values_list('post_title', flat=True)
for title in titles:
    print(title)
```

## Common Pitfalls

### Pitfall 1: Using get() on Non-Unique Fields

```python
# WRONG - may raise MultipleObjectsReturned
post = Post.objects.get(post_title="Django")

# CORRECT - use filter()
posts = Post.objects.filter(post_title="Django")
```

### Pitfall 2: Forgetting That Methods Return New QuerySets

```python
# WRONG - doesn't modify original
posts = Post.objects.all()
posts.filter(post_title__icontains="test")  # Returns new QuerySet, ignored
for post in posts:  # Still has all posts
    print(post.post_title)

# CORRECT - assign the result
posts = Post.objects.filter(post_title__icontains="test")
for post in posts:
    print(post.post_title)
```

### Pitfall 3: Using first() on Empty QuerySet Without Checking

```python
# WRONG - may cause NoneType error
post = Post.objects.filter(id=999).first()
print(post.post_title)  # Error if None

# CORRECT - check first
post = Post.objects.filter(id=999).first()
if post:
    print(post.post_title)
```

### Pitfall 4: Chaining Methods on get() Result

```python
# WRONG - get() returns object, not QuerySet
post = Post.objects.get(id=1)
post.filter(...)  # Error!

# CORRECT - use filter() for chaining
posts = Post.objects.filter(id=1)
```

## Key Takeaways

- **get()** retrieves a single object, raises exceptions if not unique or not found
- **filter()** returns a QuerySet of matching objects, never raises exceptions
- **exclude()** returns a QuerySet of non-matching objects
- **order_by()** sorts the QuerySet, use `-` for descending
- **reverse()** reverses the current ordering
- **values()** returns dictionaries instead of objects, specify fields to include
- **count()** efficiently counts objects using SQL COUNT
- **contains()** checks if an object is in the QuerySet
- **first()** and **last()** get first/last objects, return None if empty
- Methods that return QuerySets are chainable
- Methods that return objects are not chainable
- Always use unique identifiers with `get()`
- Check for None when using `first()` or `last()`

## Additional Context & Best Practices

### Performance: select_related and prefetch_related

For related objects (advanced topic), use these to avoid N+1 queries:

```python
# Optimized for foreign keys
posts = Post.objects.select_related('author')

# Optimized for many-to-many
posts = Post.objects.prefetch_related('tags')
```

### Performance: only() and defer()

Limit fields loaded from database:

```python
# Load only specific fields
posts = Post.objects.only('post_title', 'published_date')

# Exclude specific fields
posts = Post.objects.defer('post_content')
```

### Using exists() Instead of count()

If you just need to know if records exist:

```python
# More efficient than count()
if Post.objects.filter(post_title__icontains="django").exists():
    print("Django posts exist")
```

### Understanding the Default Ordering

You can set default ordering in your model:

```python
class Post(models.Model):
    post_title = models.CharField(max_length=60)
    published_date = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-published_date']  # Default ordering
```

## Practice Exercises

### Exercise 1: Filter and Order

1. Get all posts with "Django" in the title
2. Order them by published date (newest first)
3. Display the title and date of each

<details>
<summary>Solution</summary>

```python
from post.models import Post

django_posts = Post.objects.filter(
    post_title__icontains="django"
).order_by('-published_date')

for post in django_posts:
    print(f"{post.published_date}: {post.post_title}")
```

</details>

### Exercise 2: Count and Exclude

1. Count all posts
2. Count all posts excluding those with "Test" in the title
3. Display both counts

<details>
<summary>Solution</summary>

```python
from post.models import Post

total = Post.objects.all().count()
non_test = Post.objects.exclude(
    post_title__icontains="test"
).count()

print(f"Total posts: {total}")
print(f"Non-test posts: {non_test}")
```

</details>

### Exercise 3: Safe Object Retrieval

Write a function that safely retrieves a post by ID, returning None if not found:

```python
def get_post_by_id(post_id):
    # Your code here
```

<details>
<summary>Solution</summary>

```python
from post.models import Post

def get_post_by_id(post_id):
    try:
        return Post.objects.get(id=post_id)
    except Post.DoesNotExist:
        return None

# Or using filter and first
def get_post_by_id(post_id):
    return Post.objects.filter(id=post_id).first()
```

</details>

## Next Steps

Now you know the core QuerySet API methods. The next guide covers **advanced querying techniques** including field lookups, Q objects for complex queries, AND/OR operators, and limiting QuerySets. These techniques will give you even more power and flexibility in your database queries.
