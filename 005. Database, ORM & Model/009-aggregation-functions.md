# Aggregation Functions

## Introduction

Sometimes you need to retrieve values that are derived by summarizing or aggregating a collection of objects. Instead of getting individual records, you want calculated results like averages, maximums, minimums, or totals. This guide covers Django's aggregation functions and how to use them to perform calculations on your QuerySets.

## What is Aggregation?

**Aggregation** is the process of computing a single result from a collection of values. Common examples include:
- Average age of users
- Maximum price in a product catalog
- Total sales for the month
- Count of active users

Django provides the `aggregate()` method to perform these calculations directly in the database, which is much more efficient than loading all data into Python and calculating manually.

## The aggregate() Method

The `aggregate()` method performs calculations on a QuerySet and returns a dictionary containing the results.

### Syntax

```python
from django.db.models import AggregateFunction

queryset.aggregate(field_name=AggregateFunction('field'))
```

### Importing Aggregation Functions

```python
from django.db.models import Avg, Max, Min, Sum, Count
```

## Common Aggregation Functions

### Avg - Average

Calculates the average value of a numeric field.

```python
from django.db.models import Avg
from post.models import Post

# Calculate average of a numeric field (assuming you have one)
result = Post.objects.aggregate(average_id=Avg('id'))
print(result['average_id'])
```

### Max - Maximum

Finds the maximum value in a field.

```python
from django.db.models import Max
from post.models import Post

# Find the maximum ID
result = Post.objects.aggregate(max_id=Max('id'))
print(result['max_id'])
```

### Min - Minimum

Finds the minimum value in a field.

```python
from django.db.models import Min
from post.models import Post

# Find the minimum ID
result = Post.objects.aggregate(min_id=Min('id'))
print(result['min_id'])
```

### Sum - Total

Calculates the sum of values in a numeric field.

```python
from django.db.models import Sum
from post.models import Post

# Sum all IDs (just as an example)
result = Post.objects.aggregate(total_id=Sum('id'))
print(result['total_id'])
```

### Count - Count

Counts the number of objects (similar to QuerySet's `count()` method).

```python
from django.db.models import Count
from post.models import Post

# Count all posts
result = Post.objects.aggregate(total=Count('id'))
print(result['total'])
```

## Complete Example: Student Ages

Let's use a more realistic example with a Student model that has an age field.

```python
from django.db.models import Avg, Max, Min
from students.models import Student

# Get all students
all_students = Student.objects.all()

# Calculate average age
result = all_students.aggregate(average_age=Avg('age'))
print(f"Average age: {result['average_age']}")

# Find maximum age
result = all_students.aggregate(max_age=Max('age'))
print(f"Maximum age: {result['max_age']}")

# Find minimum age
result = all_students.aggregate(min_age=Min('age'))
print(f"Minimum age: {result['min_age']}")
```

## Understanding the Return Value

The `aggregate()` method always returns a dictionary:

```python
from django.db.models import Avg
from post.models import Post

result = Post.objects.aggregate(average_id=Avg('id'))
print(result)
# Output: {'average_id': 5.5}
```

The key name is determined by the keyword argument you pass to `aggregate()`.

## Custom Naming Aggregation Results

By default, Django names the result key as `field__aggregation_function`:

```python
from django.db.models import Avg
from post.models import Post

# Default naming
result = Post.objects.aggregate(Avg('id'))
print(result)
# Output: {'id__avg': 5.5}
```

You can provide custom names:

```python
from django.db.models import Avg
from post.models import Post

# Custom naming
result = Post.objects.aggregate(average=Avg('id'))
print(result)
# Output: {'average': 5.5}
```

## Aggregation with Filtering

You can aggregate on filtered QuerySets:

```python
from django.db.models import Avg, Max
from students.models import Student

# Average age of students 18 or older
result = Student.objects.filter(
    age__gte=18
).aggregate(average_age=Avg('age'))
print(f"Average age (18+): {result['average_age']}")

# Maximum age of students under 21
result = Student.objects.filter(
    age__lt=21
).aggregate(max_age=Max('age'))
print(f"Max age (under 21): {result['max_age']}")
```

## Multiple Aggregations in One Call

You can perform multiple aggregations in a single `aggregate()` call:

```python
from django.db.models import Avg, Max, Min
from students.models import Student

result = Student.objects.aggregate(
    average_age=Avg('age'),
    max_age=Max('age'),
    min_age=Min('age')
)

print(f"Average: {result['average_age']}")
print(f"Maximum: {result['max_age']}")
print(f"Minimum: {result['min_age']}")
```

This is more efficient than multiple separate calls.

## Direct Aggregation on objects

You can call `aggregate()` directly on `objects` without filtering first:

```python
from django.db.models import Avg
from students.models import Student

# Both are equivalent
result1 = Student.objects.aggregate(Avg('age'))
result2 = Student.objects.all().aggregate(Avg('age'))
```

## Complete Example: E-commerce Analytics

Let's imagine a Product model with a price field:

```python
from django.db.models import Avg, Max, Min, Sum, Count
from products.models import Product

# Get all products
all_products = Product.objects.all()

# Calculate multiple metrics
analytics = all_products.aggregate(
    total_products=Count('id'),
    average_price=Avg('price'),
    max_price=Max('price'),
    min_price=Min('price'),
    total_value=Sum('price')
)

print("=== Product Analytics ===")
print(f"Total products: {analytics['total_products']}")
print(f"Average price: ${analytics['average_price']:.2f}")
print(f"Maximum price: ${analytics['max_price']:.2f}")
print(f"Minimum price: ${analytics['min_price']:.2f}")
print(f"Total inventory value: ${analytics['total_value']:.2f}")
```

## Aggregation with Date Fields

You can aggregate on date fields using specific date lookups:

```python
from django.db.models import Count
from post.models import Post
from datetime import datetime

# Count posts by year
result = Post.objects.aggregate(
    count_2024=Count('id', filter=models.Q(published_date__year=2024)),
    count_2023=Count('id', filter=models.Q(published_date__year=2023))
)
```

## Common Patterns

### Pattern 1: Statistics Dashboard

```python
from django.db.models import Count, Avg, Sum
from orders.models import Order

def get_dashboard_stats():
    return Order.objects.aggregate(
        total_orders=Count('id'),
        total_revenue=Sum('total'),
        average_order_value=Avg('total'),
    )
```

### Pattern 2: Conditional Aggregation

```python
from django.db.models import Count, Q
from post.models import Post

# Count posts with and without "Django" in title
result = Post.objects.aggregate(
    django_posts=Count('id', filter=Q(post_title__icontains='django')),
    other_posts=Count('id', filter=~Q(post_title__icontains='django'))
)
```

### Pattern 3: Aggregation on Related Fields

```python
from django.db.models import Count
from blog.models import Post

# Count comments per post (assuming related model)
posts_with_counts = Post.objects.annotate(
    comment_count=Count('comments')
)
```

### Pattern 4: Range Analysis

```python
from django.db.models import Min, Max, Avg
from students.models import Student

stats = Student.objects.aggregate(
    min_age=Min('age'),
    max_age=Max('age'),
    avg_age=Avg('age')
)

age_range = stats['max_age'] - stats['min_age']
print(f"Age range: {age_range} years")
```

## Common Pitfalls

### Pitfall 1: Forgetting to Import Functions

```python
# WRONG - Avg is not defined
result = Post.objects.aggregate(Avg('id'))

# CORRECT - import the function
from django.db.models import Avg
result = Post.objects.aggregate(Avg('id'))
```

### Pitfall 2: Aggregating Non-Numeric Fields

```python
# WRONG - can't average text fields
result = Post.objects.aggregate(Avg('post_title'))

# CORRECT - aggregate numeric fields
result = Post.objects.aggregate(Count('id'))
```

### Pitfall 3: Not Handling None Results

```python
# May return None if QuerySet is empty
result = Post.objects.filter(id=999).aggregate(Avg('id'))
average = result['id__avg']  # Could be None

# CORRECT - handle None
result = Post.objects.filter(id=999).aggregate(Avg('id'))
average = result['id__avg'] or 0
```

### Pitfall 4: Confusing aggregate() with annotate()

```python
# aggregate() - returns single dictionary for entire QuerySet
result = Post.objects.aggregate(Avg('id'))
# Output: {'id__avg': 5.5}

# annotate() - adds field to each object
posts = Post.objects.annotate(avg_rating=Avg('ratings__score'))
# Each post has an avg_rating field
```

### Pitfall 5: Using Wrong Field Names

```python
# WRONG - field doesn't exist
result = Post.objects.aggregate(Avg('nonexistent_field'))

# CORRECT - use actual field names
result = Post.objects.aggregate(Avg('id'))
```

## Key Takeaways

- **Aggregation** computes single results from collections of data
- **aggregate()** method performs calculations on QuerySets
- Import functions from `django.db.models`: `Avg`, `Max`, `Min`, `Sum`, `Count`
- **aggregate()** returns a dictionary with the results
- You can **custom name** aggregation results with keyword arguments
- **Multiple aggregations** can be performed in a single call
- Aggregation works on **filtered QuerySets** for conditional calculations
- Direct aggregation on `objects` is equivalent to `objects.all()`
- Results may be **None** if QuerySet is empty
- Use aggregation for **analytics, statistics, and summaries**
- aggregate() is different from annotate() (aggregate = single result, annotate = per-object)

## Additional Context & Best Practices

### Performance Considerations

Aggregation is performed in the database, which is much more efficient than Python:

```python
# INEFFICIENT - loads all data into Python
posts = Post.objects.all()
total = sum(post.id for post in posts)

# EFFICIENT - calculation in database
result = Post.objects.aggregate(Sum('id'))
total = result['id__sum']
```

### Using Conditional Aggregation

Filter within the aggregation for complex conditions:

```python
from django.db.models import Sum, Q, F
from orders.models import Order

result = Order.objects.aggregate(
    total_revenue=Sum('total'),
    discounted_revenue=Sum('total', filter=Q(discount__gt=0)),
    premium_revenue=Sum('total', filter=Q(customer__is_premium=True))
)
```

### Aggregation with Expression Wrappers

Use `F()` expressions for calculations:

```python
from django.db.models import Sum, F
from orders.models import Order

# Sum of price * quantity (assuming these fields exist)
result = OrderItem.objects.aggregate(
    total_value=Sum(F('price') * F('quantity'))
)
```

### Understanding annotate() vs aggregate()

This is a common point of confusion:

**aggregate()**: Returns a single value for the entire QuerySet
```python
result = Post.objects.aggregate(Avg('id'))
# One average for all posts
```

**annotate()**: Adds a calculated field to each object
```python
posts = Post.objects.annotate(comment_count=Count('comments'))
# Each post has its own comment_count
```

### Database-Specific Aggregations

Some databases support additional aggregation functions. Django supports database-specific functions through expressions.

## Practice Exercises

### Exercise 1: Calculate Average

Calculate the average ID of all posts.

<details>
<summary>Solution</summary>

```python
from django.db.models import Avg
from post.models import Post

result = Post.objects.aggregate(average_id=Avg('id'))
print(f"Average ID: {result['average_id']}")
```

</details>

### Exercise 2: Multiple Aggregations

Calculate the minimum, maximum, and average ID in a single call.

<details>
<summary>Solution</summary>

```python
from django.db.models import Min, Max, Avg
from post.models import Post

result = Post.objects.aggregate(
    min_id=Min('id'),
    max_id=Max('id'),
    avg_id=Avg('id')
)

print(f"Min: {result['min_id']}")
print(f"Max: {result['max_id']}")
print(f"Average: {result['avg_id']}")
```

</details>

### Exercise 3: Filtered Aggregation

Calculate the average ID for posts with ID greater than 5.

<details>
<summary>Solution</summary>

```python
from django.db.models import Avg
from post.models import Post

result = Post.objects.filter(id__gt=5).aggregate(average_id=Avg('id'))
print(f"Average ID (ID > 5): {result['average_id']}")
```

</details>

## Conclusion

Congratulations! You've completed all 9 guides covering Django's ORM and database operations:

1. **Database & ORM Fundamentals** - Understanding databases, ORM, and models
2. **Migrations Workflow** - Synchronizing models with database schema
3. **Creating Data** - Adding records with save() and create()
4. **QuerySet Fundamentals** - Retrieving data with all() and lazy evaluation
5. **QuerySet API Methods** - get(), filter(), exclude(), and more
6. **Advanced Querying** - Field lookups, Q objects, AND/OR operators
7. **Updating & Deleting Data** - Modifying and removing records
8. **Rendering in Templates** - Displaying data to users
9. **Aggregation Functions** - Calculating summaries and statistics

You now have a solid foundation in Django's ORM and can perform all common database operations in your Django applications. These skills will serve you well as you build real-world Django projects.

For further learning, explore:
- Django's official documentation on QuerySets
- Advanced topics like select_related, prefetch_related
- Database transactions and optimization
- Custom model managers and querysets
- Django signals for automation
