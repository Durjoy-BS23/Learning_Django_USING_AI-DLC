# Creating and Processing Forms

## Introduction

Creating and processing forms is the core of user interaction in Django applications. This guide covers how to define Django Form classes, the various field types available, the complete view pattern for handling form submissions, form validation, and the critical distinction between bound and unbound forms. Using our Post model as the example, you'll learn the complete workflow from form definition to data processing.

## Concept Explanation

### The Django Form Class

The Django Form class is the foundation of form handling. Just as Django models describe the structure of your database tables, Form classes describe the structure and behavior of your forms. A Form class defines:

- **Fields**: What data the form accepts (text, email, numbers, etc.)
- **Validation**: Rules for validating each field
- **Widgets**: How fields appear in HTML (text input, textarea, checkbox, etc.)
- **Labels**: Human-readable field names
- **Help Text**: Additional guidance for users
- **Error Messages**: Custom messages when validation fails

### Form Field Types

Django provides numerous field types, each with built-in validation:

**Common Field Types:**

| Field Type | Purpose | HTML Widget | Validation |
|------------|---------|-------------|------------|
| CharField | Text input | `<input type="text">` | Max length, required |
| EmailField | Email addresses | `<input type="email">` | Email format validation |
| IntegerField | Whole numbers | `<input type="number">` | Integer validation |
| DecimalField | Decimal numbers | `<input type="number">` | Decimal validation |
| BooleanField | True/False | `<input type="checkbox">` | Boolean validation |
| DateField | Dates | `<input type="date">` | Date format validation |
| DateTimeField | Date and time | `<input type="datetime-local">` | DateTime validation |
| TextField | Multi-line text | `<textarea>` | Max length, required |
| URLField | URLs | `<input type="url">` | URL format validation |
| ChoiceField | Single choice | `<select>` | Value in choices |
| MultipleChoiceField | Multiple choices | `<select multiple>` | Values in choices |

### The View Pattern for Forms

Django forms follow a consistent pattern in views:

**1. Check Request Method**
```python
if request.method == 'POST':
    # Process form submission
else:
    # Display empty form
```

**2. Instantiate Form with Data (POST) or Without (GET)**
```python
if request.method == 'POST':
    form = PostForm(request.POST)  # Bound form
else:
    form = PostForm()  # Unbound form
```

**3. Validate Form**
```python
if form.is_valid():
    # Process validated data
else:
    # Redisplay form with errors
```

**4. Access Cleaned Data**
```python
title = form.cleaned_data['title']
content = form.cleaned_data['content']
```

**5. Redirect on Success or Render on Failure**
```python
if form.is_valid():
    # Process and redirect
    return redirect('success_url')
return render(request, 'template.html', {'form': form})
```

### Form Validation

Django performs automatic validation when you call `form.is_valid()`:

**Validation Steps:**
1. **Field-level validation**: Each field validates its own data
2. **Form-level validation**: Cross-field validation (e.g., password confirmation)
3. **Cleaned data**: Validated data converted to Python types
4. **Error generation**: Errors collected for invalid data

**What Gets Validated:**
- Required fields are present
- Data types are correct (email format, numbers, etc.)
- Length constraints are met
- Values are within allowed ranges
- Custom validation rules you define

### Cleaned Data

When `form.is_valid()` returns `True`, the validated data is available in `form.cleaned_data`:

**Benefits of Cleaned Data:**
- Data is converted to appropriate Python types (strings to integers, dates, etc.)
- Data is validated and safe to use
- Whitespace is trimmed
- Default values are applied
- Data is ready for database operations

**Example:**
```python
# User submits "123" as quantity
# cleaned_data['quantity'] = 123 (int, not string)

# User submits "user@example.com" as email
# cleaned_data['email'] = "user@example.com" (validated email)
```

### Bound vs Unbound Forms

Understanding the distinction between bound and unbound forms is critical:

**Unbound Form:**
- Has no data associated with it
- Created without passing data: `form = PostForm()`
- Displays empty or default values
- Cannot be validated (has no data to validate)
- Used for initial form display

**Bound Form:**
- Has data bound to it
- Created with data: `form = PostForm(request.POST)`
- Can be validated
- Displays submitted data (including errors if invalid)
- Used for processing form submissions

**Checking Form State:**
```python
form = PostForm(request.POST)
print(form.is_bound)  # True - has data

form = PostForm()
print(form.is_bound)  # False - no data
```

### The Complete Form Workflow

1. **User visits URL** → View creates unbound form → Template renders empty form
2. **User fills and submits** → View receives POST request → View creates bound form
3. **Validation** → Form validates data → If invalid, redisplay with errors
4. **Success** → Access cleaned_data → Process data → Redirect to success page

## Code Examples

### Basic Form Definition

Create a form for our Post model:

```python
# forms.py
from django import forms

class PostForm(forms.Form):
    title = forms.CharField(
        max_length=200,
        label='Post Title',
        required=True,
        help_text='Enter a descriptive title (max 200 characters)',
        error_messages={
            'required': 'Please enter a title for your post',
            'max_length': 'Title cannot exceed 200 characters'
        }
    )
    
    content = forms.CharField(
        label='Post Content',
        required=True,
        widget=forms.Textarea(attrs={
            'rows': 10,
            'cols': 50,
            'placeholder': 'Write your post content here...'
        }),
        help_text='Share your thoughts with the world',
        error_messages={
            'required': 'Please enter some content for your post'
        }
    )
    
    is_published = forms.BooleanField(
        label='Publish Immediately',
        required=False,
        help_text='Check this box to publish your post right away'
    )
```

### Complete View with Form Handling

```python
# views.py
from django.shortcuts import render, redirect
from django.http import HttpResponse
from .forms import PostForm
from .models import Post

def create_post(request):
    """
    Handle post creation form.
    GET: Display empty form
    POST: Validate and process form submission
    """
    if request.method == 'POST':
        # Create form instance with submitted data (bound form)
        form = PostForm(request.POST)
        
        # Validate the form
        if form.is_valid():
            # Access cleaned, validated data
            title = form.cleaned_data['title']
            content = form.cleaned_data['content']
            is_published = form.cleaned_data['is_published']
            
            # Create the post in the database
            post = Post.objects.create(
                title=title,
                content=content,
                is_published=is_published
            )
            
            # Redirect to success page
            return redirect('post_detail', pk=post.pk)
        else:
            # Form is invalid - will be redisplayed with errors
            pass
    else:
        # GET request - create empty form (unbound form)
        form = PostForm()
    
    # Render the template with the form
    return render(request, 'create_post.html', {'form': form})
```

### URL Configuration

```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('create/', views.create_post, name='create_post'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
]
```

### Basic Template

```html
<!-- templates/create_post.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
    <style>
        .errorlist {
            color: red;
            list-style-type: none;
        }
        .helptext {
            font-size: 0.8em;
            color: #666;
        }
    </style>
</head>
<body>
    <h1>Create a New Post</h1>
    
    <form method="post">
        {% csrf_token %}
        
        {{ form.as_p }}
        
        <button type="submit">Create Post</button>
    </form>
</body>
</html>
```

### Form with Multiple Field Types

```python
# forms.py
from django import forms

class ExtendedPostForm(forms.Form):
    title = forms.CharField(max_length=200, required=True)
    
    content = forms.CharField(
        widget=forms.Textarea,
        required=True
    )
    
    category = forms.ChoiceField(
        choices=[
            ('', 'Select Category'),
            ('tech', 'Technology'),
            ('lifestyle', 'Lifestyle'),
            ('travel', 'Travel'),
        ],
        required=True
    )
    
    tags = forms.CharField(
        max_length=200,
        required=False,
        help_text='Separate tags with commas'
    )
    
    is_published = forms.BooleanField(required=False)
    
    publish_date = forms.DateField(
        required=False,
        widget=forms.DateInput(attrs={'type': 'date'})
    )
```

### Form with Custom Validation

```python
# forms.py
from django import forms

class PostForm(forms.Form):
    title = forms.CharField(max_length=200)
    content = forms.CharField(widget=forms.Textarea)
    
    def clean_title(self):
        """Custom validation for title field"""
        title = self.cleaned_data.get('title')
        
        # Check if title contains 'Django'
        if 'django' not in title.lower():
            raise forms.ValidationError(
                "Title must mention 'Django'"
            )
        
        # Check for minimum length
        if len(title) < 10:
            raise forms.ValidationError(
                "Title must be at least 10 characters long"
            )
        
        return title
    
    def clean(self):
        """Custom validation for multiple fields"""
        cleaned_data = super().clean()
        title = cleaned_data.get('title')
        content = cleaned_data.get('content')
        
        # Cross-field validation
        if title and content:
            if len(title) > len(content):
                raise forms.ValidationError(
                    "Content should be longer than title"
                )
        
        return cleaned_data
```

### Accessing Raw vs Cleaned Data

```python
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        
        if form.is_valid():
            # ✅ CORRECT - Use cleaned_data (validated)
            title = form.cleaned_data['title']
            content = form.cleaned_data['content']
            
            # ❌ WRONG - Don't use raw POST data
            # title = request.POST['title']
            # content = request.POST['content']
            
            Post.objects.create(title=title, content=content)
            return redirect('post_list')
    else:
        form = PostForm()
    
    return render(request, 'create_post.html', {'form': form})
```

### Handling Form Errors in Views

```python
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        
        if form.is_valid():
            # Process valid data
            Post.objects.create(**form.cleaned_data)
            return redirect('post_list')
        else:
            # Access errors for logging or custom handling
            errors = form.errors
            print(f"Form errors: {errors}")
    else:
        form = PostForm()
    
    return render(request, 'create_post.html', {'form': form})
```

### Pre-populating Forms with Initial Data

```python
def edit_post(request, pk):
    post = Post.objects.get(pk=pk)
    
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            # Update post
            post.title = form.cleaned_data['title']
            post.content = form.cleaned_data['content']
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        # Pre-populate form with existing data
        form = PostForm(initial={
            'title': post.title,
            'content': post.content,
            'is_published': post.is_published
        })
    
    return render(request, 'edit_post.html', {'form': form})
```

## Key Takeaways

- Django Form classes define form structure, validation, and behavior
- Field types determine HTML widgets and automatic validation
- The view pattern checks request method, instantiates forms, validates, and processes
- `form.is_valid()` performs validation and populates `cleaned_data`
- Always use `form.cleaned_data` for validated, type-converted data
- Bound forms have data (for processing), unbound forms don't (for display)
- `form.is_bound` indicates whether a form has data
- Custom validation can be added with `clean_fieldname()` and `clean()` methods
- Forms automatically repopulate with submitted data on validation errors
- Use `initial` parameter to pre-populate forms with existing data

## Additional Context & Best Practices

### Form Definition Best Practices

**1. Use Descriptive Labels**
```python
# ✅ GOOD - Clear, user-friendly labels
title = forms.CharField(label='Post Title', max_length=200)

# ❌ BAD - Unclear or missing labels
title = forms.CharField(max_length=200)
```

**2. Provide Help Text**
```python
# ✅ GOOD - Helpful guidance
content = forms.CharField(
    help_text='Write your post content here (minimum 50 characters)'
)

# ❌ BAD - No guidance
content = forms.CharField()
```

**3. Use Appropriate Field Types**
```python
# ✅ GOOD - Use specific field types
email = forms.EmailField()
price = forms.DecimalField(max_digits=10, decimal_places=2)

# ❌ BAD - Use generic CharField
email = forms.CharField()
price = forms.CharField()
```

**4. Customize Error Messages**
```python
# ✅ GOOD - User-friendly errors
title = forms.CharField(
    error_messages={
        'required': 'Please enter a title',
        'max_length': 'Title is too long'
    }
)

# ❌ BAD - Generic errors
title = forms.CharField()
```

### View Pattern Best Practices

**1. Always Validate Before Processing**
```python
# ✅ CORRECT - Validate first
if form.is_valid():
    Post.objects.create(**form.cleaned_data)

# ❌ WRONG - Process without validation
Post.objects.create(**request.POST)
```

**2. Use Redirect After Successful POST**
```python
# ✅ CORRECT - Redirect after POST
if form.is_valid():
    Post.objects.create(**form.cleaned_data)
    return redirect('post_list')

# ❌ WRONG - Render after POST (causes duplicate submissions)
if form.is_valid():
    Post.objects.create(**form.cleaned_data)
return render(request, 'success.html')
```

**3. Reuse View for GET and POST**
```python
# ✅ CORRECT - Single view handles both
def create_post(request):
    if request.method == 'POST':
        # Handle POST
    else:
        # Handle GET
    return render(...)

# ❌ WRONG - Separate views for GET and POST
def create_post_get(request):
    return render(...)

def create_post_post(request):
    # Handle POST
```

### Validation Best Practices

**1. Use Field-Level Validation for Single Fields**
```python
def clean_title(self):
    title = self.cleaned_data['title']
    if 'badword' in title.lower():
        raise forms.ValidationError('Title contains inappropriate content')
    return title
```

**2. Use Form-Level Validation for Cross-Field Checks**
```python
def clean(self):
    cleaned_data = super().clean()
    password = cleaned_data.get('password')
    confirm = cleaned_data.get('confirm_password')
    
    if password != confirm:
        raise forms.ValidationError('Passwords do not match')
    
    return cleaned_data
```

**3. Provide Helpful Error Messages**
```python
# ✅ GOOD - Specific, actionable errors
raise forms.ValidationError('Password must be at least 8 characters long')

# ❌ BAD - Vague errors
raise forms.ValidationError('Invalid password')
```

### Common Pitfalls

**1. Forgetting to Call is_valid()**
```python
# ❌ WRONG - No validation
form = PostForm(request.POST)
Post.objects.create(**request.POST)

# ✅ CORRECT - Always validate
form = PostForm(request.POST)
if form.is_valid():
    Post.objects.create(**form.cleaned_data)
```

**2. Using request.POST Instead of cleaned_data**
```python
# ❌ WRONG - Unvalidated data
title = request.POST['title']

# ✅ CORRECT - Validated data
title = form.cleaned_data['title']
```

**3. Not Handling Invalid Forms**
```python
# ❌ WRONG - Doesn't handle invalid forms
if form.is_valid():
    Post.objects.create(**form.cleaned_data)
# Missing else - what happens if invalid?

# ✅ CORRECT - Handle both cases
if form.is_valid():
    Post.objects.create(**form.cleaned_data)
    return redirect('post_list')
# Falls through to render with errors
```

**4. Confusing Bound and Unbound Forms**
```python
# ❌ WRONG - Can't validate unbound form
form = PostForm()
if form.is_valid():  # Always False
    pass

# ✅ CORRECT - Validate bound form
form = PostForm(request.POST)
if form.is_valid():
    pass
```

### Performance Considerations

**1. Form Instantiation**
Forms are lightweight. Don't worry about instantiation overhead.

**2. Validation Cost**
Validation is necessary for security. The performance cost is negligible compared to security benefits.

**3. Database Operations**
Always validate before database operations to prevent unnecessary queries on invalid data.

### Advanced Tips

**1. Dynamic Field Choices**
```python
class PostForm(forms.Form):
    category = forms.ChoiceField(choices=[])
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['category'].choices = [
            (cat.id, cat.name) for cat in Category.objects.all()
        ]
```

**2. Conditional Fields**
```python
class PostForm(forms.Form):
    has_featured_image = forms.BooleanField(required=False)
    featured_image = forms.ImageField(required=False)
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        if not self.data.get('has_featured_image'):
            self.fields['featured_image'].required = False
```

**3. Field Sets for Organization**
```python
class PostForm(forms.Form):
    title = forms.CharField()
    content = forms.CharField(widget=forms.Textarea)
    
    class Meta:
        fieldsets = (
            ('Basic Info', {'fields': ('title',)}),
            ('Content', {'fields': ('content',)}),
        )
```

## Practice Exercises

### Exercise 1: Create a Contact Form

Create a contact form with the following fields:
- name (CharField, required, max 100)
- email (EmailField, required)
- subject (CharField, required, max 200)
- message (CharField with Textarea, required)

<details>
<summary>Solution</summary>

```python
# forms.py
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(
        max_length=100,
        label='Your Name',
        required=True,
        help_text='Enter your full name'
    )
    
    email = forms.EmailField(
        label='Your Email',
        required=True,
        help_text='We will reply to this address'
    )
    
    subject = forms.CharField(
        max_length=200,
        label='Subject',
        required=True,
        help_text='Brief description of your message'
    )
    
    message = forms.CharField(
        label='Message',
        required=True,
        widget=forms.Textarea(attrs={'rows': 6}),
        help_text='Write your message here'
    )
```
</details>

### Exercise 2: Create a View for the Contact Form

Create a view that:
- Displays the empty form on GET
- Validates the form on POST
- Prints the cleaned data to console on success
- Redisplay the form with errors on validation failure

<details>
<summary>Solution</summary>

```python
# views.py
from django.shortcuts import render
from .forms import ContactForm

def contact(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        
        if form.is_valid():
            # Access cleaned data
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            subject = form.cleaned_data['subject']
            message = form.cleaned_data['message']
            
            # Print to console (in real app, send email)
            print(f"Contact from: {name} ({email})")
            print(f"Subject: {subject}")
            print(f"Message: {message}")
            
            # Here you would typically send an email
            # and redirect to a success page
            return render(request, 'contact_success.html')
    else:
        form = ContactForm()
    
    return render(request, 'contact.html', {'form': form})
```
</details>

### Exercise 3: Add Custom Validation

Add custom validation to the PostForm to ensure:
- Title is at least 10 characters long
- Content is at least 50 characters long
- Title doesn't contain the word "spam"

<details>
<summary>Solution</summary>

```python
# forms.py
from django import forms

class PostForm(forms.Form):
    title = forms.CharField(max_length=200)
    content = forms.CharField(widget=forms.Textarea)
    
    def clean_title(self):
        title = self.cleaned_data.get('title')
        
        # Check minimum length
        if len(title) < 10:
            raise forms.ValidationError(
                'Title must be at least 10 characters long'
            )
        
        # Check for spam
        if 'spam' in title.lower():
            raise forms.ValidationError(
                'Title cannot contain the word "spam"'
            )
        
        return title
    
    def clean_content(self):
        content = self.cleaned_data.get('content')
        
        # Check minimum length
        if len(content) < 50:
            raise forms.ValidationError(
                'Content must be at least 50 characters long'
            )
        
        return content
```
</details>

### Exercise 4: Pre-populate Form for Editing

Create an edit_post view that:
- Takes a post ID as parameter
- Pre-populates the form with existing post data
- Updates the post on successful form submission

<details>
<summary>Solution</summary>

```python
# views.py
from django.shortcuts import render, redirect, get_object_or_404
from .forms import PostForm
from .models import Post

def edit_post(request, pk):
    # Get the post or return 404
    post = get_object_or_404(Post, pk=pk)
    
    if request.method == 'POST':
        form = PostForm(request.POST)
        
        if form.is_valid():
            # Update the post with cleaned data
            post.title = form.cleaned_data['title']
            post.content = form.cleaned_data['content']
            post.is_published = form.cleaned_data.get('is_published', False)
            post.save()
            
            return redirect('post_detail', pk=post.pk)
    else:
        # Pre-populate form with existing data
        form = PostForm(initial={
            'title': post.title,
            'content': post.content,
            'is_published': post.is_published
        })
    
    return render(request, 'edit_post.html', {
        'form': form,
        'post': post
    })
```
</details>

### Exercise 5: Identify Bound vs Unbound Forms

For each scenario, determine if the form is bound or unbound:

1. `form = PostForm()`
2. `form = PostForm(request.POST)`
3. `form = PostForm(initial={'title': 'Hello'})`
4. `form = PostForm(request.GET)`
5. `form = PostForm(data={'title': 'Test'})`

<details>
<summary>Solution</summary>

1. **Unbound** - No data passed
2. **Bound** - Has POST data
3. **Unbound** - initial data doesn't bind the form
4. **Bound** - Has GET data
5. **Bound** - Has data parameter
</details>

## Next Steps

Now that you understand how to create and process Django forms, the next step is to learn how to render forms in templates and handle the display of forms and errors.

Continue to **[003-form-rendering-templates.md](003-form-rendering-templates.md)** to learn:
- How to render forms in templates using various methods
- CSRF protection implementation
- Manual field rendering for custom layouts
- Displaying form errors to users
- Looping over form fields dynamically
- Handling hidden vs visible fields
- Customizing form appearance with CSS
