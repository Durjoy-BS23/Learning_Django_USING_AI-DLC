# Creating Data with Django ORM

## Introduction

Now that you have models and migrations set up, your database tables exist but are empty. This guide covers how to create and add data to your database using Django's ORM. You'll learn two primary methods for creating records, how to use the Django shell for database interactions, and how to peek behind the scenes at the SQL Django generates.

## The Django Shell

Before creating data, let's introduce the Django shell - an interactive Python environment that gives you access to your Django project's database and models.

### Starting the Django Shell

```bash
python manage.py shell
```

The Django shell is different from a regular Python shell because:
- It sets up Django's environment automatically
- You can import your models and interact with the database
- It's perfect for testing queries and debugging

### Exiting the Shell

```bash
exit()
```

Or press `Ctrl+D`

## Method 1: Creating Objects with save()

The first method to create data involves three steps:
1. Create an instance of the model
2. Set the field values
3. Call the `save()` method

### Step-by-Step Example

**Step 1: Import the Model**

```python
from post.models import Post
```

**Step 2: Create an Instance**

```python
post1 = Post(
    post_title="First Post Title",
    post_content="First post content"
)
```

Note: We don't pass a value for `published_date` because we set `auto_now=True` in the model definition.

**Step 3: Save to Database**

```python
post1.save()
```

Now the record exists in your database!

### Complete Example

```python
# Import the model
from post.models import Post

# Create an instance
post1 = Post(
    post_title="First Post Title",
    post_content="This is the content of my first post"
)

# Save to database
post1.save()
```

### How It Works Behind the Scenes

When you call `save()`:
1. Django ORM translates your Python object into SQL
2. It generates an `INSERT` statement
3. The SQL is executed on the database
4. The database assigns an auto-incrementing ID
5. Django updates your object with the assigned ID

**Generated SQL (simplified):**
```sql
INSERT INTO post_post (post_title, post_content, published_date)
VALUES ('First Post Title', 'This is the content...', '2024-01-15 10:30:00');
```

## Method 2: Creating Objects with create()

The `create()` method is a shortcut that combines instantiation and saving into a single step.

### Syntax

```python
Model.objects.create(field1=value1, field2=value2, ...)
```

### Example

```python
from post.models import Post

post2 = Post.objects.create(
    post_title="Second Post Title",
    post_content="This is the content of my second post"
)
```

That's it! One line instead of three.

### How It Works Behind the Scenes

The `create()` method:
1. Instantiates the model object
2. Calls `save()` automatically
3. Returns the created object

**Generated SQL:**
```sql
INSERT INTO post_post (post_title, post_content, published_date)
VALUES ('Second Post Title', 'This is the content...', '2024-01-15 10:35:00');
```

## Understanding Model Managers

You've seen `Post.objects` in the examples. What is `objects`?

### What is a Model Manager?

A **model manager** is the interface through which database query operations are provided to Django models. Think of it as a bridge between your model and the database.

### The Default Manager: objects

Every Django model gets a default manager named `objects`. This manager provides access to all the database query methods like:
- `create()` - Create new records
- `all()` - Retrieve all records
- `filter()` - Filter records
- `get()` - Retrieve single record
- And many more...

### Query Structure

Most Django queries follow this pattern:

```python
Model.objects.query_method(parameters)
```

- **Model**: The model class you want to query
- **objects**: The default model manager
- **query_method**: The specific operation (create, filter, get, etc.)
- **parameters**: Any arguments the method needs

## save() vs create(): When to Use Which?

### Use save() when:
- You need to modify an object before saving
- You need to perform additional logic between creation and saving
- You're updating an existing object
- You need to handle exceptions before saving

**Example:**
```python
post = Post(post_title="Draft", post_content="...")
# Do some processing
post.post_title = "Final Title"
post.save()
```

### Use create() when:
- You have all the data upfront
- You want a simple, one-line operation
- You don't need to modify the object before saving
- You want to return the created object immediately

**Example:**
```python
post = Post.objects.create(
    post_title="Quick Post",
    post_content="Content here"
)
```

## Accessing Created Objects

After creating an object, you can access its fields using dot notation:

```python
post = Post.objects.create(
    post_title="My Post",
    post_content="Content"
)

# Access fields
print(post.post_title)  # Output: My Post
print(post.post_content)  # Output: Content
print(post.published_date)  # Output: 2024-01-15 10:30:00
print(post.id)  # Output: 1 (auto-assigned)
```

## Viewing SQL Behind the Scenes

Django provides a way to see the SQL queries it generates, which is invaluable for learning and debugging.

### Using connection.queries

```python
from django.db import connection

# Run a query
Post.objects.create(post_title="Test", post_content="Test content")

# View the SQL
print(connection.queries)
```

**Example output:**
```python
[
    {
        'sql': 'INSERT INTO "post_post" ("post_title", "post_content", "published_date") VALUES (%s, %s, %s)',
        'time': '0.002'
    }
]
```

### Important Notes

1. **Debug Mode Only**: `connection.queries` only works when `DEBUG = True` in settings
2. **Development Only**: Don't use this in production
3. **Cumulative**: It shows all queries since the last request
4. **Performance**: Each query includes execution time

### Clearing the Query Log

```python
from django.db import connection
connection.queries.clear()
```

## Complete Workflow Example

Let's put everything together in a complete example.

```python
# Start Django shell
# python manage.py shell

# Import the model
from post.models import Post
from django.db import connection

# Clear previous queries
connection.queries.clear()

# Method 1: Using save()
print("=== Method 1: save() ===")
post1 = Post(
    post_title="First Post",
    post_content="Content for first post"
)
post1.save()
print(f"Created post with ID: {post1.id}")
print(f"SQL queries: {connection.queries}")

# Clear for next example
connection.queries.clear()

# Method 2: Using create()
print("\n=== Method 2: create() ===")
post2 = Post.objects.create(
    post_title="Second Post",
    post_content="Content for second post"
)
print(f"Created post with ID: {post2.id}")
print(f"SQL queries: {connection.queries}")

# Access the created objects
print("\n=== Accessing Objects ===")
print(f"Post 1 Title: {post1.post_title}")
print(f"Post 2 Title: {post2.post_title}")
```

## Common Pitfalls

### Forgetting to Call save()

```python
# WRONG
post = Post(post_title="Title", post_content="Content")
# Forgot post.save() - nothing saved to database!

# CORRECT
post = Post(post_title="Title", post_content="Content")
post.save()
```

### Not Importing the Model

```python
# WRONG
Post.objects.create(...)  # NameError: name 'Post' is not defined

# CORRECT
from post.models import Post
Post.objects.create(...)
```

### Passing Values for auto_now Fields

```python
# Model definition
class Post(models.Model):
    published_date = models.DateTimeField(auto_now=True)

# WRONG - Django will ignore this value
post = Post(published_date=datetime.now())
post.save()

# CORRECT - Let Django handle it
post = Post(post_title="Title", post_content="Content")
post.save()
```

### Using Wrong Field Names

```python
# Model has post_title, not title
# WRONG
post = Post(title="Wrong")  # FieldError

# CORRECT
post = Post(post_title="Correct")
```

## Key Takeaways

- **Django shell** provides interactive access to your database
- **save() method**: Create instance → Set values → Call save()
- **create() method**: One-line shortcut that combines instantiation and saving
- **Model managers** (like `objects`) provide query methods
- **objects** is the default manager on every Django model
- Query pattern: `Model.objects.query_method(parameters)`
- **connection.queries** shows SQL generated by Django (debug mode only)
- Use `save()` when you need to modify before saving
- Use `create()` for simple, one-step creation
- Always call `save()` when using the instantiation method
- Don't pass values for fields with `auto_now` or `auto_now_add`

## Additional Context & Best Practices

### Bulk Creation

For creating multiple objects efficiently, use `bulk_create()`:

```python
posts = [
    Post(post_title=f"Post {i}", post_content=f"Content {i}")
    for i in range(100)
]
Post.objects.bulk_create(posts)
```

This is much faster than creating objects one by one because it generates a single SQL INSERT statement.

### Validation

Django automatically validates data before saving. For example:

```python
# CharField with max_length=60
post = Post(post_title="A" * 100)  # Will raise validation error on save()
post.save()  # Error!
```

### Custom save() Methods

You can override the `save()` method to add custom logic:

```python
class Post(models.Model):
    post_title = models.CharField(max_length=60)
    post_content = models.TextField()
    published_date = models.DateTimeField(auto_now=True)
    slug = models.SlugField(blank=True)

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.post_title)
        super().save(*args, **kwargs)
```

### Understanding the ID Field

Django automatically adds an auto-incrementing primary key called `id`. You can:
- Access it: `post.id`
- Use it to identify records uniquely
- Specify a custom primary key if needed

```python
# Custom primary key
class Post(models.Model):
    custom_id = models.AutoField(primary_key=True)
    post_title = models.CharField(max_length=60)
```

## Practice Exercises

### Exercise 1: Create Multiple Posts

Using the Django shell:
1. Import the Post model
2. Create 3 posts using the `save()` method
3. Create 2 posts using the `create()` method
4. Print the title and ID of each post

<details>
<summary>Solution</summary>

```python
from post.models import Post

# Using save()
post1 = Post(post_title="Post 1", post_content="Content 1")
post1.save()

post2 = Post(post_title="Post 2", post_content="Content 2")
post2.save()

post3 = Post(post_title="Post 3", post_content="Content 3")
post3.save()

# Using create()
post4 = Post.objects.create(post_title="Post 4", post_content="Content 4")
post5 = Post.objects.create(post_title="Post 5", post_content="Content 5")

# Print all
for post in [post1, post2, post3, post4, post5]:
    print(f"ID: {post.id}, Title: {post.post_title}")
```

</details>

### Exercise 2: View Generated SQL

1. Clear the query log
2. Create a post using `create()`
3. Print the generated SQL
4. Create a post using `save()`
5. Print the generated SQL again

<details>
<summary>Solution</summary>

```python
from post.models import Post
from django.db import connection

connection.queries.clear()

# Using create()
Post.objects.create(post_title="Test 1", post_content="Content 1")
print("After create():")
print(connection.queries)

connection.queries.clear()

# Using save()
post = Post(post_title="Test 2", post_content="Content 2")
post.save()
print("\nAfter save():")
print(connection.queries)
```

</details>

### Exercise 3: Fix the Code

What's wrong with this code and how would you fix it?

```python
post = Post(
    title="My Post",
    content="My Content"
)
```

<details>
<summary>Solution</summary>

Issues:
1. Field names are wrong (should be `post_title` and `post_content`)
2. Forgot to call `save()`

Fixed version:
```python
post = Post(
    post_title="My Post",
    post_content="My Content"
)
post.save()

# Or using create():
post = Post.objects.create(
    post_title="My Post",
    post_content="My Content"
)
```

</details>

## Next Steps

Now you know how to create data in your database. The next guide covers **QuerySet fundamentals** - how to retrieve data from your database using Django's ORM. You'll learn about the `all()` method, what QuerySets are, and the important concept of lazy evaluation.
