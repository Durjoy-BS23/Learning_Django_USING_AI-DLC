# Advanced Querying Techniques

## Introduction

The basic QuerySet methods you've learned are powerful, but Django offers even more advanced querying capabilities. This guide covers field lookups for precise filtering, AND/OR operators for combining conditions, Q objects for complex queries, and QuerySet slicing for limiting results. These techniques will give you fine-grained control over your database queries.

## Field Lookups

Field lookups are special syntax that allow you to perform specific types of queries on fields. They use the pattern `field__lookup=value` (double underscore).

### Syntax

```python
Model.objects.filter(field__lookup=value)
```

The double underscore (`__`) is the key - it separates the field name from the lookup type.

### Common Field Lookups

#### Exact Match (exact, iexact)

```python
from post.models import Post

# Case-sensitive exact match
posts = Post.objects.filter(post_title__exact="Django")

# Case-insensitive exact match
posts = Post.objects.filter(post_title__iexact="django")
```

**Use when:** You need exact matches, with or without case sensitivity.

#### Greater Than / Less Than (gt, gte, lt, lte)

```python
from post.models import Post

# Greater than
posts = Post.objects.filter(id__gt=5)

# Greater than or equal to
posts = Post.objects.filter(id__gte=5)

# Less than
posts = Post.objects.filter(id__lt=5)

# Less than or equal to
posts = Post.objects.filter(id__lte=5)
```

**Use when:** Working with numeric or date fields where you need comparisons.

#### Contains (contains, icontains)

```python
from post.models import Post

# Case-sensitive contains
posts = Post.objects.filter(post_title__contains="Django")

# Case-insensitive contains
posts = Post.objects.filter(post_title__icontains="django")
```

**Use when:** Searching for text within a field.

#### Starts With / Ends With (startswith, istartswith, endswith, iendswith)

```python
from post.models import Post

# Case-sensitive starts with
posts = Post.objects.filter(post_title__startswith="Django")

# Case-insensitive starts with
posts = Post.objects.filter(post_title__istartswith="django")

# Case-insensitive ends with
posts = Post.objects.filter(post_title__iendswith="tutorial")
```

**Use when:** You need to match text at the beginning or end of a field.

#### In List (in)

```python
from post.models import Post

# Get posts with specific IDs
posts = Post.objects.filter(id__in=[1, 3, 5, 7])

# Get posts with specific titles
posts = Post.objects.filter(post_title__in=["Django", "Python", "Web"])
```

**Use when:** You want to match against multiple specific values.

### Case-Sensitive vs Case-Insensitive

The `i` prefix indicates case-insensitive:

| Lookup | Case-Sensitive | Case-Insensitive |
|--------|----------------|------------------|
| exact | `exact` | `iexact` |
| contains | `contains` | `icontains` |
| startswith | `startswith` | `istartswith` |
| endswith | `endswith` | `iendswith` |

**Best Practice:** Use case-insensitive lookups for user-facing searches (more forgiving), case-sensitive for exact matches where case matters.

### Complete Field Lookup Example

```python
from post.models import Post

# Complex query using multiple lookups
posts = Post.objects.filter(
    post_title__icontains="django",
    id__gte=1,
    id__lte=100
).order_by('-id')

for post in posts:
    print(f"{post.id}: {post.post_title}")
```

### All Available Field Lookups

Django provides many more field lookups. You can find the complete list at:
- [Django Field Lookups Reference](https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups)

Additional lookups include:
- `range` - Within a range
- `date` - Date part of a datetime field
- `year`, `month`, `day` - Specific date components
- `week_day` - Day of the week
- `hour`, `minute`, `second` - Time components
- `isnull` - Check for NULL values
- `regex`, `iregex` - Regular expression matching

## AND and OR Operators

### AND Operator (Implicit)

When you pass multiple keyword arguments to `filter()`, Django uses AND logic implicitly.

```python
from post.models import Post

# Both conditions must be true (AND)
posts = Post.objects.filter(
    post_title__icontains="django",
    id__gt=5
)

# Equivalent SQL: WHERE title LIKE '%django%' AND id > 5
```

**Key point:** Multiple arguments in `filter()` are always AND-ed together.

### OR Operator (Using pipe)

You can use the pipe operator (`|`) between filter calls for OR logic.

```python
from post.models import Post

# Either condition can be true (OR)
posts = Post.objects.filter(
    post_title__icontains="django"
) | Post.objects.filter(
    post_title__icontains="python"
)

# Equivalent SQL: WHERE title LIKE '%django%' OR title LIKE '%python%'
```

**Note:** This approach works but can get verbose for complex queries.

## Q Objects - The Powerful Alternative

Q objects provide a more readable and flexible way to construct complex queries with AND, OR, and NOT operators.

### Importing Q Objects

```python
from django.db.models import Q
```

### Basic Q Object Syntax

```python
from django.db.models import Q
from post.models import Post

# Simple Q object (equivalent to filter)
query = Q(post_title__icontains="django")
posts = Post.objects.filter(query)
```

### Combining Q Objects

#### AND with Q Objects

```python
from django.db.models import Q
from post.models import Post

# AND using &
posts = Post.objects.filter(
    Q(post_title__icontains="django") & Q(id__gt=5)
)
```

#### OR with Q Objects

```python
from django.db.models import Q
from post.models import Post

# OR using |
posts = Post.objects.filter(
    Q(post_title__icontains="django") | Q(post_title__icontains="python")
)
```

#### NOT with Q Objects

```python
from django.db.models import Q
from post.models import Post

# NOT using ~
posts = Post.objects.filter(
    ~Q(post_title__icontains="test")
)
```

### Complex Combinations

You can combine multiple Q objects for complex queries:

```python
from django.db.models import Q
from post.models import Post

# (title contains 'django' AND id > 5) OR title contains 'python'
posts = Post.objects.filter(
    (Q(post_title__icontains="django") & Q(id__gt=5)) |
    Q(post_title__icontains="python")
)
```

### Mixing Q Objects with Regular Arguments

You can mix Q objects with regular filter arguments:

```python
from django.db.models import Q
from post.models import Post

# Q object AND regular argument
posts = Post.objects.filter(
    Q(post_title__icontains="django"),
    id__gt=5
)
```

### When to Use Q Objects

Use Q objects when:
- You need OR logic
- You need NOT logic
- You have complex nested conditions
- You want more readable complex queries
- You're building dynamic queries programmatically

### Example: Dynamic Query Building

Q objects are perfect for building queries based on user input:

```python
from django.db.models import Q
from post.models import Post

def search_posts(search_term, min_id=None, max_id=None):
    query = Q(post_title__icontains=search_term)

    if min_id:
        query &= Q(id__gte=min_id)

    if max_id:
        query &= Q(id__lte=max_id)

    return Post.objects.filter(query)

# Usage
results = search_posts("django", min_id=5, max_id=20)
```

## Limiting QuerySets (Slicing)

QuerySets support slicing, similar to Python lists, to limit the number of results.

### Basic Slicing

```python
from post.models import Post

# Get first 5 posts
posts = Post.objects.all()[:5]

# Get posts from index 5 to 10
posts = Post.objects.all()[5:10]

# Get every other post
posts = Post.objects.all()[::2]
```

### SQL Optimization

Django translates slicing into SQL's LIMIT clause for efficiency:

```python
# Django generates: SELECT ... FROM post_post LIMIT 5
posts = Post.objects.all()[:5]
```

This means Django doesn't retrieve all rows then slice - it only retrieves what you need.

### Getting the Latest N Items

A common pattern is getting the most recent items:

```python
from post.models import Post

# Get 10 most recent posts
latest_posts = Post.objects.all().order_by('-published_date')[:10]
```

### Important Limitation: No Negative Slicing

Unlike Python lists, QuerySets don't support negative slicing:

```python
# WRONG - doesn't work with QuerySets
posts = Post.objects.all()[-1]

# CORRECT - use reverse() or last()
posts = Post.objects.all().reverse()
last_post = posts[0]

# Or
last_post = Post.objects.all().last()
```

### Slicing and Lazy Evaluation

Slicing doesn't execute the query immediately - it's still lazy:

```python
from post.models import Post
from django.db import connection

connection.queries.clear()

# Create sliced QuerySet - no query executed
posts = Post.objects.all()[:5]
print(f"Queries: {len(connection.queries)}")  # 0

# Access data - now query executes
for post in posts:
    print(post.post_title)
print(f"Queries: {len(connection.queries)}")  # 1
```

## Complete Example: Combining All Techniques

Let's combine field lookups, Q objects, and slicing in a realistic scenario.

```python
from django.db.models import Q
from post.models import Post

# Scenario: Get the 5 most recent posts that:
# - Have "Django" or "Python" in the title
# - Have ID greater than 10
# - Don't have "Test" in the title

recent_relevant_posts = (
    Post.objects.filter(
        (Q(post_title__icontains="django") | Q(post_title__icontains="python"))
        & Q(id__gt=10)
        & ~Q(post_title__icontains="test")
    )
    .order_by('-published_date')
    [:5]
)

for post in recent_relevant_posts:
    print(f"{post.id}: {post.post_title}")
```

## Common Patterns

### Pattern 1: Search with Multiple Terms

```python
from django.db.models import Q
from post.models import Post

def search_posts(terms):
    query = Q()
    for term in terms:
        query |= Q(post_title__icontains=term)
    return Post.objects.filter(query)

# Usage
results = search_posts(['django', 'python', 'web'])
```

### Pattern 2: Date Range Queries

```python
from post.models import Post
from datetime import datetime, timedelta

# Posts from the last 7 days
week_ago = datetime.now() - timedelta(days=7)
recent_posts = Post.objects.filter(published_date__gte=week_ago)

# Posts in a specific date range
start_date = datetime(2024, 1, 1)
end_date = datetime(2024, 1, 31)
posts = Post.objects.filter(
    published_date__range=(start_date, end_date)
)
```

### Pattern 3: Exclude Multiple Values

```python
from django.db.models import Q
from post.models import Post

# Exclude posts with multiple titles
excluded_titles = ['Test', 'Draft', 'Temp']
query = Q()
for title in excluded_titles:
    query |= Q(post_title__icontains=title)

posts = Post.objects.filter(~query)
```

### Pattern 4: Pagination with Slicing

```python
from post.models import Post

def get_posts_page(page, per_page=10):
    start = (page - 1) * per_page
    end = start + per_page
    return Post.objects.all().order_by('-published_date')[start:end]

# Usage
page_1 = get_posts_page(1)  # Posts 1-10
page_2 = get_posts_page(2)  # Posts 11-20
```

## Common Pitfalls

### Pitfall 1: Forgetting Field Lookup Syntax

```python
# WRONG - Python comparison operators don't work
posts = Post.objects.filter(post_title >= "A")

# CORRECT - use field lookups
posts = Post.objects.filter(post_title__gte="A")
```

### Pitfall 2: Confusing Case Sensitivity

```python
# May miss results if case differs
posts = Post.objects.filter(post_title__contains="django")

# More forgiving for user searches
posts = Post.objects.filter(post_title__icontains="django")
```

### Pitfall 3: Using Negative Slicing

```python
# WRONG - doesn't work
last_post = Post.objects.all()[-1]

# CORRECT - use last()
last_post = Post.objects.all().last()
```

### Pitfall 4: Overusing OR with Regular filter()

```python
# This doesn't work as OR - it's AND
posts = Post.objects.filter(
    post_title__icontains="django",
    post_title__icontains="python"
)

# CORRECT - use Q objects for OR
posts = Post.objects.filter(
    Q(post_title__icontains="django") | Q(post_title__icontains="python")
)
```

### Pitfall 5: Complex Queries Without Parentheses

```python
# Ambiguous - unclear operator precedence
posts = Post.objects.filter(
    Q(field1__icontains="a") & Q(field2="b") | Q(field3="c")
)

# CORRECT - use parentheses for clarity
posts = Post.objects.filter(
    (Q(field1__icontains="a") & Q(field2="b")) | Q(field3="c")
)
```

## Key Takeaways

- **Field lookups** use `field__lookup=value` syntax with double underscores
- Common lookups: `exact`, `iexact`, `gt`, `gte`, `lt`, `lte`, `contains`, `icontains`, `startswith`, `istartswith`, `in`
- **Case-insensitive lookups** have an `i` prefix (e.g., `icontains`)
- **AND** is implicit with multiple filter arguments
- **OR** requires Q objects or pipe operator
- **Q objects** provide flexible AND, OR, and NOT logic with `&`, `|`, and `~`
- **Slicing** limits QuerySet results like Python lists
- Django optimizes slicing with SQL LIMIT clause
- **No negative slicing** on QuerySets
- Use Q objects for complex or dynamic queries
- Field lookups make queries more precise and powerful

## Additional Context & Best Practices

### Performance: Indexes

Field lookups benefit from database indexes. Add indexes to frequently queried fields:

```python
class Post(models.Model):
    post_title = models.CharField(max_length=60, db_index=True)
    published_date = models.DateTimeField(auto_now=True, db_index=True)
```

### Using range() for Date/Number Ranges

The `range` lookup is cleaner than combining `gte` and `lte`:

```python
from post.models import Post
from datetime import datetime

# Instead of this
start = datetime(2024, 1, 1)
end = datetime(2024, 12, 31)
posts = Post.objects.filter(published_date__gte=start, published_date__lte=end)

# Use this
posts = Post.objects.filter(published_date__range=(start, end))
```

### Using isnull for NULL Checks

Check for NULL or NOT NULL values:

```python
# Get posts where title is NULL
posts = Post.objects.filter(post_title__isnull=True)

# Get posts where title is NOT NULL
posts = Post.objects.filter(post_title__isnull=False)
```

### Date Component Lookups

Query by specific date components:

```python
from post.models import Post

# Posts from 2024
posts = Post.objects.filter(published_date__year=2024)

# Posts from January
posts = Post.objects.filter(published_date__month=1)

# Posts from a specific day
posts = Post.objects.filter(published_date__day=15)
```

### Regular Expression Lookups

For complex pattern matching:

```python
from post.models import Post

# Case-sensitive regex
posts = Post.objects.filter(post_title__regex=r'^Django \d+')

# Case-insensitive regex
posts = Post.objects.filter(post_title__iregex=r'^django \d+')
```

## Practice Exercises

### Exercise 1: Field Lookups

Write a query that:
- Gets posts with ID greater than 5
- Title contains "Django" (case-insensitive)
- Published in 2024

<details>
<summary>Solution</summary>

```python
from post.models import Post
from datetime import datetime

posts = Post.objects.filter(
    id__gt=5,
    post_title__icontains="django",
    published_date__year=2024
)
```

</details>

### Exercise 2: Q Objects

Write a query that gets posts where:
- Title contains "Django" OR "Python"
- AND ID is greater than 10
- AND title does NOT contain "Test"

<details>
<summary>Solution</summary>

```python
from django.db.models import Q
from post.models import Post

posts = Post.objects.filter(
    (Q(post_title__icontains="django") | Q(post_title__icontains="python"))
    & Q(id__gt=10)
    & ~Q(post_title__icontains="test")
)
```

</details>

### Exercise 3: Slicing and Ordering

Get the 3 most recent posts that have "tutorial" in the title.

<details>
<summary>Solution</summary>

```python
from post.models import Post

recent_tutorials = Post.objects.filter(
    post_title__icontains="tutorial"
).order_by('-published_date')[:3]
```

</details>

## Next Steps

Now you have advanced querying techniques at your disposal. The next guide covers **updating and deleting data** - how to modify and remove records from your database using Django's ORM. You'll learn about updating single and multiple rows, deleting records, and the best practices for these operations.
