# Django Relationship Fundamentals

## Introduction

Database relationships are the foundation of relational databases, allowing you to connect different pieces of data and model real-world associations between entities. In Django, relationships are defined using special field types that create connections between models, enabling you to query related data efficiently and maintain data integrity. Understanding relationships is crucial for building any non-trivial Django application.

## Concept Explanation

### What Are Database Relationships?

Database relationships define how data in one table relates to data in another table. They mirror real-world relationships between objects:
- A user has one profile
- A blog post has many comments
- A student enrolls in many courses

These relationships are enforced by the database through foreign keys and constraints, ensuring data consistency and integrity.

### Why Use Relationships?

**Data Integrity:**
- Prevent orphaned records (comments without posts)
- Enforce referential integrity
- Maintain consistency across related data

**Query Efficiency:**
- Join related data in single queries
- Avoid manual ID lookups
- Leverage database optimization

**Code Organization:**
- Model real-world domain accurately
- Reduce code duplication
- Make code more maintainable

### Three Types of Relationships

Django supports three fundamental relationship types, each suited for different scenarios:

#### 1. One-to-One (1:1)

**Definition:** One record in Table A relates to exactly one record in Table B, and vice versa.

**Real-World Examples:**
- Person ↔ Passport (one person has one passport)
- User ↔ Profile (one user has one profile)
- Husband ↔ Wife (in monogamous relationships)

**When to Use:**
- When you want to split a large model into smaller, focused models
- When you have optional data that doesn't always exist
- When you want to extend a model without modifying it

**Field Type:** `models.OneToOneField`

#### 2. Many-to-One (M:1)

**Definition:** Many records in Table A relate to one record in Table B. This is the most common relationship type.

**Real-World Examples:**
- Comments → Post (many comments belong to one post)
- Employees → Department (many employees work in one department)
- Children → Mother (many children have one mother)

**When to Use:**
- When one entity "belongs to" another
- When you need to group related items under a parent
- When you want to easily access all children from a parent

**Field Type:** `models.ForeignKey` (note: Django calls it "ForeignKey" even though it's many-to-one)

#### 3. Many-to-Many (M:N)

**Definition:** Many records in Table A relate to many records in Table B.

**Real-World Examples:**
- Posts ↔ Tags (posts have multiple tags, tags appear in multiple posts)
- Students ↔ Courses (students take multiple courses, courses have multiple students)
- Authors ↔ Books (authors write multiple books, books have multiple authors)

**When to Use:**
- When both sides of the relationship can have multiple connections
- When you need to associate items in many-to-many fashion
- When you need a flexible, bidirectional relationship

**Field Type:** `models.ManyToManyField`

**Database Implementation:** Django creates an intermediate "through" table to manage M:N relationships automatically.

### Choosing the Right Relationship Type

**Decision Framework:**

| Question | Answer | Relationship Type |
|----------|--------|-------------------|
| Does A have only one B, and B has only one A? | Yes | One-to-One |
| Does A have many B's, but each B belongs to only one A? | Yes | Many-to-One |
| Do both A and B have multiple connections to each other? | Yes | Many-to-Many |

**Examples:**

**User and Profile:**
- One user has one profile → One-to-One

**Post and Comments:**
- One post has many comments → Many-to-One
- Each comment belongs to one post

**Posts and Tags:**
- One post has many tags → Many-to-Many
- One tag appears in many posts

### Foreign Key Direction

A common point of confusion is determining which model should hold the relationship field.

**General Rule:** Place the ForeignKey on the "many" side of the relationship.

**Examples:**
- Comments → Post: ForeignKey on Comment (many comments, one post)
- Employees → Department: ForeignKey on Employee (many employees, one department)
- Children → Mother: ForeignKey on Child (many children, one mother)

**For One-to-One:** Choose based on which model makes sense to "own" the relationship or which is optional.

## Code Examples

### Basic Model Structure

**models.py:**
```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    
    def __str__(self):
        return self.name

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    # One author can write many books (many-to-one)
    
    def __str__(self):
        return self.title

class Profile(models.Model):
    user = models.OneToOneField('User', on_delete=models.CASCADE)
    bio = models.TextField()
    # One user has one profile (one-to-one)
    
    def __str__(self):
        return f"{self.user}'s profile"

class Tag(models.Model):
    name = models.CharField(max_length=50)
    
    def __str__(self):
        return self.name

class Article(models.Model):
    title = models.CharField(max_length=200)
    tags = models.ManyToManyField(Tag)
    # Articles have multiple tags, tags in multiple articles (many-to-many)
    
    def __str__(self):
        return self.title
```

### One-to-One Example

**models.py:**
```python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=100)
    
    def __str__(self):
        return self.name

class Passport(models.Model):
    person = models.OneToOneField(Person, on_delete=models.CASCADE)
    passport_number = models.CharField(max_length=20)
    expiry_date = models.DateField()
    
    def __str__(self):
        return f"Passport {self.passport_number}"
```

**Usage:**
```python
# Create a person
person = Person.objects.create(name="John Doe")

# Create a passport for the person
passport = Passport.objects.create(
    person=person,
    passport_number="AB1234567",
    expiry_date="2030-12-31"
)

# Access passport from person
print(person.passport)  # Returns the Passport object

# Access person from passport
print(passport.person)  # Returns the Person object
```

### Many-to-One Example

**models.py:**
```python
from django.db import models

class Department(models.Model):
    name = models.CharField(max_length=100)
    
    def __str__(self):
        return self.name

class Employee(models.Model):
    name = models.CharField(max_length=100)
    department = models.ForeignKey(Department, on_delete=models.CASCADE)
    
    def __str__(self):
        return self.name
```

**Usage:**
```python
# Create a department
dept = Department.objects.create(name="Engineering")

# Create employees in the department
emp1 = Employee.objects.create(name="Alice", department=dept)
emp2 = Employee.objects.create(name="Bob", department=dept)

# Access department from employee
print(emp1.department.name)  # "Engineering"

# Access all employees from department
print(dept.employee_set.all())  # QuerySet of all employees
```

### Many-to-Many Example

**models.py:**
```python
from django.db import models

class Student(models.Model):
    name = models.CharField(max_length=100)
    
    def __str__(self):
        return self.name

class Course(models.Model):
    name = models.CharField(max_length=100)
    students = models.ManyToManyField(Student)
    
    def __str__(self):
        return self.name
```

**Usage:**
```python
# Create students
student1 = Student.objects.create(name="Alice")
student2 = Student.objects.create(name="Bob")

# Create a course
course = Course.objects.create(name="Python Programming")

# Add students to course
course.students.add(student1, student2)

# Access courses from student
print(student1.course_set.all())  # QuerySet of all courses

# Access students from course
print(course.students.all())  # QuerySet of all students
```

## Key Takeaways

- Database relationships connect data between tables and model real-world associations
- Three relationship types: One-to-One, Many-to-One, and Many-to-Many
- One-to-One: Use `OneToOneField` for unique 1:1 connections (user profiles)
- Many-to-One: Use `ForeignKey` when many items belong to one parent (comments on posts)
- Many-to-Many: Use `ManyToManyField` when both sides have multiple connections (posts and tags)
- Place ForeignKey on the "many" side of the relationship
- Relationships maintain data integrity and enable efficient queries
- Choose relationship type based on how entities relate in the real world
- Django automatically creates intermediate tables for many-to-many relationships

## Additional Context & Best Practices

### Relationship Field Naming Conventions

**Use Descriptive Names:**
```python
# ✅ GOOD - Clear and descriptive
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    # "post" clearly indicates what this field represents

# ❌ BAD - Ambiguous
class Comment(models.Model):
    related_post = models.ForeignKey(Post, on_delete=models.CASCADE)
```

**Use Singular for ForeignKey:**
```python
# ✅ GOOD - Singular form
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)

# ❌ BAD - Plural form (confusing)
class Comment(models.Model):
    posts = models.ForeignKey(Post, on_delete=models.CASCADE)
```

### Performance Considerations

**N+1 Query Problem:**
```python
# ❌ BAD - N+1 queries
posts = Post.objects.all()
for post in posts:
    print(post.author.name)  # Separate query for each post

# ✅ GOOD - Single query with select_related
posts = Post.objects.select_related('author').all()
for post in posts:
    print(post.author.name)  # No additional queries
```

**Use select_related for ForeignKey:**
- Fetches related objects in a single query
- Use for one-to-one and many-to-one relationships
- Follows foreign keys automatically

**Use prefetch_related for ManyToMany:**
- Fetches related objects in separate queries but efficiently
- Use for many-to-many and reverse foreign key relationships
- Reduces query count significantly

### Common Pitfalls

**1. Wrong Relationship Type:**
```python
# ❌ WRONG - Using OneToOne when you need ForeignKey
class Comment(models.Model):
    post = models.OneToOneField(Post, on_delete=models.CASCADE)
    # Only allows one comment per post!

# ✅ CORRECT - Using ForeignKey for many-to-one
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    # Allows multiple comments per post
```

**2. Forgetting on_delete:**
```python
# ❌ WRONG - Missing on_delete argument
class Comment(models.Model):
    post = models.ForeignKey(Post)  # Error in Django 2.0+

# ✅ CORRECT - Always specify on_delete
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
```

**3. Circular Imports:**
```python
# ❌ WRONG - Can cause circular import
from .models import ModelB

class ModelA(models.Model):
    related = models.ForeignKey(ModelB, on_delete=models.CASCADE)

# ✅ CORRECT - Use string reference
class ModelA(models.Model):
    related = models.ForeignKey('ModelB', on_delete=models.CASCADE)
```

**4. Not Understanding Related Name:**
```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

# Access books from author
author.book_set.all()  # Default: modelname_set
```

### Database Design Best Practices

**Normalize Data:**
- Don't repeat data across tables
- Use relationships to reference shared data
- Update in one place, reflect everywhere

**Choose Appropriate Constraints:**
- Use `on_delete=CASCADE` when child can't exist without parent
- Use `on_delete=PROTECT` to prevent accidental deletion
- Use `on_delete=SET_NULL` when relationship is optional

**Index Foreign Keys:**
- Django automatically indexes ForeignKey fields
- This improves query performance for joins
- Don't add manual indexes unless needed

### Advanced Concepts

**Self-Referential Relationships:**
```python
class Employee(models.Model):
    name = models.CharField(max_length=100)
    manager = models.ForeignKey('self', on_delete=models.CASCADE, null=True)
    # An employee can have a manager who is also an employee
```

**Abstract Base Classes:**
```python
class TimeStampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True

class Post(TimeStampedModel):
    title = models.CharField(max_length=200)
    # Inherits created_at and updated_at
```

**Proxy Models:**
```python
class Person(models.Model):
    name = models.CharField(max_length=100)

class Employee(Person):
    class Meta:
        proxy = True  # No new table, just different behavior
```

## Practice Exercises

### Exercise 1: Identify Relationship Types

For each scenario, identify the appropriate relationship type:

1. A customer places many orders
2. A user has one address
3. A student attends many classes, and each class has many students
4. A country has one capital city
5. A product belongs to one category

<details>
<summary>Solution</summary>

1. **Many-to-One** - Customer (one) has many orders (many)
2. **One-to-One** - User has one address
3. **Many-to-Many** - Students and classes
4. **One-to-One** - Country has one capital
5. **Many-to-One** - Product belongs to one category
</details>

### Exercise 2: Create Models

Create models for a blog with:
- Users who can write posts
- Posts that can have many comments
- Posts that can have many tags

<details>
<summary>Solution</summary>

```python
from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title

class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return f"Comment on {self.post.title}"

class Tag(models.Model):
    name = models.CharField(max_length=50)
    
    def __str__(self):
        return self.name

# Add ManyToManyField to Post
Post.tags = models.ManyToManyField(Tag)
```
</details>

### Exercise 3: Query Relationships

Given the models from Exercise 2, write queries to:
1. Get all posts by a specific user
2. Get all comments on a specific post
3. Get all posts with a specific tag

<details>
<summary>Solution</summary>

```python
from django.contrib.auth.models import User

# 1. Get all posts by a specific user
user = User.objects.get(username='john')
posts_by_user = Post.objects.filter(author=user)

# 2. Get all comments on a specific post
post = Post.objects.get(id=1)
comments_on_post = post.comment_set.all()
# or
comments_on_post = Comment.objects.filter(post=post)

# 3. Get all posts with a specific tag
tag = Tag.objects.get(name='python')
posts_with_tag = tag.post_set.all()
# or
posts_with_tag = Post.objects.filter(tags=tag)
```
</details>

### Exercise 4: Fix the Relationship

What's wrong with this model definition?

```python
class Book(models.Model):
    title = models.CharField(max_length=200)
    authors = models.OneToOneField(Author, on_delete=models.CASCADE)
```

<details>
<summary>Solution</summary>

**Problem:** Using `OneToOneField` when a book can have multiple authors.

**Fix:**
```python
class Book(models.Model):
    title = models.CharField(max_length=200)
    authors = models.ManyToManyField(Author)
    # Use ManyToManyField for multiple authors
```
</details>

### Exercise 5: Design a Schema

Design models for a library system with:
- Books
- Authors (books can have multiple authors)
- Categories (each book belongs to one category)
- Members who can borrow books

<details>
<summary>Solution</summary>

```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    
    def __str__(self):
        return self.name

class Category(models.Model):
    name = models.CharField(max_length=100)
    
    def __str__(self):
        return self.name

class Book(models.Model):
    title = models.CharField(max_length=200)
    isbn = models.CharField(max_length=20, unique=True)
    category = models.ForeignKey(Category, on_delete=models.PROTECT)
    authors = models.ManyToManyField(Author)
    
    def __str__(self):
        return self.title

class Member(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    joined_date = models.DateField(auto_now_add=True)
    
    def __str__(self):
        return self.name

class BorrowRecord(models.Model):
    book = models.ForeignKey(Book, on_delete=models.CASCADE)
    member = models.ForeignKey(Member, on_delete=models.CASCADE)
    borrowed_date = models.DateField(auto_now_add=True)
    returned_date = models.DateField(null=True, blank=True)
    
    def __str__(self):
        return f"{self.member} borrowed {self.book}"
```
</details>

## Next Steps

Now that you understand the fundamentals of database relationships, continue to **[002-django-one-to-one-relationships.md](002-django-one-to-one-relationships.md)** to learn:
- Detailed implementation of OneToOneField
- The on_delete argument and its options (CASCADE, PROTECT, SET_NULL)
- Querying one-to-one relationships with field lookups
- Accessing related objects from both sides of the relationship
- Migration considerations and common pitfalls
