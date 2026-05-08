# Database & ORM Fundamentals

## Introduction

Before diving into Django's powerful ORM (Object Relational Mapper), it's essential to understand the foundational concepts of databases and how Django bridges the gap between Python code and database operations. This guide covers database basics, what ORM is, why we need it, and how Django models represent database tables.

## Database Basics

### What is a Database?

A database is an organized collection of data, stored and accessed electronically. In web development, databases store everything from user information to blog posts, product details, and more.

### Types of Databases

There are two main types of databases:

1. **SQL Databases (Relational Databases)**
   - Store data in tables with rows and columns
   - Use SQL (Structured Query Language) to interact with data
   - Examples: PostgreSQL, MySQL, SQLite
   - Django officially supports SQL databases

2. **NoSQL Databases**
   - Store data in various formats (documents, key-value pairs, graphs)
   - Don't use traditional table structures
   - Examples: MongoDB, Redis, Cassandra

**Django Focus:** Django officially supports SQL databases, so everything we discuss in this guide relates to SQL/relational databases.

### Database Structure

In SQL databases, data is organized as follows:

```
Database
└── Tables
    └── Columns (Fields)
    └── Rows (Records)
```

**Example - A "babies" table:**

| id | name | parent | date_of_birth |
|----|------|--------|--------------|
| 1  | Tom  | John   | 2023-01-15   |
| 2  | Emma | Sarah  | 2023-03-22   |

- **Columns/Fields:** id, name, parent, date_of_birth
- **Rows/Records:** Each individual entry (Tom's data, Emma's data)
- **Primary Key (id):** A unique identifier for each row

### The SQL Challenge

To interact with a database, you traditionally need SQL. Here are some common SQL operations:

**Retrieve all data:**
```sql
SELECT * FROM babies;
```

**Insert new data:**
```sql
INSERT INTO babies (name, parent, date_of_birth)
VALUES ('Lucy', 'Mike', '2023-06-10');
```

**Delete data:**
```sql
DELETE FROM babies WHERE name = 'Tom';
```

**The Problem:** SQL is a large, complex language that requires its own dedicated study. As a Django developer, you don't want to learn SQL just to store and retrieve data.

## What is ORM?

**ORM (Object Relational Mapper)** is a tool that allows you to interact with databases using your programming language (Python) instead of SQL.

### How ORM Works

1. You write Python code to interact with the database
2. The ORM translates your Python code into SQL behind the scenes
3. The SQL is executed on the database
4. The ORM converts the database response back into Python data types

**Example:**

```python
# Instead of writing SQL:
# SELECT * FROM babies;

# You write Python:
Babies.objects.all()
```

### Django ORM

Django comes with its own built-in ORM called **Django ORM**. It allows you to:
- Work with databases using only Python
- No need to write SQL queries manually
- Automatically handle data type conversions
- Generate and execute SQL behind the scenes

**Key Benefits:**
- **Simplicity:** Write Python, not SQL
- **Security:** Automatic SQL injection protection
- **Productivity:** Faster development
- **Database Agnostic:** Switch databases with minimal code changes

## Django Models

### What is a Model?

In Django, a **model** is a Python class that represents a database table. This is the single most important concept to understand:

> **A model represents a database table.**

That's it. All the complex explanations boil down to this simple truth.

### How to Create a Model

Models are created as Python classes inside your app's `models.py` file.

**Key Rules:**
1. Each model class must inherit from `django.db.models.Model`
2. Each attribute of the model represents a database column
3. The table name in the database will be: `appname_modelname`
4. Django automatically adds an auto-incrementing primary key field called `id`

### Example: Creating a Blog Post Model

Let's say you're building a blog and want to store posts. Here's how you'd create a Post model:

```python
# post/models.py
from django.db import models

class Post(models.Model):
    post_title = models.CharField(max_length=60)
    post_content = models.TextField()
    published_date = models.DateTimeField(auto_now=True)
```

**What this creates in the database:**

Table name: `post_post`

| Column | Type | Description |
|--------|------|-------------|
| id | Integer (auto-increment) | Primary key (added automatically) |
| post_title | Varchar(60) | Title of the post |
| post_content | Text | Content/body of the post |
| published_date | DateTime | When the post was published |

### Model Field Types

Django provides various field types to match different data needs:

#### CharField
- Used for short-to-medium length strings
- **Required argument:** `max_length`
- Example: names, titles, short descriptions

```python
title = models.CharField(max_length=100)
```

#### TextField
- Used for large amounts of text
- No max_length required
- Example: blog content, comments, descriptions

```python
content = models.TextField()
```

#### DateTimeField
- Stores date and time
- Common arguments:
  - `auto_now=True`: Automatically updates to current time on every save
  - `auto_now_add=True`: Sets to current time only on creation (never updates)

```python
created_at = models.DateTimeField(auto_now_add=True)
updated_at = models.DateTimeField(auto_now=True)
```

#### Other Common Field Types
- `IntegerField` - Whole numbers
- `FloatField` - Decimal numbers
- `BooleanField` - True/False
- `DateField` - Date only (no time)
- `EmailField` - Email addresses
- `URLField` - URLs

**Pro Tip:** You can find all available field types in the [Django Model Field Reference](https://docs.djangoproject.com/en/stable/ref/models/fields/)

### DateTime Field Arguments: auto_now vs auto_now_add

This is a common point of confusion, so let's clarify:

**auto_now=True:**
- Updates to current time EVERY time the object is saved
- Use for: "last modified" timestamps
- Cannot be manually overridden

**auto_now_add=True:**
- Sets to current time ONLY when the object is first created
- Use for: "created at" timestamps
- Cannot be manually overridden

```python
class Post(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)  # Set once on creation
    updated_at = models.DateTimeField(auto_now=True)      # Updates on every save
```

**Best Practice:** If you need both creation and modification timestamps, use two separate fields.

## Complete Example: Creating Your First Model

Let's walk through creating a complete model step by step.

### Step 1: Open models.py

Navigate to your app's `models.py` file. Django creates this file automatically when you create an app.

### Step 2: Import the models module

```python
from django.db import models
```

### Step 3: Create Your Model Class

```python
from django.db import models

class Post(models.Model):
    post_title = models.CharField(max_length=60)
    post_content = models.TextField()
    published_date = models.DateTimeField(auto_now=True)
```

**Breakdown:**
- `class Post(models.Model)`: Creates a class named Post that inherits from Django's Model
- `post_title`: A CharField with max length of 60 characters
- `post_content`: A TextField for long content
- `published_date`: A DateTimeField that auto-updates

### Step 4: What Happens Behind the Scenes

When you run migrations (covered in the next guide), Django will:
1. Create a table named `post_post` in your database
2. Add columns for each field you defined
3. Add an auto-incrementing `id` column as primary key
4. Set up the appropriate data types for each column

## Key Takeaways

- **Databases** store data in tables with rows and columns
- **SQL** is the language used to interact with databases, but it's complex
- **ORM** (Object Relational Mapper) lets you use Python instead of SQL
- **Django ORM** is Django's built-in ORM that handles SQL generation automatically
- **Models** in Django are Python classes that represent database tables
- Each model attribute represents a database column
- Django automatically adds an `id` primary key to every model
- Table names follow the pattern: `appname_modelname`
- Common field types: CharField, TextField, DateTimeField, IntegerField
- Use `auto_now_add=True` for creation timestamps, `auto_now=True` for modification timestamps

## Additional Context & Best Practices

### Choosing Field Types

**Use CharField when:**
- You have a known maximum length
- Data is relatively short (names, titles, codes)
- You want database-level validation of length

**Use TextField when:**
- You have variable-length content
- Data can be very long (blog posts, comments)
- Length is unpredictable

### Model Naming Conventions

- Use singular names for models: `Post`, not `Posts`
- Use descriptive, clear names: `BlogPost`, not `BP` or `Data`
- Follow Python naming conventions (PascalCase for classes)

### Field Naming Conventions

- Use snake_case for field names: `post_title`, not `postTitle`
- Be descriptive: `first_name`, not `fn` or `name1`
- Avoid using reserved words: don't name a field `class`, `type`, `import`

### When to Use Which DateTime Argument

| Scenario | Use |
|----------|-----|
| Track when record was created | `auto_now_add=True` |
| Track when record was last modified | `auto_now=True` |
| Allow manual date setting | Don't use either argument |
| Need both creation and modification | Use two separate fields |

### Understanding Django's Default Database

Django comes with **SQLite** as the default database. SQLite is:
- A file-based database (stored as `db.sqlite3`)
- Perfect for development and small applications
- Zero configuration required
- Not ideal for high-traffic production sites

For production, you might switch to PostgreSQL or MySQL, but your Django code remains the same - that's the power of ORM!

## Practice Exercises

### Exercise 1: Create a Student Model

Create a model for a school application that stores student information with the following fields:
- First name (max 50 characters)
- Last name (max 50 characters)
- Email address
- Date of birth
- Enrollment date (auto-set on creation)

```python
# Your solution here
```

<details>
<summary>Solution</summary>

```python
from django.db import models

class Student(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    email = models.EmailField()
    date_of_birth = models.DateField()
    enrollment_date = models.DateField(auto_now_add=True)
```

</details>

### Exercise 2: Create a Product Model

Create a model for an e-commerce site with:
- Product name (max 100 characters)
- Description
- Price (decimal number)
- Stock quantity (integer)
- Created timestamp
- Last updated timestamp

```python
# Your solution here
```

<details>
<summary>Solution</summary>

```python
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock_quantity = models.IntegerField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

</details>

### Exercise 3: Identify the Issues

What's wrong with this model?

```python
class blog(models.Model):
    title = models.CharField()
    content = models.TextField(max_length=500)
    date = models.DateTimeField(auto_now=True, auto_now_add=True)
```

<details>
<summary>Solution</summary>

Issues:
1. `blog` should be PascalCase: `Blog`
2. `CharField()` requires `max_length` argument
3. `TextField` doesn't accept `max_length`
4. Can't use both `auto_now` and `auto_now_add` together
5. `date` is too generic - use `published_date` or similar

Corrected version:
```python
class Blog(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    published_date = models.DateTimeField(auto_now=True)
```

</details>

## Next Steps

Now that you understand models, the next guide covers **migrations** - the process of translating your model definitions into actual database tables. Without migrations, your models won't create any tables in the database!
