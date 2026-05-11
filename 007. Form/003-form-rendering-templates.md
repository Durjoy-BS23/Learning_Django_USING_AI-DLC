# Form Rendering and Templates

## Introduction

Rendering forms in templates is where your Django forms become visible to users. Django provides multiple ways to render forms, from simple one-line rendering to complete manual control over every field. This guide covers form rendering techniques, CSRF protection, manual field rendering, error message display, and looping over form fields for dynamic layouts. Understanding these techniques will help you create beautiful, accessible, and user-friendly form interfaces.

## Concept Explanation

### Automatic Form Rendering

Django provides three built-in methods for automatic form rendering:

**1. `{{ form.as_p }}`**
Renders each field as a paragraph (`<p>`) tag:
```html
<p><label for="id_title">Title:</label> <input type="text" name="title" id="id_title"></p>
<p><label for="id_content">Content:</label> <textarea name="content" id="id_content"></textarea></p>
```

**2. `{{ form.as_table }}`**
Renders fields as table rows (`<tr>`):
```html
<tr><th><label for="id_title">Title:</label></th><td><input type="text" name="title" id="id_title"></td></tr>
<tr><th><label for="id_content">Content:</label></th><td><textarea name="content" id="id_content"></textarea></td></tr>
```

**3. `{{ form.as_ul }}`**
Renders fields as list items (`<li>`):
```html
<li><label for="id_title">Title:</label> <input type="text" name="title" id="id_title"></li>
<li><label for="id_content">Content:</label> <textarea name="content" id="id_content"></textarea></li>
```

**Note**: None of these methods include the surrounding `<form>` tags or submit button - you must add those yourself.

### CSRF Protection

Cross-Site Request Forgery (CSRF) is a security vulnerability where attackers trick users into submitting forms they didn't intend. Django provides built-in CSRF protection through a CSRF token.

**How CSRF Protection Works:**
1. Django generates a unique token for each user session
2. This token is included in forms using `{% csrf_token %}`
3. When the form is submitted, Django validates the token
4. If the token is missing or invalid, the request is rejected

**Implementation:**
```html
<form method="post">
    {% csrf_token %}  <!-- Include this in every POST form -->
    {{ form }}
    <button type="submit">Submit</button>
</form>
```

**Important**: CSRF protection is required for all POST forms. GET forms don't need CSRF tokens.

### Manual Field Rendering

For complete control over form layout, you can render each field manually:

**Benefits:**
- Custom field ordering
- Custom HTML structure
- Individual field styling
- Access to field attributes
- Fine-grained error control

**Basic Manual Rendering:**
```html
<form method="post">
    {% csrf_token %}
    
    <div class="form-field">
        <label for="{{ form.title.id_for_label }}">Title:</label>
        {{ form.title }}
        {{ form.title.errors }}
    </div>
    
    <div class="form-field">
        <label for="{{ form.content.id_for_label }}">Content:</label>
        {{ form.content }}
        {{ form.content.errors }}
    </div>
    
    <button type="submit">Submit</button>
</form>
```

### Error Message Display

Django automatically generates error messages when validation fails. You can display these errors in several ways:

**Automatic Error Display:**
When using `{{ form.as_p }}`, `{{ form.as_table }}`, or `{{ form.as_ul }}`, errors are automatically displayed above each field.

**Manual Error Display:**
```html
{{ form.title.errors }}
<!-- Renders: -->
<ul class="errorlist">
    <li>This field is required.</li>
</ul>
```

**Conditional Error Display:**
```html
{% if form.title.errors %}
    <div class="error">
        {{ form.title.errors }}
    </div>
{% endif %}
```

**Custom Error Styling:**
```html
{% if form.title.errors %}
    <div class="alert alert-danger">
        {% for error in form.title.errors %}
            <p>{{ error }}</p>
        {% endfor %}
    </div>
{% endif %}
```

### Looping Over Form Fields

When you have many fields with similar rendering requirements, looping over fields reduces code duplication:

**Basic Loop:**
```html
{% for field in form %}
    <div class="form-field">
        {{ field.label_tag }}
        {{ field }}
        {{ field.errors }}
        {% if field.help_text %}
            <p class="help-text">{{ field.help_text }}</p>
        {% endif %}
    </div>
{% endfor %}
```

**Field Attributes Available in Loops:**
- `field.errors` - Error messages for this field
- `field.label_tag` - Field label wrapped in `<label>` tag
- `field` - The field widget itself
- `field.help_text` - Help text for the field
- `field.id_for_label` - The ID for the label's `for` attribute
- `field.html_name` - The name attribute for the input
- `field.is_hidden` - Boolean indicating if field is hidden
- `field.value` - The current value of the field

### Hidden vs Visible Fields

Some forms contain hidden fields (e.g., CSRF tokens, IDs). You can handle them separately:

**Separate Hidden and Visible Fields:**
```html
<form method="post">
    {% csrf_token %}
    
    {# Render hidden fields #}
    {% for hidden in form.hidden_fields %}
        {{ hidden }}
    {% endfor %}
    
    {# Render visible fields #}
    {% for field in form.visible_fields %}
        <div class="form-field">
            {{ field.label_tag }}
            {{ field }}
            {{ field.errors }}
        </div>
    {% endfor %}
    
    <button type="submit">Submit</button>
</form>
```

**Check if Field is Hidden:**
```html
{% for field in form %}
    {% if not field.is_hidden %}
        <div class="form-field">
            {{ field.label_tag }}
            {{ field }}
        </div>
    {% else %}
        {{ field }}
    {% endif %}
{% endfor %}
```

### Non-Field Errors

Some validation errors apply to the entire form, not specific fields (e.g., cross-field validation):

**Display Non-Field Errors:**
```html
{% if form.non_field_errors %}
    <div class="alert alert-danger">
        {% for error in form.non_field_errors %}
            <p>{{ error }}</p>
        {% endfor %}
    </div>
{% endif %}
```

## Code Examples

### Basic Template with Automatic Rendering

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
            padding: 0;
        }
        .helptext {
            font-size: 0.85em;
            color: #666;
            margin-top: 5px;
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

### Manual Field Rendering with Custom Layout

```html
<!-- templates/create_post_manual.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
    <style>
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            font-weight: bold;
            margin-bottom: 5px;
        }
        .errorlist {
            color: red;
            list-style-type: none;
            padding: 0;
            margin: 5px 0;
        }
        .help-text {
            font-size: 0.85em;
            color: #666;
            margin-top: 5px;
        }
        textarea {
            width: 100%;
            min-height: 200px;
        }
    </style>
</head>
<body>
    <h1>Create a New Post</h1>
    
    {% if form.non_field_errors %}
        <div class="alert alert-danger">
            {% for error in form.non_field_errors %}
                <p>{{ error }}</p>
            {% endfor %}
        </div>
    {% endif %}
    
    <form method="post">
        {% csrf_token %}
        
        <div class="form-group">
            <label for="{{ form.title.id_for_label }}">Post Title:</label>
            {{ form.title }}
            {{ form.title.errors }}
            {% if form.title.help_text %}
                <p class="help-text">{{ form.title.help_text }}</p>
            {% endif %}
        </div>
        
        <div class="form-group">
            <label for="{{ form.content.id_for_label }}">Post Content:</label>
            {{ form.content }}
            {{ form.content.errors }}
            {% if form.content.help_text %}
                <p class="help-text">{{ form.content.help_text }}</p>
            {% endif %}
        </div>
        
        <div class="form-group">
            <label for="{{ form.is_published.id_for_label }}">
                {{ form.is_published.label }}
            </label>
            {{ form.is_published }}
            {{ form.is_published.errors }}
        </div>
        
        <button type="submit">Create Post</button>
    </form>
</body>
</html>
```

### Looping Over Fields with Custom Styling

```html
<!-- templates/create_post_loop.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
    <style>
        .form-field {
            margin-bottom: 20px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 5px;
        }
        .form-field label {
            display: block;
            font-weight: bold;
            margin-bottom: 10px;
        }
        .form-field input[type="text"],
        .form-field textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        .form-field textarea {
            min-height: 150px;
        }
        .errorlist {
            color: #d9534f;
            list-style-type: none;
            padding: 0;
            margin: 10px 0;
        }
        .errorlist li {
            background: #f8d7da;
            padding: 10px;
            border-radius: 4px;
        }
        .help-text {
            font-size: 0.85em;
            color: #777;
            margin-top: 5px;
            font-style: italic;
        }
    </style>
</head>
<body>
    <h1>Create a New Post</h1>
    
    {% if form.non_field_errors %}
        <div class="alert alert-danger">
            <h3>Form Errors:</h3>
            {% for error in form.non_field_errors %}
                <p>{{ error }}</p>
            {% endfor %}
        </div>
    {% endif %}
    
    <form method="post">
        {% csrf_token %}
        
        {% for field in form %}
            <div class="form-field">
                {{ field.label_tag }}
                {{ field }}
                
                {% if field.errors %}
                    <ul class="errorlist">
                        {% for error in field.errors %}
                            <li>{{ error }}</li>
                        {% endfor %}
                    </ul>
                {% endif %}
                
                {% if field.help_text %}
                    <p class="help-text">{{ field.help_text }}</p>
                {% endif %}
            </div>
        {% endfor %}
        
        <button type="submit">Create Post</button>
    </form>
</body>
</html>
```

### Bootstrap-Styled Form

```html
<!-- templates/create_post_bootstrap.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-5">
        <h1>Create a New Post</h1>
        
        {% if form.non_field_errors %}
            <div class="alert alert-danger">
                {% for error in form.non_field_errors %}
                    {{ error }}
                {% endfor %}
            </div>
        {% endif %}
        
        <form method="post">
            {% csrf_token %}
            
            {% for field in form %}
                <div class="mb-3">
                    <label for="{{ field.id_for_label }}" class="form-label">
                        {{ field.label }}
                    </label>
                    
                    {% if field.field.widget.input_type == 'checkbox' %}
                        <div class="form-check">
                            <input class="form-check-input" 
                                   type="checkbox" 
                                   name="{{ field.html_name }}"
                                   id="{{ field.id_for_label }}"
                                   {% if field.value %}checked{% endif %}>
                            <label class="form-check-label" for="{{ field.id_for_label }}">
                                {{ field.label }}
                            </label>
                        </div>
                    {% else %}
                        {{ field }}
                    {% endif %}
                    
                    {% if field.errors %}
                        <div class="invalid-feedback d-block">
                            {% for error in field.errors %}
                                {{ error }}
                            {% endfor %}
                        </div>
                    {% endif %}
                    
                    {% if field.help_text %}
                        <div class="form-text">{{ field.help_text }}</div>
                    {% endif %}
                </div>
            {% endfor %}
            
            <button type="submit" class="btn btn-primary">Create Post</button>
        </form>
    </div>
</body>
</html>
```

### Form with Success/Error Messages

```html
<!-- templates/create_post_with_messages.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
    <style>
        .message {
            padding: 15px;
            margin-bottom: 20px;
            border-radius: 5px;
        }
        .message.success {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        .message.error {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
    </style>
</head>
<body>
    <h1>Create a New Post</h1>
    
    {% if messages %}
        {% for message in messages %}
            <div class="message {{ message.tags }}">
                {{ message }}
            </div>
        {% endfor %}
    {% endif %}
    
    <form method="post">
        {% csrf_token %}
        
        {% for field in form %}
            <div class="form-field">
                {{ field.label_tag }}
                {{ field }}
                {{ field.errors }}
                {% if field.help_text %}
                    <p class="help-text">{{ field.help_text }}</p>
                {% endif %}
            </div>
        {% endfor %}
        
        <button type="submit">Create Post</button>
    </form>
</body>
</html>
```

**View with Messages:**
```python
from django.contrib import messages
from django.shortcuts import render, redirect

def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            Post.objects.create(**form.cleaned_data)
            messages.success(request, 'Post created successfully!')
            return redirect('post_list')
        else:
            messages.error(request, 'Please correct the errors below.')
    else:
        form = PostForm()
    
    return render(request, 'create_post.html', {'form': form})
```

### Form with Hidden Fields

```html
<!-- templates/create_post_with_hidden.html -->
<form method="post">
    {% csrf_token %}
    
    {# Render all hidden fields first #}
    {% for hidden in form.hidden_fields %}
        {{ hidden }}
    {% endfor %}
    
    {# Then render visible fields #}
    {% for field in form.visible_fields %}
        <div class="form-field">
            {{ field.label_tag }}
            {{ field }}
            {{ field.errors }}
        </div>
    {% endfor %}
    
    <button type="submit">Submit</button>
</form>
```

## Key Takeaways

- Django provides `{{ form.as_p }}`, `{{ form.as_table }}`, and `{{ form.as_ul }}` for automatic rendering
- Always include `{% csrf_token %}` in POST forms for security
- Manual field rendering provides complete control over form layout
- Display errors with `{{ field.errors }}` or `{{ form.non_field_errors }}`
- Loop over fields with `{% for field in form %}` to reduce code duplication
- Hidden fields can be handled separately with `form.hidden_fields()` and `form.visible_fields()`
- Field attributes include `label_tag`, `id_for_label`, `help_text`, `html_name`, `is_hidden`, and `value`
- CSRF tokens are required for POST forms but not GET forms
- Use Django messages framework for success/error feedback
- Custom CSS and frameworks (Bootstrap) can enhance form appearance

## Additional Context & Best Practices

### Rendering Best Practices

**1. Choose the Right Rendering Method**
```html
<!-- ✅ Use automatic rendering for simple forms -->
{{ form.as_p }}

<!-- ✅ Use manual rendering for custom layouts -->
<div class="custom-field">
    {{ field.label_tag }}
    {{ field }}
</div>

<!-- ✅ Use looping for repetitive field styling -->
{% for field in form %}
    <div class="standard-field">
        {{ field.label_tag }}
        {{ field }}
    </div>
{% endfor %}
```

**2. Always Include CSRF Token**
```html
<!-- ✅ CORRECT - CSRF token in POST form -->
<form method="post">
    {% csrf_token %}
    {{ form }}
</form>

<!-- ❌ WRONG - Missing CSRF token -->
<form method="post">
    {{ form }}
</form>
```

**3. Display Errors Clearly**
```html
<!-- ✅ GOOD - Clear error display -->
{% if field.errors %}
    <div class="error-message">
        {% for error in field.errors %}
            <p>{{ error }}</p>
        {% endfor %}
    </div>
{% endif %}

<!-- ❌ BAD - No error display or unclear -->
{{ field.errors }}
```

### Security Best Practices

**1. CSRF Protection is Mandatory**
```html
<!-- ✅ CORRECT - Always include CSRF token -->
<form method="post">
    {% csrf_token %}
    {{ form }}
</form>

<!-- ❌ WRONG - No CSRF protection -->
<form method="post">
    {{ form }}
</form>
```

**2. Don't Disable CSRF Unless Necessary**
```python
# ❌ AVOID - Disabling CSRF protection
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def my_view(request):
    pass
```

**3. Validate on Server Side**
```html
<!-- Client-side validation is helpful but not sufficient -->
<!-- Always validate on the server side -->
```

### Accessibility Best Practices

**1. Use Proper Label Associations**
```html
<!-- ✅ GOOD - Proper label for attribute -->
<label for="{{ field.id_for_label }}">{{ field.label }}</label>
{{ field }}

<!-- ✅ GOOD - Use label_tag -->
{{ field.label_tag }}
{{ field }}
```

**2. Provide Error Messages**
```html
<!-- ✅ GOOD - Clear error messages -->
{% if field.errors %}
    <div role="alert">
        {{ field.errors }}
    </div>
{% endif %}
```

**3. Include Help Text**
```html
<!-- ✅ GOOD - Helpful guidance -->
{% if field.help_text %}
    <p class="help">{{ field.help_text }}</p>
{% endif %}
```

### Common Pitfalls

**1. Forgetting CSRF Token**
```html
<!-- ❌ WRONG - No CSRF token -->
<form method="post">
    {{ form }}
</form>

<!-- ✅ CORRECT - Include CSRF token -->
<form method="post">
    {% csrf_token %}
    {{ form }}
</form>
```

**2. Not Displaying Errors**
```html
<!-- ❌ WRONG - No error display -->
{{ field }}

<!-- ✅ CORRECT - Display errors -->
{{ field }}
{{ field.errors }}
```

**3. Missing Form Tags**
```html
<!-- ❌ WRONG - No form tags -->
{{ form.as_p }}
<button>Submit</button>

<!-- ✅ CORRECT - Include form tags -->
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button>Submit</button>
</form>
```

**4. Not Handling Hidden Fields**
```html
<!-- ❌ WRONG - Hidden fields might be missed -->
{% for field in form %}
    {{ field }}
{% endfor %}

<!-- ✅ CORRECT - Handle hidden fields separately -->
{% for hidden in form.hidden_fields %}
    {{ hidden }}
{% endfor %}
{% for field in form.visible_fields %}
    {{ field }}
{% endfor %}
```

### Performance Considerations

**1. Form Rendering Overhead**
Form rendering is lightweight. Don't worry about performance for typical forms.

**2. CSS Frameworks**
Using frameworks like Bootstrap adds download overhead but improves UX. Consider using CDN or minified versions.

**3. Template Inheritance**
Use template inheritance to avoid repeating form styling across multiple templates.

### Advanced Tips

**1. Custom Form Templates**
Create reusable form templates:
```html
<!-- templates/form_snippet.html -->
{% for field in form %}
    <div class="form-field">
        {{ field.label_tag }}
        {{ field }}
        {{ field.errors }}
        {% if field.help_text %}
            <p class="help">{{ field.help_text }}</p>
        {% endif %}
    </div>
{% endfor %}
```

Use it in other templates:
```html
{% include "form_snippet.html" %}
```

**2. Form Mixins**
Create form rendering mixins in views:
```python
class FormMixin:
    def get_form_context(self):
        return {'form': self.form}
```

**3. Dynamic Field Attributes**
Add attributes dynamically:
```python
# In form definition
title = forms.CharField(
    widget=forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Enter title'})
)
```

## Practice Exercises

### Exercise 1: Basic Form Rendering

Create a template that renders a PostForm using `{{ form.as_p }}` with:
- CSRF token
- Submit button
- Basic error styling

<details>
<summary>Solution</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
    <style>
        .errorlist {
            color: red;
            list-style-type: none;
        }
    </style>
</head>
<body>
    <h1>Create Post</h1>
    
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Create Post</button>
    </form>
</body>
</html>
```
</details>

### Exercise 2: Manual Field Rendering

Create a template that manually renders each field of PostForm with:
- Custom div wrappers
- Error display below each field
- Help text display

<details>
<summary>Solution</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
</head>
<body>
    <h1>Create Post</h1>
    
    <form method="post">
        {% csrf_token %}
        
        <div class="form-field">
            <label for="{{ form.title.id_for_label }}">Title:</label>
            {{ form.title }}
            {{ form.title.errors }}
            {% if form.title.help_text %}
                <p class="help">{{ form.title.help_text }}</p>
            {% endif %}
        </div>
        
        <div class="form-field">
            <label for="{{ form.content.id_for_label }}">Content:</label>
            {{ form.content }}
            {{ form.content.errors }}
            {% if form.content.help_text %}
                <p class="help">{{ form.content.help_text }}</p>
            {% endif %}
        </div>
        
        <div class="form-field">
            <label for="{{ form.is_published.id_for_label }}">Publish:</label>
            {{ form.is_published }}
            {{ form.is_published.errors }}
        </div>
        
        <button type="submit">Create Post</button>
    </form>
</body>
</html>
```
</details>

### Exercise 3: Loop Over Fields

Create a template that loops over all fields with:
- Consistent styling for all fields
- Conditional error display
- Help text when available

<details>
<summary>Solution</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
    <style>
        .form-field {
            margin-bottom: 15px;
        }
        .errorlist {
            color: red;
            list-style-type: none;
        }
        .help {
            font-size: 0.85em;
            color: #666;
        }
    </style>
</head>
<body>
    <h1>Create Post</h1>
    
    <form method="post">
        {% csrf_token %}
        
        {% for field in form %}
            <div class="form-field">
                {{ field.label_tag }}
                {{ field }}
                
                {% if field.errors %}
                    <ul class="errorlist">
                        {% for error in field.errors %}
                            <li>{{ error }}</li>
                        {% endfor %}
                    </ul>
                {% endif %}
                
                {% if field.help_text %}
                    <p class="help">{{ field.help_text }}</p>
                {% endif %}
            </div>
        {% endfor %}
        
        <button type="submit">Create Post</button>
    </form>
</body>
</html>
```
</details>

### Exercise 4: Handle Hidden Fields

Create a template that:
- Renders hidden fields separately
- Renders visible fields with styling
- Handles non-field errors

<details>
<summary>Solution</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
</head>
<body>
    <h1>Create Post</h1>
    
    {% if form.non_field_errors %}
        <div class="error">
            {% for error in form.non_field_errors %}
                <p>{{ error }}</p>
            {% endfor %}
        </div>
    {% endif %}
    
    <form method="post">
        {% csrf_token %}
        
        {# Hidden fields #}
        {% for hidden in form.hidden_fields %}
            {{ hidden }}
        {% endfor %}
        
        {# Visible fields #}
        {% for field in form.visible_fields %}
            <div class="form-field">
                {{ field.label_tag }}
                {{ field }}
                {{ field.errors }}
            </div>
        {% endfor %}
        
        <button type="submit">Submit</button>
    </form>
</body>
</html>
```
</details>

### Exercise 5: Bootstrap Form

Create a Bootstrap-styled form using Bootstrap 5 classes for:
- Form groups
- Input styling
- Error messages
- Help text

<details>
<summary>Solution</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <title>Create Post</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-5">
        <h1>Create Post</h1>
        
        {% if form.non_field_errors %}
            <div class="alert alert-danger">
                {% for error in form.non_field_errors %}
                    {{ error }}
                {% endfor %}
            </div>
        {% endif %}
        
        <form method="post">
            {% csrf_token %}
            
            {% for field in form %}
                <div class="mb-3">
                    <label for="{{ field.id_for_label }}" class="form-label">
                        {{ field.label }}
                    </label>
                    
                    {{ field }}
                    
                    {% if field.errors %}
                        <div class="invalid-feedback d-block">
                            {% for error in field.errors %}
                                {{ error }}
                            {% endfor %}
                        </div>
                    {% endif %}
                    
                    {% if field.help_text %}
                        <div class="form-text">{{ field.help_text }}</div>
                    {% endif %}
                </div>
            {% endfor %}
            
            <button type="submit" class="btn btn-primary">Create Post</button>
        </form>
    </div>
</body>
</html>
```
</details>

## Next Steps

Now that you understand how to render forms in templates and handle errors, the final step is to learn about ModelForm, which automatically creates forms from your Django models.

Continue to **[004-modelform.md](004-modelform.md)** to learn:
- What ModelForm is and when to use it
- How to create ModelForm classes from models
- ModelForm view patterns for creating and updating data
- Saving ModelForm data to the database
- Customizing ModelForm fields and behavior
- ModelForm vs regular Form comparison
