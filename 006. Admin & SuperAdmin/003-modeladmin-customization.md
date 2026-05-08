# ModelAdmin Customization

## Introduction

While basic model registration provides a functional admin interface, Django's ModelAdmin class offers powerful customization options to tailor the admin to your specific needs. This guide covers how to use ModelAdmin classes to control what fields are displayed, add filtering capabilities, implement search functionality, and create a more efficient and user-friendly administrative interface.

## Concept Explanation

### What is ModelAdmin?

ModelAdmin is a Django class that encapsulates all customization options for a model's admin interface. By creating a ModelAdmin subclass, you can control almost every aspect of how your model appears and behaves in the admin panel, from the list view to the form layout.

### Why Use ModelAdmin?

Basic registration (`admin.site.register(Model)`) uses default settings, which may not be optimal for your use case. ModelAdmin allows you to:

- Display specific fields in the list view
- Make fields clickable links to edit pages
- Add filters for easy data sorting
- Implement search functionality
- Control form field ordering
- Add custom actions
- Optimize database queries
- Customize permissions

### ModelAdmin Properties Overview

Django provides numerous ModelAdmin properties for customization:

- **list_display**: Controls which fields appear in the list view
- **list_display_links**: Makes specified fields clickable links
- **list_filter**: Adds sidebar filters for data filtering
- **search_fields**: Adds a search bar for text-based searching
- **ordering**: Controls default sort order
- **readonly_fields**: Makes fields read-only
- **fields**: Controls form field layout
- **exclude**: Excludes specific fields from forms

### Decorator Registration Pattern

Instead of using `admin.site.register()`, you can use the `@admin.register()` decorator for cleaner code. This decorator combines model registration with ModelAdmin class definition in a single step.

### Filter Types

Django's `list_filter` supports various filter types:
- **DateField**: Automatic date range filters (Today, Past 7 days, This month, etc.)
- **BooleanField**: Yes/No filters
- **ForeignKey**: Related object filters
- **ManyToManyField**: Multiple selection filters
- **Custom filters**: You can create custom filter classes

### Search Limitations

The `search_fields` property only works with:
- CharField
- TextField
- Related fields with `__` lookup (e.g., `author__name`)

It does NOT work with:
- IntegerField
- DateField
- BooleanField
- ForeignKey directly (use `__` lookup instead)

## Code Examples

### Basic ModelAdmin Class

Here's the transition from basic registration to ModelAdmin:

**Before (Basic Registration):**
```python
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

**After (ModelAdmin Class):**
```python
from django.contrib import admin
from .models import Post

class PostAdmin(admin.ModelAdmin):
    pass  # We'll add customization here

admin.site.register(Post, PostAdmin)
```

### Customizing list_display

Control which fields appear in the list view:

```python
from django.contrib import admin
from .models import Post

class PostAdmin(admin.ModelAdmin):
    # Display ID, title, and publish date in list view
    list_display = ['id', 'post_title', 'published_date']

admin.site.register(Post, PostAdmin)
```

**Benefits:**
- See multiple fields at a glance
- No need to click into each object to see details
- Better overview of your data

**Field Naming:**
Use the exact field names from your model:
```python
class Post(models.Model):
    post_title = models.CharField(max_length=200)
    published_date = models.DateTimeField(auto_now=True)
```

### Making Fields Clickable with list_display_links

By default, only the first field in `list_display` is clickable. Make multiple fields clickable:

```python
class PostAdmin(admin.ModelAdmin):
    list_display = ['id', 'post_title', 'published_date']
    list_display_links = ['id', 'post_title']  # Both ID and title are clickable
```

**Requirements:**
- Fields in `list_display_links` must also be in `list_display`
- At least one field must be clickable for editing

**Use Cases:**
- Make the title clickable for intuitive navigation
- Make ID clickable for quick access
- Make multiple fields clickable based on user preference

### Using the Decorator Pattern

Cleaner registration using decorators:

```python
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['id', 'post_title', 'published_date']
    list_display_links = ['id', 'post_title']
```

**Benefits:**
- Combines registration and class definition
- More concise and readable
- Follows Python decorator conventions
- Eliminates the separate `admin.site.register()` call

**Note:** When switching from `admin.site.register()` to the decorator, you may need to restart the development server to avoid "already registered" errors.

### Adding Filters with list_filter

Add sidebar filters for data filtering:

```python
class PostAdmin(admin.ModelAdmin):
    list_display = ['id', 'post_title', 'published_date', 'is_published']
    list_display_links = ['id', 'post_title']
    list_filter = ['published_date', 'is_published']  # Add filters
```

**DateField Filters:**
When filtering on a DateField or DateTimeField, Django automatically provides:
- Any date
- Today
- Past 7 days
- This month
- This year
- Custom date range

**BooleanField Filters:**
For BooleanField, Django provides:
- All
- Yes
- No

**Show/Hide Counts:**
Click "Show Counts" to see how many items match each filter option.

### Implementing Search with search_fields

Add a search bar for text-based searching:

```python
class PostAdmin(admin.ModelAdmin):
    list_display = ['id', 'post_title', 'published_date']
    list_display_links = ['id', 'post_title']
    list_filter = ['published_date']
    search_fields = ['post_title']  # Search by title
```

**Search Behavior:**
- Performs case-insensitive partial matching
- Searches across all specified fields
- Returns results matching any field
- Empty search shows all results

**Multiple Search Fields:**
```python
search_fields = ['post_title', 'post_content']  # Search both fields
```

**Searching Related Fields:**
```python
# If Post has an author ForeignKey
search_fields = ['post_title', 'author__username']  # Search author's username
```

**Limitations:**
- Only works with CharField and TextField
- Cannot search IntegerField, DateField, or BooleanField directly
- For numeric/date searches, use filters instead

### Complete Example: Fully Customized ModelAdmin

```python
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    # Display these fields in list view
    list_display = ['id', 'post_title', 'published_date', 'is_published']
    
    # Make these fields clickable
    list_display_links = ['id', 'post_title']
    
    # Add sidebar filters
    list_filter = ['published_date', 'is_published']
    
    # Add search functionality
    search_fields = ['post_title', 'post_content']
    
    # Default ordering (newest first)
    ordering = ['-published_date']
    
    # Number of items per page
    list_per_page = 25
```

### Custom Field Display Methods

You can add custom methods to `list_display`:

```python
class PostAdmin(admin.ModelAdmin):
    list_display = ['id', 'post_title', 'published_date', 'is_recent']
    
    @admin.display(boolean=True, description='Published recently?')
    def is_recent(self, obj):
        from django.utils import timezone
        return obj.published_date >= timezone.now() - timezone.timedelta(days=7)
```

### Server Restart Requirement

When switching from `admin.site.register()` to the decorator pattern, you may encounter an error:
```
django.contrib.admin.sites.AlreadyRegistered: The model Post is already registered
```

**Solution:** Restart your development server:
```bash
# Stop the server (Ctrl+C)
# Start it again
python manage.py runserver
```

This happens because Django caches admin registrations during startup.

## Key Takeaways

- ModelAdmin classes provide extensive customization for admin interfaces
- Use `list_display` to control which fields appear in list views
- Use `list_display_links` to make fields clickable for editing
- Use `list_filter` to add sidebar filters for data filtering
- Use `search_fields` to add search functionality (limited to text fields)
- The `@admin.register()` decorator provides cleaner registration syntax
- Date fields automatically get smart date range filters
- Search performs case-insensitive partial matching
- Restart the server when switching registration methods to avoid errors

## Additional Context & Best Practices

### ModelAdmin Naming Convention

Follow the convention: `ModelName + Admin`

```python
# ✅ GOOD - Clear and consistent
class PostAdmin(admin.ModelAdmin):
    pass

class UserAdmin(admin.ModelAdmin):
    pass

# ❌ BAD - Unclear naming
class PostConfiguration(admin.ModelAdmin):
    pass
```

### list_display Best Practices

**1. Show Important Fields First**
```python
list_display = ['id', 'title', 'author', 'published_date', 'status']
# Most important fields appear first
```

**2. Include Identifying Information**
Always include fields that help identify objects:
```python
list_display = ['id', 'title', 'slug']  # Multiple identifiers
```

**3. Consider Performance**
Avoid fields that trigger expensive queries:
```python
# ❌ BAD - Triggers N+1 queries
list_display = ['id', 'title', 'author']  # author is ForeignKey

# ✅ GOOD - Use select_related in get_queryset
class PostAdmin(admin.ModelAdmin):
    list_display = ['id', 'title', 'author']
    
    def get_queryset(self, request):
        return super().get_queryset(request).select_related('author')
```

### list_display_links Best Practices

**1. Make Intuitive Fields Clickable**
```python
list_display = ['id', 'title', 'published_date']
list_display_links = ['title']  # Title is most intuitive to click
```

**2. Consider User Workflow**
Think about how users navigate:
```python
# If users frequently search by ID
list_display_links = ['id', 'title']
```

**3. Don't Over-Link**
Too many clickable fields can be confusing:
```python
# ❌ BAD - Everything is clickable
list_display_links = ['id', 'title', 'published_date', 'status']

# ✅ GOOD - Selective linking
list_display_links = ['title']
```

### list_filter Best Practices

**1. Filter on Useful Fields**
Choose fields that users actually filter by:
```python
# ✅ GOOD - Users filter by these
list_filter = ['status', 'category', 'published_date']

# ❌ BAD - Rarely filtered
list_filter = ['id', 'created_at', 'updated_at']
```

**2. Use Date Filters for Time-Based Data**
```python
list_filter = ['published_date']  # Provides smart date ranges
```

**3. Combine Related Object Filters**
```python
# Filter by category and author
list_filter = ['category', 'author__name']
```

**4. Custom Filter Classes for Complex Logic**
```python
from django.contrib.admin import SimpleListFilter

class PublishedFilter(SimpleListFilter):
    title = 'Published Status'
    parameter_name = 'published'

    def lookups(self, request, model_admin):
        return (
            ('published', 'Published'),
            ('draft', 'Draft'),
        )

    def queryset(self, request, queryset):
        if self.value() == 'published':
            return queryset.filter(is_published=True)
        if self.value() == 'draft':
            return queryset.filter(is_published=False)

class PostAdmin(admin.ModelAdmin):
    list_filter = [PublishedFilter]
```

### search_fields Best Practices

**1. Search on Meaningful Text Fields**
```python
# ✅ GOOD - Users search by these
search_fields = ['title', 'content', 'summary']

# ❌ BAD - Not useful for search
search_fields = ['id', 'created_at']
```

**2. Use Related Field Lookups**
```python
# Search by author's name or email
search_fields = ['title', 'author__username', 'author__email']
```

**3. Limit Search Fields for Performance**
Too many search fields can slow down queries:
```python
# ❌ BAD - Too many fields
search_fields = ['title', 'content', 'summary', 'notes', 'description']

# ✅ GOOD - Focused search
search_fields = ['title', 'content']
```

**4. Understand Search Limitations**
Remember that `search_fields` only works with text fields:
```python
# ❌ WRONG - Won't work
search_fields = ['id', 'published_date', 'price']

# ✅ RIGHT - Use filters instead
search_fields = ['title', 'description']
list_filter = ['published_date']
```

### Decorator vs. register() Comparison

**Decorator Pattern:**
```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    pass
```
- Pros: Cleaner, more Pythonic, combines registration and definition
- Cons: Requires server restart when switching from register()

**register() Pattern:**
```python
class PostAdmin(admin.ModelAdmin):
    pass

admin.site.register(Post, PostAdmin)
```
- Pros: Explicit, easier to conditionally register
- Cons: More verbose, separate registration step

**Recommendation:** Use the decorator pattern for most cases.

### Common Pitfalls

**1. Forgetting list_display for list_display_links**
```python
# ❌ ERROR - Field not in list_display
list_display = ['id', 'title']
list_display_links = ['id', 'title', 'status']  # status not in list_display

# ✅ CORRECT
list_display = ['id', 'title', 'status']
list_display_links = ['id', 'title']
```

**2. Searching Non-Text Fields**
```python
# ❌ WRONG - Won't work
search_fields = ['id', 'price', 'published_date']

# ✅ RIGHT - Use filters
search_fields = ['title', 'description']
list_filter = ['published_date']
```

**3. Not Restarting Server After Registration Changes**
When switching registration methods, always restart the server to avoid "already registered" errors.

**4. Overloading list_display**
Too many fields can make the list view cluttered:
```python
# ❌ BAD - Too many fields
list_display = ['id', 'title', 'content', 'author', 'category', 'status', 'created_at', 'updated_at']

# ✅ GOOD - Focused fields
list_display = ['id', 'title', 'author', 'status', 'published_date']
```

**5. Inefficient Queries in list_display**
```python
# ❌ BAD - Triggers N+1 queries for each row
list_display = ['title', 'author', 'category']  # Both are ForeignKeys

# ✅ GOOD - Optimize with get_queryset
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'category']
    
    def get_queryset(self, request):
        return super().get_queryset(request).select_related('author', 'category')
```

### Performance Considerations

**1. Pagination**
Django automatically paginates list views (100 items per page by default). Customize this:
```python
list_per_page = 50  # Show 50 items per page
```

**2. Query Optimization**
Use `select_related` and `prefetch_related` to optimize queries:
```python
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'category']
    
    def get_queryset(self, request):
        return super().get_queryset(request).select_related('author', 'category')
```

**3. Database Indexes**
Ensure fields used for filtering and searching have indexes:
```python
class Post(models.Model):
    title = models.CharField(max_length=200, db_index=True)
    published_date = models.DateTimeField(db_index=True)
```

**4. Limit search_fields**
Search uses LIKE queries which can be slow on large datasets:
```python
# Limit to essential fields
search_fields = ['title']  # Instead of ['title', 'content', 'summary', 'notes']
```

### Advanced Tips

**1. Custom Actions**
Add bulk actions:
```python
@admin.action(description='Mark as published')
def make_published(modeladmin, request, queryset):
    queryset.update(is_published=True)

class PostAdmin(admin.ModelAdmin):
    actions = [make_published]
```

**2. Read-Only Fields**
Prevent modification of certain fields:
```python
readonly_fields = ['created_at', 'updated_at', 'slug']
```

**3. Field Ordering in Forms**
Control form field layout:
```python
fields = ['title', 'content', 'category', 'status']
```

**4. Exclude Fields**
Hide specific fields from forms:
```python
exclude = ['created_at', 'updated_at']
```

**5. Date Hierarchy**
Add date navigation:
```python
date_hierarchy = 'published_date'  # Adds date drill-down navigation
```

**6. Save Model Hook**
Add custom logic on save:
```python
def save_model(self, request, obj, form, change):
    obj.author = request.user
    super().save_model(request, obj, form, change)
```

## Practice Exercises

### Exercise 1: Create a Basic ModelAdmin

Create a ModelAdmin class for a Product model with the following:

**Model:**
- name (CharField)
- price (DecimalField)
- stock (IntegerField)
- is_available (BooleanField)

**Requirements:**
- Create ModelAdmin class
- Display: name, price, stock, is_available
- Make name clickable
- Register using decorator pattern

<details>
<summary>Solution</summary>

**models.py**
```python
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField()
    is_available = models.BooleanField(default=True)

    def __str__(self):
        return self.name
```

**admin.py**
```python
from django.contrib import admin
from .models import Product

@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ['name', 'price', 'stock', 'is_available']
    list_display_links = ['name']
```
</details>

### Exercise 2: Add Filters and Search

Extend the ProductAdmin from Exercise 1:

**Requirements:**
- Add filter for is_available
- Add search by product name
- Add filter for stock ranges (using a custom approach or numeric field)

<details>
<summary>Solution</summary>

**admin.py**
```python
from django.contrib import admin
from .models import Product

@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ['name', 'price', 'stock', 'is_available']
    list_display_links = ['name']
    list_filter = ['is_available']
    search_fields = ['name']
```

**Note:** For numeric stock ranges, you would need a custom filter class (advanced topic).
</details>

### Exercise 3: Customize Blog Post Admin

Create a comprehensive ModelAdmin for a BlogPost model:

**Model:**
- title (CharField)
- content (TextField)
- author (ForeignKey to User)
- category (ForeignKey to Category)
- published_date (DateTimeField)
- is_published (BooleanField)

**Requirements:**
- Display: title, author, category, published_date, is_published
- Make title and author clickable
- Filter by: published_date, is_published, category
- Search by: title, content
- Order by: published_date (newest first)
- Use decorator pattern

<details>
<summary>Solution</summary>

**models.py**
```python
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name

class BlogPost(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    published_date = models.DateTimeField(auto_now_add=True)
    is_published = models.BooleanField(default=False)

    def __str__(self):
        return self.title
```

**admin.py**
```python
from django.contrib import admin
from .models import BlogPost

@admin.register(BlogPost)
class BlogPostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'category', 'published_date', 'is_published']
    list_display_links = ['title', 'author']
    list_filter = ['published_date', 'is_published', 'category']
    search_fields = ['title', 'content']
    ordering = ['-published_date']
    
    def get_queryset(self, request):
        return super().get_queryset(request).select_related('author', 'category')
```
</details>

### Exercise 4: Switch Between Registration Methods

Practice switching between `admin.site.register()` and decorator pattern:

**Requirements:**
1. Start with `admin.site.register(Post, PostAdmin)`
2. Test in admin
3. Switch to `@admin.register(Post)` decorator
4. Restart server
5. Verify it still works

<details>
<summary>Solution</summary>

**Step 1: Using register()**
```python
from django.contrib import admin
from .models import Post

class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'published_date']

admin.site.register(Post, PostAdmin)
```

**Step 2: Switch to decorator**
```python
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'published_date']
```

**Step 3: Restart server**
```bash
# Stop server (Ctrl+C)
python manage.py runserver
```

**Step 4: Verify**
- Access admin panel
- Check that Post model appears
- Test that customization works
</details>

### Exercise 5: Implement Custom Display Method

Add a custom method to display additional information:

**Requirements:**
- Add a method to show "Recent" badge for posts published in last 7 days
- Display this in list_display
- Use the @admin.display decorator

<details>
<summary>Solution</summary>

**admin.py**
```python
from django.contrib import admin
from django.utils import timezone
import datetime
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'published_date', 'is_recent', 'is_published']
    
    @admin.display(boolean=True, description='Published recently?')
    def is_recent(self, obj):
        seven_days_ago = timezone.now() - datetime.timedelta(days=7)
        return obj.published_date >= seven_days_ago
```

**Result:**
- A checkmark appears for recent posts
- An empty space appears for older posts
- The column header shows "Published recently?"
</details>

## Next Steps

Now that you can customize ModelAdmin classes with filters, search, and display options, the final step is to customize the admin panel's branding to match your project.

Continue to **[004-admin-branding.md](004-admin-branding.md)** to learn:
- How to change the admin site header title
- How to customize the site administration text
- Additional branding opportunities
- Internationalization considerations for admin titles
