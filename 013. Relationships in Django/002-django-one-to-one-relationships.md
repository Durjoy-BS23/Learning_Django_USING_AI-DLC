# Django One-to-One Relationships

## Introduction

One-to-one relationships connect two models where each record in one model relates to exactly one record in another model, and vice versa. This is useful when you want to extend a model with additional information, split large models into smaller focused ones, or model unique associations like a person having a single passport. Django provides the `OneToOneField` to implement these relationships with built-in data integrity and efficient querying.

## Concept Explanation

### What Is a One-to-One Relationship?

A one-to-one relationship ensures that:
- One record in Model A can be linked to only one record in Model B
- One record in Model B can be linked to only one record in Model A
- The relationship is bidirectional and unique on both sides

**Real-World Examples:**
- Person ↔ Passport (one person has one passport)
- User ↔ Profile (one user has one profile)
- Employee ↔ Office (one employee has one office)
- Car ↔ License Plate (one car has one license plate)

### When to Use One-to-One Relationships

**1. Extending Models:**
Add optional or supplementary data to an existing model without modifying it directly.
```python
# User model (built-in)
# Profile model extends User with additional fields
```

**2. Splitting Large Models:**
Break down a model with many fields into smaller, focused models for better organization.
```python
# Product model (basic info)
# ProductDetails model (extended info, specs, reviews)
```

**3. Unique Associations:**
Model relationships where uniqueness is required.
```python
# Employee ↔ ParkingSpot (each employee has one parking spot)
```

**4. Optional Data:**
Store data that may not always exist for every record.
```python
# User ↔ Profile (not all users may have a profile)
```

### OneToOneField Parameters

The `OneToOneField` requires two main parameters:

**`to`:**
- The model to which the relationship is established
- Can be a model class or string reference (to avoid circular imports)
- Required parameter

**`on_delete`:**
- Specifies what happens when the referenced object is deleted
- Required parameter (Django 2.0+)
- Options: CASCADE, PROTECT, SET_NULL, SET_DEFAULT, DO_NOTHING, SET()

### on_delete Argument Explained

The `on_delete` argument is crucial for maintaining data integrity when related objects are deleted.

**CASCADE:**
- When the referenced object is deleted, also delete the related object
- Use when the related object cannot exist without the parent
- Example: Delete user → also delete their profile

**PROTECT:**
- Prevents deletion of the referenced object if related objects exist
- Raises `ProtectedError` if you try to delete
- Use when you want to enforce that the relationship must be manually resolved first
- Example: Prevent deleting a user if they have active orders

**SET_NULL:**
- Sets the ForeignKey to NULL when the referenced object is deleted
- Requires `null=True` on the field
- Use when the relationship is optional
- Example: Delete employee → set their office to NULL

**SET_DEFAULT:**
- Sets the ForeignKey to its default value when the referenced object is deleted
- Requires a `default` value to be set
- Use when you want to fallback to a default object

**DO_NOTHING:**
- Takes no action when the referenced object is deleted
- Database integrity may be compromised if not handled at database level
- Rarely used, requires database-level constraints

**SET():**
- Sets the ForeignKey to the value returned by a callable
- Useful for dynamic default values

### Reverse Relationship Access

Django automatically creates a reverse relationship on the referenced model:
- Access the related model by using the lowercase model name
- For example, if `Profile` has `user = OneToOneField(User)`, you can access `user.profile`

### Uniqueness Constraint

OneToOneField automatically adds a unique constraint to the database, ensuring that each record can only be linked to one other record.

## Code Examples

### Basic One-to-One Relationship

**models.py:**
```python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    
    def __str__(self):
        return self.name

class Passport(models.Model):
    person = models.OneToOneField(Person, on_delete=models.CASCADE)
    passport_number = models.CharField(max_length=20, unique=True)
    expiry_date = models.DateField()
    
    def __str__(self):
        return f"Passport {self.passport_number}"
```

**Creating Objects:**
```python
# Create a person
person = Person.objects.create(
    name="John Doe",
    email="john@example.com"
)

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

### on_delete Examples

**CASCADE (Delete Related):**
```python
class User(models.Model):
    username = models.CharField(max_length=100)

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField()

# When you delete the user, the profile is also deleted
user = User.objects.get(username='john')
user.delete()  # Profile is automatically deleted too
```

**PROTECT (Prevent Deletion):**
```python
class Employee(models.Model):
    name = models.CharField(max_length=100)

class Office(models.Model):
    employee = models.OneToOneField(Employee, on_delete=models.PROTECT)
    office_number = models.CharField(max_length=10)

# Trying to delete an employee with an office will raise ProtectedError
employee = Employee.objects.get(name='john')
employee.delete()  # Raises ProtectedError if office exists
```

**SET_NULL (Set to NULL):**
```python
class Employee(models.Model):
    name = models.CharField(max_length=100)

class Office(models.Model):
    employee = models.OneToOneField(
        Employee, 
        on_delete=models.SET_NULL, 
        null=True
    )
    office_number = models.CharField(max_length=10)

# When employee is deleted, office.employee becomes NULL
employee = Employee.objects.get(name='john')
employee.delete()  # Office remains, but employee field is NULL
```

### String Reference to Avoid Circular Imports

```python
# models.py
class Profile(models.Model):
    # Use string reference to avoid circular import
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE)
    bio = models.TextField()
```

### Querying One-to-One Relationships

**models.py:**
```python
from django.db import models

class Husband(models.Model):
    name = models.CharField(max_length=100)
    
    def __str__(self):
        return self.name

class Wife(models.Model):
    name = models.CharField(max_length=100)
    husband = models.OneToOneField(Husband, on_delete=models.CASCADE)
    
    def __str__(self):
        return self.name
```

**Querying with Field Lookups:**
```python
# Get wife whose husband's name starts with 'J'
wives = Wife.objects.filter(husband__name__startswith='J')

# Get wife whose husband's name is 'John'
wife = Wife.objects.get(husband__name='John')

# Get all husbands who have a wife
husbands_with_wives = Husband.objects.filter(wife__isnull=False)
```

**Accessing Related Objects:**
```python
husband = Husband.objects.get(name='John')

# Access the wife
wife = husband.wife
print(wife.name)  # Prints the wife's name

# Access husband from wife
wife = Wife.objects.get(name='Jane')
print(wife.husband.name)  # Prints the husband's name
```

### Creating Objects with Relationships

```python
# Method 1: Create objects separately then link
husband = Husband.objects.create(name='John')
wife = Wife.objects.create(name='Jane', husband=husband)

# Method 2: Create related object first
husband = Husband.objects.create(name='John')
wife = Wife(name='Jane')
wife.husband = husband
wife.save()

# Method 3: Retrieve existing object and link
husband = Husband.objects.get(name='John')
wife = Wife.objects.create(name='Jane', husband=husband)
```

### Admin Configuration

**admin.py:**
```python
from django.contrib import admin
from .models import Husband, Wife

@admin.register(Husband)
class HusbandAdmin(admin.ModelAdmin):
    list_display = ('name',)
    search_fields = ('name',)

@admin.register(Wife)
class WifeAdmin(admin.ModelAdmin):
    list_display = ('name', 'husband')
    list_select_related = ('husband',)  # Optimizes queries
    search_fields = ('name', 'husband__name')
```

### Complete Example with User Profile

**models.py:**
```python
from django.db import models
from django.contrib.auth.models import User

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(max_length=500, blank=True)
    location = models.CharField(max_length=100, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    
    def __str__(self):
        return f"{self.user.username}'s profile"
```

**signals.py (Auto-create profile):**
```python
from django.db.models.signals import post_save
from django.contrib.auth.models import User
from django.dispatch import receiver
from .models import Profile

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

**apps.py:**
```python
from django.apps import AppConfig

class UsersConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'users'

    def ready(self):
        import users.signals
```

## Key Takeaways

- OneToOneField creates a unique 1:1 relationship between two models
- Requires `to` (referenced model) and `on_delete` (deletion behavior) parameters
- `on_delete=CASCADE` deletes related objects when parent is deleted
- `on_delete=PROTECT` prevents deletion if related objects exist
- `on_delete=SET_NULL` sets field to NULL (requires `null=True`)
- Access related objects using lowercase model name (e.g., `user.profile`)
- Query across relationships using double underscore syntax (`__`)
- Use string references (`'ModelName'`) to avoid circular imports
- Django automatically adds unique constraint to enforce one-to-one
- Ideal for extending models, splitting large models, and unique associations

## Additional Context & Best Practices

### Performance Optimization

**Use select_related for One-to-One:**
```python
# ❌ BAD - N+1 queries
profiles = Profile.objects.all()
for profile in profiles:
    print(profile.user.username)  # Separate query each time

# ✅ GOOD - Single query
profiles = Profile.objects.select_related('user').all()
for profile in profiles:
    print(profile.user.username)  # No additional queries
```

**Use prefetch_related for reverse relationships:**
```python
# ✅ GOOD - Optimized reverse queries
users = User.objects.prefetch_related('profile').all()
for user in users:
    print(user.profile.bio)  # Efficiently loaded
```

### Common Pitfalls

**1. Forgetting on_delete:**
```python
# ❌ WRONG - Missing on_delete (Django 2.0+ error)
class Profile(models.Model):
    user = models.OneToOneField(User)

# ✅ CORRECT - Always specify on_delete
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
```

**2. SET_NULL without null=True:**
```python
# ❌ WRONG - SET_NULL requires null=True
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.SET_NULL)

# ✅ CORRECT - Add null=True
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.SET_NULL, null=True)
```

**3. Circular Imports:**
```python
# ❌ WRONG - Can cause circular import
from .models import User

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)

# ✅ CORRECT - Use string reference
class Profile(models.Model):
    user = models.OneToOneField('User', on_delete=models.CASCADE)
```

**4. Accessing Non-Existent Related Object:**
```python
# ❌ WRONG - Will raise DoesNotExist if no profile exists
profile = user.profile

# ✅ CORRECT - Use hasattr or try/except
if hasattr(user, 'profile'):
    profile = user.profile

# or
try:
    profile = user.profile
except Profile.DoesNotExist:
    profile = None
```

### Best Practices

**1. Add __str__ Methods:**
```python
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    
    def __str__(self):
        return f"{self.user.username}'s profile"
```

**2. Use Descriptive Field Names:**
```python
# ✅ GOOD - Clear field name
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)

# ❌ BAD - Ambiguous field name
class Profile(models.Model):
    related_user = models.OneToOneField(User, on_delete=models.CASCADE)
```

**3. Consider Using Signals for Auto-Creation:**
```python
# Auto-create profile when user is created
@receiver(post_save, sender=User)
def create_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)
```

**4. Use Related Name for Clarity:**
```python
class Profile(models.Model):
    user = models.OneToOneField(
        User, 
        on_delete=models.CASCADE,
        related_name='profile'  # Explicit, though this is the default
    )
```

### Migration Considerations

**Adding OneToOneField to Existing Model:**
```python
# When adding to existing model with data, provide default or allow null
class Profile(models.Model):
    user = models.OneToOneField(
        User, 
        on_delete=models.CASCADE,
        null=True,  # Required if User objects already exist
        blank=True
    )
```

**Removing OneToOneField:**
- Django will handle dropping the foreign key constraint
- Consider data migration if you need to preserve relationships

### Database-Level Behavior

**Unique Constraint:**
- OneToOneField adds a unique constraint at the database level
- This ensures data integrity even if application logic fails
- Cannot have two records pointing to the same related object

**Indexing:**
- Django automatically indexes the field
- Improves query performance for lookups
- No need to add manual index

## Practice Exercises

### Exercise 1: Create One-to-One Relationship

Create models for a library system where each member has one library card.

<details>
<summary>Solution</summary>

```python
from django.db import models

class Member(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    
    def __str__(self):
        return self.name

class LibraryCard(models.Model):
    member = models.OneToOneField(Member, on_delete=models.CASCADE)
    card_number = models.CharField(max_length=20, unique=True)
    issue_date = models.DateField(auto_now_add=True)
    
    def __str__(self):
        return f"Card {self.card_number}"
```
</details>

### Exercise 2: Implement Different on_delete Options

Create models for Employee and Office with different on_delete behaviors based on business rules.

<details>
<summary>Solution</summary>

```python
from django.db import models

class Employee(models.Model):
    name = models.CharField(max_length=100)
    
    def __str__(self):
        return self.name

class Office(models.Model):
    # Option 1: CASCADE - delete office when employee leaves
    owner = models.OneToOneField(
        Employee, 
        on_delete=models.CASCADE,
        related_name='owned_office'
    )
    office_number = models.CharField(max_length=10)
    
    # Option 2: PROTECT - can't delete employee if they have office
    # occupant = models.OneToOneField(
    #     Employee,
    #     on_delete=models.PROTECT,
    #     related_name='assigned_office'
    # )
    
    # Option 3: SET_NULL - office remains but becomes unassigned
    # occupant = models.OneToOneField(
    #     Employee,
    #     on_delete=models.SET_NULL,
    #     null=True,
    #     blank=True,
    #     related_name='assigned_office'
    # )
    
    def __str__(self):
        return f"Office {self.office_number}"
```
</details>

### Exercise 3: Query One-to-One Relationships

Given Husband and Wife models, write queries to:
1. Get all husbands who have a wife
2. Get wife whose husband's name contains 'John'
3. Get husband whose wife's name starts with 'M'

<details>
<summary>Solution</summary>

```python
# 1. Get all husbands who have a wife
husbands_with_wives = Husband.objects.filter(wife__isnull=False)

# 2. Get wife whose husband's name contains 'John'
wife = Wife.objects.get(husband__name__icontains='john')

# 3. Get husband whose wife's name starts with 'M'
husband = Husband.objects.get(wife__name__startswith='M')
```
</details>

### Exercise 4: Handle DoesNotExist

Write a function to safely get a user's profile, returning None if it doesn't exist.

<details>
<summary>Solution</summary>

```python
from django.contrib.auth.models import User
from .models import Profile

def get_user_profile(user):
    try:
        return user.profile
    except Profile.DoesNotExist:
        return None

# Alternative using hasattr
def get_user_profile(user):
    if hasattr(user, 'profile'):
        return user.profile
    return None
```
</details>

### Exercise 5: Create User Profile with Auto-Creation

Implement a User profile model with signal to auto-create profile when user is created.

<details>
<summary>Solution</summary>

```python
# models.py
from django.db import models
from django.contrib.auth.models import User

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(blank=True)
    
    def __str__(self):
        return f"{self.user.username}'s profile"

# signals.py
from django.db.models.signals import post_save
from django.contrib.auth.models import User
from django.dispatch import receiver
from .models import Profile

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()

# apps.py
from django.apps import AppConfig

class UsersConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'users'

    def ready(self):
        import users.signals
```
</details>

## Next Steps

Now that you understand one-to-one relationships, continue to **[003-django-many-to-one-relationships.md](003-django-many-to-one-relationships.md)** to learn:
- ForeignKey field for many-to-one relationships
- Creating comment systems with post relationships
- Querying foreign keys and related objects
- Using the `_set` accessor for reverse relationships
- Adding user fields to comments with authentication
- Rendering related objects in templates
- Common patterns for many-to-one relationships
