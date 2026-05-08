# Updating and Deleting Data

## Introduction

Creating data is only half the story. In real applications, you'll frequently need to update existing records and delete records that are no longer needed. This guide covers how to update single and multiple rows, delete single and multiple rows, and the best practices for these operations in Django's ORM.

## Updating Single Rows

To update a single row, you follow a three-step process:
1. Retrieve the object
2. Modify its fields
3. Call `save()`

### Step-by-Step Example

```python
from post.models import Post

# Step 1: Retrieve the object
post = Post.objects.get(id=1)

# Step 2: Modify fields
post.post_title = "Updated Title"
post.post_content = "Updated content"

# Step 3: Save changes
post.save()
```

### Complete Example

```python
from post.models import Post

# Retrieve a specific post
post = Post.objects.get(id=1)

print(f"Before: {post.post_title}")

# Update the title
post.post_title = "New and Improved Title"

# Save to database
post.save()

print(f"After: {post.post_title}")
```

### Using Unique Identifiers

Always use unique identifiers (like the primary key `id`) when retrieving objects for updates:

```python
from post.models import Post

# CORRECT - uses unique ID
post = Post.objects.get(id=1)
post.post_title = "Updated"
post.save()

# RISKY - multiple posts might have the same title
post = Post.objects.get(post_title="Django")
post.post_title = "Updated"
post.save()
```

**Why?** Using non-unique fields might update the wrong record if multiple rows match.

## Updating Multiple Rows

For updating multiple rows efficiently, use the `update()` method on a QuerySet.

### Syntax

```python
Model.objects.filter(condition).update(field=new_value)
```

### Example: Update All Rows

```python
from post.models import Post

# Update all posts' titles
updated_count = Post.objects.all().update(post_title="Default Title")
print(f"Updated {updated_count} rows")
```

### Example: Update Filtered Rows

```python
from post.models import Post

# Update posts with "Test" in the title
updated_count = Post.objects.filter(
    post_title__icontains="test"
).update(post_title="Updated Title")
print(f"Updated {updated_count} rows")
```

### The update() Method Characteristics

- **Works only on QuerySets**, not on individual objects
- **Returns an integer** (number of rows updated)
- **Efficient** - performs a single SQL UPDATE statement
- **Doesn't call model methods** (like custom `save()` methods)
- **Doesn't send signals** (important for some advanced use cases)

### Example: Conditional Updates

```python
from post.models import Post

# Update posts where ID is greater than 10
updated_count = Post.objects.filter(
    id__gt=10
).update(post_title="Archived Post")
```

## update() vs save(): Key Differences

### save() Method

```python
from post.models import Post

post = Post.objects.get(id=1)
post.post_title = "New Title"
post.save()
```

**Characteristics:**
- Works on individual objects
- Calls custom `save()` methods if overridden
- Sends pre-save and post-save signals
- Less efficient for bulk updates
- More flexible (can add custom logic)

### update() Method

```python
from post.models import Post

Post.objects.filter(id=1).update(post_title="New Title")
```

**Characteristics:**
- Works only on QuerySets
- Doesn't call custom `save()` methods
- Doesn't send signals
- More efficient (single SQL statement)
- Less flexible (no custom logic)

### When to Use Which

| Scenario | Use |
|----------|-----|
| Updating single object with custom logic | `save()` |
| Updating multiple objects efficiently | `update()` |
| Need signals to fire | `save()` |
| Performance is critical for bulk updates | `update()` |
| Complex conditional logic before saving | `save()` |

## Using update() for Single Row

You can use `update()` for single rows by filtering to a unique value:

```python
from post.models import Post

# Filter to a single unique ID
updated_count = Post.objects.filter(id=1).update(post_title="Updated")
print(f"Updated {updated_count} row")
```

This is more efficient than retrieve → modify → save when you don't need custom logic.

## Deleting Single Rows

To delete a single row, retrieve the object and call `delete()`.

### Example

```python
from post.models import Post

# Retrieve the object
post = Post.objects.get(id=1)

# Delete it
post.delete()
```

### Complete Example

```python
from post.models import Post

# Check if post exists
try:
    post = Post.objects.get(id=1)
    print(f"Deleting: {post.post_title}")
    post.delete()
    print("Deleted successfully")
except Post.DoesNotExist:
    print("Post not found")
```

### Deleting with first()

```python
from post.models import Post

# Get and delete first post
post = Post.objects.all().first()
if post:
    post.delete()
```

## Deleting Multiple Rows

To delete multiple rows, filter to a QuerySet and call `delete()`.

### Example: Delete Filtered Rows

```python
from post.models import Post

# Delete all posts with "Test" in the title
deleted_count = Post.objects.filter(
    post_title__icontains="test"
).delete()
print(f"Deleted {deleted_count[0]} rows")
```

### Example: Delete All Rows

```python
from post.models import Post

# Delete all posts (be careful!)
deleted_count = Post.objects.all().delete()
print(f"Deleted {deleted_count[0]} rows")
```

### The delete() Method Characteristics

- **Works on both objects and QuerySets**
- **Returns a dictionary** with deletion counts
- **Cascades** to related objects (if configured)
- **Sends pre-delete and post-delete signals**

### Understanding the Return Value

`delete()` returns a dictionary with counts:

```python
from post.models import Post

result = Post.objects.filter(id=1).delete()
print(result)
# Output: {'post.Post': 1}
```

## Complete Workflow Examples

### Example 1: Update Single Row with Conditions

```python
from post.models import Post

# Scenario: Update post ID 5 only if it has "Old" in the title
post = Post.objects.get(id=5)
if "Old" in post.post_title:
    post.post_title = post.post_title.replace("Old", "New")
    post.save()
    print("Updated successfully")
else:
    print("No update needed")
```

### Example 2: Bulk Update with Conditions

```python
from post.models import Post

# Scenario: Mark all posts older than a year as "Archived"
from datetime import datetime, timedelta

one_year_ago = datetime.now() - timedelta(days=365)
updated_count = Post.objects.filter(
    published_date__lt=one_year_ago
).update(post_title__icontains="[Archived] ")

print(f"Archived {updated_count} posts")
```

### Example 3: Safe Deletion

```python
from post.models import Post

# Scenario: Delete posts with "Draft" in title, but only if less than 30 days old
from datetime import datetime, timedelta

thirty_days_ago = datetime.now() - timedelta(days=30)

drafts = Post.objects.filter(
    post_title__icontains="draft",
    published_date__gte=thirty_days_ago
)

if drafts:
    deleted_count = drafts.delete()
    print(f"Deleted {deleted_count[0]} draft posts")
else:
    print("No recent drafts to delete")
```

### Example 4: Conditional Update or Create

```python
from post.models import Post

# Scenario: Update if exists, create if not
post_title = "Django Tutorial"
post, created = Post.objects.get_or_create(
    post_title=post_title,
    defaults={'post_content': 'Default content'}
)

if not created:
    # Post existed, update it
    post.post_content = "Updated content"
    post.save()
    print("Updated existing post")
else:
    print("Created new post")
```

## Common Patterns

### Pattern 1: Increment a Field

```python
from post.models import Post

# Increment a view count
Post.objects.filter(id=1).update(views=models.F('views') + 1)
```

### Pattern 2: Update Multiple Fields

```python
from post.models import Post

# Update multiple fields at once
Post.objects.filter(id=1).update(
    post_title="New Title",
    post_content="New Content"
)
```

### Pattern 3: Conditional Delete

```python
from post.models import Post

# Delete posts that meet multiple criteria
Post.objects.filter(
    post_title__icontains="test",
    published_date__lt=datetime.now() - timedelta(days=7)
).delete()
```

### Pattern 4: Soft Delete (Mark as Deleted)

```python
from post.models import Post

# Instead of hard delete, mark as deleted
Post.objects.filter(id=1).update(is_deleted=True)
```

## Common Pitfalls

### Pitfall 1: Forgetting to Call save()

```python
# WRONG - changes not saved to database
post = Post.objects.get(id=1)
post.post_title = "New Title"
# Forgot post.save()

# CORRECT
post = Post.objects.get(id=1)
post.post_title = "New Title"
post.save()
```

### Pitfall 2: Using update() on get() Result

```python
# WRONG - get() returns object, not QuerySet
post = Post.objects.get(id=1)
post.update(post_title="New")  # Error!

# CORRECT - use filter() for update()
Post.objects.filter(id=1).update(post_title="New")

# Or use save()
post = Post.objects.get(id=1)
post.post_title = "New"
post.save()
```

### Pitfall 3: Using Non-Unique Identifiers for Updates

```python
# RISKY - might update wrong record
post = Post.objects.get(post_title="Django")
post.post_title = "Updated"
post.save()

# CORRECT - use unique ID
post = Post.objects.get(id=1)
post.post_title = "Updated"
post.save()
```

### Pitfall 4: Deleting Without Confirmation

```python
# DANGEROUS - deletes all posts!
Post.objects.all().delete()

# SAFER - add condition
Post.objects.filter(post_title__icontains="test").delete()

# EVEN SAFER - check count first
count = Post.objects.filter(post_title__icontains="test").count()
if count > 0:
    confirm = input(f"Delete {count} posts? (yes/no): ")
    if confirm.lower() == 'yes':
        Post.objects.filter(post_title__icontains="test").delete()
```

### Pitfall 5: Not Considering Cascading Deletes

When you delete an object, related objects might also be deleted (if your model has cascade delete configured):

```python
# This might delete related comments too!
post = Post.objects.get(id=1)
post.delete()
```

**Best Practice:** Understand your model relationships before deleting.

## Key Takeaways

- **Update single row**: Retrieve → Modify → `save()`
- **Update multiple rows**: Use `update()` on a QuerySet
- **save()** works on objects, calls custom methods and signals
- **update()** works on QuerySets, more efficient, no custom methods/signals
- **Delete single row**: Retrieve object → `delete()`
- **Delete multiple rows**: Filter QuerySet → `delete()`
- **delete()** returns a dictionary with deletion counts
- Always use unique identifiers (like `id`) for single-row operations
- `update()` only works on QuerySets, not on `get()` results
- Be careful with bulk deletes - they can't be undone
- Consider cascading deletes when deleting objects with relationships

## Additional Context & Best Practices

### Using F() Expressions for Atomic Updates

For operations that depend on current values:

```python
from django.db.models import F
from post.models import Post

# Increment view count atomically
Post.objects.filter(id=1).update(views=F('views') + 1)
```

This prevents race conditions where multiple requests try to update the same field simultaneously.

### Bulk Updates with Bulk Update

For updating many objects with different values:

```python
from post.models import Post

# Get objects
posts = list(Post.objects.all()[:10])

# Modify in Python
for post in posts:
    post.post_title = f"Updated: {post.post_title}"

# Bulk update
Post.objects.bulk_update(posts, ['post_title'])
```

### Understanding Transaction Behavior

Django wraps each `save()`, `update()`, and `delete()` in a transaction. If an error occurs, changes are rolled back:

```python
from django.db import transaction

try:
    with transaction.atomic():
        post = Post.objects.get(id=1)
        post.post_title = "New"
        post.save()
        # If this fails, everything rolls back
        Post.objects.get(id=999)  # Will raise exception
except Post.DoesNotExist:
    # Changes to post are rolled back
    pass
```

### Soft Deletes Pattern

Instead of hard deleting, mark records as deleted:

```python
class Post(models.Model):
    post_title = models.CharField(max_length=60)
    post_content = models.TextField()
    is_deleted = models.BooleanField(default=False)
    deleted_at = models.DateTimeField(null=True, blank=True)

    def soft_delete(self):
        self.is_deleted = True
        self.deleted_at = timezone.now()
        self.save()

# Usage
post = Post.objects.get(id=1)
post.soft_delete()

# Filter out deleted posts
active_posts = Post.objects.filter(is_deleted=False)
```

### Audit Trail Pattern

Track who made changes and when:

```python
class Post(models.Model):
    post_title = models.CharField(max_length=60)
    post_content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    updated_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
```

## Practice Exercises

### Exercise 1: Update Single Row

Retrieve the post with ID 1, change its title to "Updated Title", and save it.

<details>
<summary>Solution</summary>

```python
from post.models import Post

post = Post.objects.get(id=1)
post.post_title = "Updated Title"
post.save()
```

</details>

### Exercise 2: Bulk Update

Update all posts that have "Draft" in the title to change their title to include "[Draft] " prefix.

<details>
<summary>Solution</summary>

```python
from post.models import Post

from django.db.models import F

# This is more complex - need to do it differently
# Simple approach: retrieve, modify, save
drafts = Post.objects.filter(post_title__icontains="draft")
for post in drafts:
    post.post_title = f"[Draft] {post.post_title}"
post.save()

# Or if you just want to set a specific value
Post.objects.filter(post_title__icontains="draft").update(
    post_title="[Draft] Post"
)
```

</details>

### Exercise 3: Delete Multiple Rows

Delete all posts published more than 1 year ago.

<details>
<summary>Solution</summary>

```python
from post.models import Post
from datetime import datetime, timedelta

one_year_ago = datetime.now() - timedelta(days=365)
deleted_count = Post.objects.filter(
    published_date__lt=one_year_ago
).delete()
print(f"Deleted {deleted_count[0]} posts")
```

</details>

## Next Steps

Now you know how to create, retrieve, update, and delete data - the full CRUD operations! The next guide covers **rendering data in templates** - how to fetch data in your views and display it to users in your HTML templates. You'll learn about passing QuerySets to templates, looping through data, and using the `get_object_or_404` shortcut for better error handling.
