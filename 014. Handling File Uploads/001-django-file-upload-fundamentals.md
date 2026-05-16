# Django File Upload Fundamentals

## Introduction

File upload functionality is essential for many web applications, allowing users to submit documents, images, videos, and other files. Django provides robust built-in support for handling file uploads through specialized model fields and form handling. Understanding how to properly handle file uploads is crucial for building data-driven applications that go beyond simple text input.

## Concept Explanation

### Why File Uploads Matter

Modern web applications frequently need to handle user-uploaded content:
- Profile pictures and avatars
- Document submissions (resumes, reports, PDFs)
- Media content (images, videos, audio)
- Data files (CSVs, Excel spreadsheets)
- User-generated content

### How Django Handles File Uploads

**Key Concept: Files Are Not Stored in Database**

Django follows best practices by NOT storing the actual file content in the database. Instead:
- The file is saved to the filesystem (disk)
- The database stores only the file path as a string
- This approach provides better performance and scalability

**Why This Matters:**
- **Performance**: Databases are optimized for structured data, not binary blobs
- **Scalability**: Large files can slow down database operations and backups
- **Cost**: Database storage is typically more expensive than filesystem storage
- **Flexibility**: Files can be served directly by web servers or CDNs

### FileField: The Foundation

Django's `FileField` is the primary field type for handling file uploads:
- Accepts any type of file (images, documents, etc.)
- Requires the `upload_to` parameter to specify storage location
- Stores the relative path to the file in the database
- Provides methods for accessing file properties

**upload_to Parameter:**
- Specifies the subdirectory within MEDIA_ROOT where files are stored
- Can be a string (static path) or a callable (dynamic path based on file/instance)
- Django creates the directory automatically if it doesn't exist
- Example: `upload_to='documents/'` stores files in `MEDIA_ROOT/documents/`

### The File Upload Process

**1. User submits form with file:**
- HTML form must have `enctype="multipart/form-data"`
- File is sent as part of the HTTP request body

**2. Django receives the request:**
- File data is accessible via `request.FILES`
- `request.FILES` is a dictionary-like object containing all uploaded files

**3. Form processing:**
- ModelForm automatically handles file field
- Must pass `request.FILES` to the form constructor
- Django validates and saves the file to disk

**4. Database storage:**
- Only the file path is saved in the database
- Path is relative to MEDIA_ROOT

**5. File serving:**
- Files are served from the filesystem, not database
- Requires additional configuration (covered in next guide)

### MEDIA_ROOT Configuration

`MEDIA_ROOT` is a critical setting in `settings.py`:
- Specifies the absolute path where user-uploaded files are stored
- Django creates this directory if it doesn't exist
- Should be outside the web root for security (in production)
- Example: `MEDIA_ROOT = BASE_DIR / 'media'`

**Directory Structure:**
```
project/
├── media/                    # MEDIA_ROOT - all user uploads go here
│   ├── documents/           # upload_to='documents/'
│   └── user_files/          # upload_to='user_files/'
├── myapp/
├── manage.py
└── settings.py
```

### multipart/form-data: Critical Form Attribute

When handling file uploads, the HTML form must include:
```html
<form method="post" enctype="multipart/form-data">
```

**Why It's Required:**
- Default encoding (`application/x-www-form-urlencoded`) doesn't support file uploads
- `multipart/form-data` allows binary data transmission
- Without it, file data won't be sent to the server

**What Happens Without It:**
- `request.FILES` will be empty
- File upload will fail silently or with error

### request.FILES: Accessing Uploaded Files

`request.FILES` is a dictionary-like object:
- Keys match the field names from your model/form
- Values are `UploadedFile` objects
- Contains metadata (name, size, content_type, etc.)

**Example:**
```python
# Access a specific file
uploaded_file = request.FILES['file_field']
print(uploaded_file.name)  # Original filename
print(uploaded_file.size)  # File size in bytes
```

## Code Examples

### Basic File Upload Setup

**models.py:**
```python
from django.db import models

class UserData(models.Model):
    username = models.CharField(max_length=255)
    file = models.FileField(upload_to='user_files/')
    
    def __str__(self):
        return self.username
```

**settings.py:**
```python
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# Configure media root for file storage
MEDIA_ROOT = BASE_DIR / 'media'
```

**forms.py:**
```python
from django import forms
from .models import UserData

class UserDataForm(forms.ModelForm):
    class Meta:
        model = UserData
        fields = ['username', 'file']
```

**views.py:**
```python
from django.shortcuts import render, redirect
from .forms import UserDataForm

def upload_file(request):
    if request.method == 'POST':
        form = UserDataForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('upload_success')
    else:
        form = UserDataForm()
    
    return render(request, 'upload.html', {'form': form})
```

**upload.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>File Upload</title>
</head>
<body>
    <h1>Upload File</h1>
    <!-- CRITICAL: enctype="multipart/form-data" is required -->
    <form method="post" enctype="multipart/form-data">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Upload</button>
    </form>
</body>
</html>
```

### Complete Working Example

**urls.py (app level):**
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.upload_file, name='upload'),
]
```

**urls.py (project level):**
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('upload/', include('file_uploads.urls')),
]
```

**admin.py:**
```python
from django.contrib import admin
from .models import UserData

@admin.register(UserData)
class UserDataAdmin(admin.ModelAdmin):
    list_display = ('username', 'file')
```

### Dynamic upload_to with Callable

**models.py:**
```python
import os
from django.db import models
from django.utils import timezone

def user_directory_path(instance, filename):
    # File will be uploaded to MEDIA_ROOT/user_<id>/<filename>
    return f'user_{instance.id}/{filename}'

class UserData(models.Model):
    username = models.CharField(max_length=255)
    file = models.FileField(upload_to=user_directory_path)
    
    def __str__(self):
        return self.username
```

**Date-based upload_to:**
```python
from django.utils import timezone

def upload_to_date(instance, filename):
    # File will be uploaded to MEDIA_ROOT/year/month/<filename>
    now = timezone.now()
    return f'{now.year}/{now.month}/{filename}'

class Document(models.Model):
    title = models.CharField(max_length=200)
    file = models.FileField(upload_to=upload_to_date)
```

### Accessing File Properties

**views.py:**
```python
from django.shortcuts import render
from .models import UserData

def file_list(request):
    user_data_list = UserData.objects.all()
    
    for user_data in user_data_list:
        if user_data.file:
            print(f"Filename: {user_data.file.name}")
            print(f"URL: {user_data.file.url}")  # Requires MEDIA_URL configuration
            print(f"Path: {user_data.file.path}")
            print(f"Size: {user_data.file.size} bytes")
    
    return render(request, 'file_list.html', {'user_data_list': user_data_list})
```

### Manual File Handling (Without ModelForm)

**views.py:**
```python
from django.shortcuts import render, redirect
from .models import UserData

def manual_upload(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        uploaded_file = request.FILES.get('file')
        
        if uploaded_file:
            user_data = UserData(username=username)
            # Manually assign the file
            user_data.file.save(uploaded_file.name, uploaded_file)
            user_data.save()
            return redirect('upload_success')
    
    return render(request, 'manual_upload.html')
```

## Key Takeaways

- Django stores file paths in the database, not actual file content
- FileField accepts any file type and requires the `upload_to` parameter
- `MEDIA_ROOT` in settings.py specifies where files are stored on disk
- HTML forms must have `enctype="multipart/form-data"` for file uploads
- `request.FILES` contains uploaded files, accessed by field name
- Pass `request.FILES` to ModelForm constructor to handle file uploads
- `upload_to` can be a static string or a callable for dynamic paths
- Files are saved to filesystem before the database record is created
- Always configure MEDIA_ROOT before implementing file uploads

## Additional Context & Best Practices

### Security Considerations

**1. File Type Validation:**
```python
# models.py - Validate file extensions
class Document(models.Model):
    title = models.CharField(max_length=200)
    file = models.FileField(upload_to='documents/')
    
    def clean(self):
        from django.core.exceptions import ValidationError
        import os
        ext = os.path.splitext(self.file.name)[1]  # Get file extension
        valid_extensions = ['.pdf', '.doc', '.docx', '.txt']
        if ext.lower() not in valid_extensions:
            raise ValidationError('Unsupported file extension.')
```

**2. File Size Limits:**
```python
# settings.py
MAX_UPLOAD_SIZE = 5242880  # 5MB in bytes

# views.py - Validate file size
def upload_file(request):
    if request.method == 'POST':
        uploaded_file = request.FILES.get('file')
        if uploaded_file and uploaded_file.size > MAX_UPLOAD_SIZE:
            # Handle file too large error
            pass
```

**3. Sanitize Filenames:**
```python
import os
from django.utils.text import slugify

def sanitize_filename(filename):
    name, ext = os.path.splitext(filename)
    return f"{slugify(name)}{ext}"

# In your view
clean_name = sanitize_filename(uploaded_file.name)
```

### Performance Best Practices

**1. Use CDN for Production:**
- In production, serve files from a CDN (CloudFront, S3, etc.)
- Use django-storages for cloud storage backends
- Configure MEDIA_ROOT for local development only

**2. Optimize File Storage:**
```python
# Compress images before saving (requires PIL/Pillow)
from PIL import Image
import io

def compress_image(image_field):
    img = Image.open(image_field)
    img.save(io.BytesIO(), format='JPEG', quality=85)
```

**3. Asynchronous File Processing:**
- Use Celery for large file processing
- Don't block the request/response cycle with file operations
- Consider background tasks for file conversion, resizing, etc.

### Common Pitfalls

**1. Missing enctype Attribute:**
```html
<!-- ❌ WRONG - File won't upload -->
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
</form>

<!-- ✅ CORRECT - File uploads work -->
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
</form>
```

**2. Forgetting request.FILES:**
```python
# ❌ WRONG - File data not passed to form
form = UserDataForm(request.POST)

# ✅ CORRECT - File data included
form = UserDataForm(request.POST, request.FILES)
```

**3. Not Configuring MEDIA_ROOT:**
```python
# ❌ WRONG - Files stored in project root or unpredictable location
# No MEDIA_ROOT configured

# ✅ CORRECT - Files stored in dedicated location
MEDIA_ROOT = BASE_DIR / 'media'
```

**4. Storing Files in Database:**
```python
# ❌ WRONG - Storing file content in database (bad practice)
class Document(models.Model):
    title = models.CharField(max_length=200)
    file_content = models.BinaryField()  # Don't do this!

# ✅ CORRECT - Storing file path in database
class Document(models.Model):
    title = models.CharField(max_length=200)
    file = models.FileField(upload_to='documents/')
```

### Database Considerations

**Migration with Existing Data:**
```python
# When adding FileField to existing model with data
class Post(models.Model):
    title = models.CharField(max_length=200)
    # Adding new field to existing model
    image = models.ImageField(upload_to='posts/', null=True, blank=True)
    # null=True, blank=True allows existing records without images
```

**File Deletion:**
- Deleting a model instance doesn't automatically delete the file
- Use signals or override delete() method for cleanup
- Consider using django-cleanup package for automatic file cleanup

```python
from django.db.models.signals import post_delete
from django.dispatch import receiver
from .models import UserData

@receiver(post_delete, sender=UserData)
def delete_file_on_delete(sender, instance, **kwargs):
    if instance.file:
        instance.file.delete(save=False)
```

### File Naming Strategies

**1. Unique Filenames:**
```python
import uuid
import os

def get_file_path(instance, filename):
    ext = filename.split('.')[-1]
    filename = f"{uuid.uuid4()}.{ext}"
    return os.path.join('uploads', filename)

class Document(models.Model):
    file = models.FileField(upload_to=get_file_path)
```

**2. Preserve Original Name:**
```python
def upload_to_username(instance, filename):
    return f'{instance.username}/{filename}'
```

**3. Timestamp-based Naming:**
```python
from django.utils import timezone

def upload_to_timestamp(instance, filename):
    timestamp = timezone.now().strftime('%Y%m%d_%H%M%S')
    name, ext = os.path.splitext(filename)
    return f'uploads/{timestamp}_{name}{ext}'
```

## Practice Exercises

### Exercise 1: Basic File Upload

Create a simple file upload application that allows users to upload documents and stores them in a `documents/` directory.

<details>
<summary>Solution</summary>

```python
# models.py
from django.db import models

class Document(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    file = models.FileField(upload_to='documents/')
    uploaded_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title

# forms.py
from django import forms
from .models import Document

class DocumentForm(forms.ModelForm):
    class Meta:
        model = Document
        fields = ['title', 'description', 'file']

# views.py
from django.shortcuts import render, redirect
from .forms import DocumentForm

def upload_document(request):
    if request.method == 'POST':
        form = DocumentForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('document_list')
    else:
        form = DocumentForm()
    
    return render(request, 'upload.html', {'form': form})

# settings.py
MEDIA_ROOT = BASE_DIR / 'media'
```
</details>

### Exercise 2: Fix the Broken Upload

What's wrong with this code and how would you fix it?

```python
def upload_file(request):
    if request.method == 'POST':
        form = UserDataForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('success')
    return render(request, 'upload.html', {'form': UserDataForm()})
```

<details>
<summary>Solution</summary>

**Problem:** Missing `request.FILES` in form constructor.

```python
# ✅ CORRECT
def upload_file(request):
    if request.method == 'POST':
        form = UserDataForm(request.POST, request.FILES)  # Added request.FILES
        if form.is_valid():
            form.save()
            return redirect('success')
    else:
        form = UserDataForm()  # Create form for GET request
    
    return render(request, 'upload.html', {'form': form})
```
</details>

### Exercise 3: Dynamic File Path

Create a model that stores user profile pictures in directories organized by username.

<details>
<summary>Solution</summary>

```python
from django.db import models

def user_profile_path(instance, filename):
    return f'profiles/{instance.username}/{filename}'

class UserProfile(models.Model):
    username = models.CharField(max_length=100, unique=True)
    profile_picture = models.ImageField(upload_to=user_profile_path)
    
    def __str__(self):
        return self.username
```
</details>

### Exercise 4: File Size Validation

Add file size validation to prevent uploads larger than 10MB.

<details>
<summary>Solution</summary>

```python
from django.db import models
from django.core.exceptions import ValidationError

class Document(models.Model):
    title = models.CharField(max_length=200)
    file = models.FileField(upload_to='documents/')
    
    def clean(self):
        super().clean()
        if self.file and self.file.size > 10 * 1024 * 1024:  # 10MB
            raise ValidationError('File size cannot exceed 10MB.')
```

Or in the view:
```python
MAX_SIZE = 10 * 1024 * 1024  # 10MB

def upload_document(request):
    if request.method == 'POST':
        uploaded_file = request.FILES.get('file')
        if uploaded_file and uploaded_file.size > MAX_SIZE:
            # Handle error
            return render(request, 'upload.html', {
                'form': form,
                'error': 'File too large (max 10MB)'
            })
```
</details>

### Exercise 5: Configure Complete File Upload

Set up a complete file upload system with proper settings, model, form, view, and template.

<details>
<summary>Solution</summary>

```python
# settings.py
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

MEDIA_ROOT = BASE_DIR / 'media'
MEDIA_URL = '/media/'

# models.py
from django.db import models

class UploadedFile(models.Model):
    name = models.CharField(max_length=200)
    file = models.FileField(upload_to='uploads/')
    uploaded_at = models.DateTimeField(auto_now_add=True)

# forms.py
from django import forms
from .models import UploadedFile

class UploadForm(forms.ModelForm):
    class Meta:
        model = UploadedFile
        fields = ['name', 'file']

# views.py
from django.shortcuts import render, redirect
from .forms import UploadForm

def file_upload(request):
    if request.method == 'POST':
        form = UploadForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('upload_success')
    else:
        form = UploadForm()
    
    return render(request, 'upload.html', {'form': form})

# upload.html
<!DOCTYPE html>
<html>
<head>
    <title>Upload File</title>
</head>
<body>
    <h1>Upload File</h1>
    <form method="post" enctype="multipart/form-data">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Upload</button>
    </form>
</body>
</html>
```
</details>

## Next Steps

Now that you understand file upload fundamentals, continue to **[002-django-imagefield-and-serving-files.md](002-django-imagefield-and-serving-files.md)** to learn:
- ImageField for image-specific uploads
- Pillow library dependency and installation
- Image validation in browser and server
- MEDIA_URL configuration for serving files
- Using static() in urls.py to expose media files
- The .url property for template rendering
- The .path and .size properties for file information
- Best practices for image handling and serving
