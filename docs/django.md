# Django

Django is a high-level Python web framework that encourages rapid development and clean design.

## Installation

```bash
pip install django

# Check version
python -m django --version
```

## Create Project

```bash
# Create new project
django-admin startproject myproject
cd myproject

# Create app
python manage.py startapp myapp

# Run development server
python manage.py runserver
```

## Project Structure

```
myproject/
├── manage.py
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── myapp/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    ├── views.py
    └── migrations/
```

## Settings

```python
# myproject/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # Add your app
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'USER': 'myuser',
        'PASSWORD': 'mypassword',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

## Models

```python
# myapp/models.py
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name

class Post(models.Model):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
    ]

    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    content = models.TextField()
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name='posts')
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']

    def __str__(self):
        return self.title
```

## Migrations

```bash
# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Show migrations
python manage.py showmigrations

# Rollback migration
python manage.py migrate myapp 0001
```

## Views

### Function-Based Views

```python
# myapp/views.py
from django.shortcuts import render, get_object_or_404
from django.http import JsonResponse
from .models import Post

def post_list(request):
    posts = Post.objects.filter(status='published')
    return render(request, 'post_list.html', {'posts': posts})

def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug)
    return render(request, 'post_detail.html', {'post': post})

def api_posts(request):
    posts = Post.objects.values('title', 'slug', 'created_at')
    return JsonResponse(list(posts), safe=False)
```

### Class-Based Views

```python
from django.views.generic import ListView, DetailView, CreateView
from django.urls import reverse_lazy
from .models import Post

class PostListView(ListView):
    model = Post
    template_name = 'post_list.html'
    context_object_name = 'posts'
    paginate_by = 10

    def get_queryset(self):
        return Post.objects.filter(status='published')

class PostDetailView(DetailView):
    model = Post
    template_name = 'post_detail.html'
    context_object_name = 'post'

class PostCreateView(CreateView):
    model = Post
    fields = ['title', 'content', 'status']
    template_name = 'post_form.html'
    success_url = reverse_lazy('post_list')
```

## URLs

```python
# myproject/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]

# myapp/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('post/<slug:slug>/', views.post_detail, name='post_detail'),
    path('api/posts/', views.api_posts, name='api_posts'),
]
```

## Templates

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>

<!-- templates/post_list.html -->
{% extends 'base.html' %}

{% block title %}Posts{% endblock %}

{% block content %}
<h1>Posts</h1>
{% for post in posts %}
    <article>
        <h2><a href="{% url 'post_detail' post.slug %}">{{ post.title }}</a></h2>
        <p>By {{ post.author.name }} on {{ post.created_at|date:"F d, Y" }}</p>
    </article>
{% empty %}
    <p>No posts available.</p>
{% endfor %}
{% endblock %}
```

## Forms

```python
# myapp/forms.py
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'status']
        widgets = {
            'content': forms.Textarea(attrs={'rows': 10}),
        }

    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 5:
            raise forms.ValidationError('Title must be at least 5 characters')
        return title

# Using in view
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('post_list')
    else:
        form = PostForm()

    return render(request, 'post_form.html', {'form': form})
```

## Admin

```python
# myapp/admin.py
from django.contrib import admin
from .models import Author, Post

@admin.register(Author)
class AuthorAdmin(admin.ModelAdmin):
    list_display = ['name', 'email', 'created_at']
    search_fields = ['name', 'email']

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'status', 'created_at']
    list_filter = ['status', 'created_at']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'created_at'
```

## QuerySets

```python
# Get all
posts = Post.objects.all()

# Filter
published = Post.objects.filter(status='published')
recent = Post.objects.filter(created_at__gte='2024-01-01')

# Exclude
drafts = Post.objects.exclude(status='published')

# Get single object
post = Post.objects.get(id=1)
post = Post.objects.get(slug='my-post')

# Order
posts = Post.objects.order_by('-created_at')
posts = Post.objects.order_by('author__name')

# Limit
posts = Post.objects.all()[:10]

# Count
count = Post.objects.filter(status='published').count()

# Exists
has_posts = Post.objects.filter(author_id=1).exists()

# Aggregate
from django.db.models import Count, Avg
stats = Post.objects.aggregate(
    total=Count('id'),
    avg_length=Avg('content')
)

# Annotate
authors = Author.objects.annotate(post_count=Count('posts'))
```

## Authentication

```python
from django.contrib.auth.decorators import login_required
from django.contrib.auth import authenticate, login, logout

@login_required
def protected_view(request):
    return render(request, 'protected.html')

def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)

        if user is not None:
            login(request, user)
            return redirect('home')

    return render(request, 'login.html')

def logout_view(request):
    logout(request)
    return redirect('home')
```

## Middleware

```python
# myapp/middleware.py
class CustomMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # Code before view
        print(f"Request: {request.path}")

        response = self.get_response(request)

        # Code after view
        print(f"Response: {response.status_code}")

        return response

# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'myapp.middleware.CustomMiddleware',  # Add custom middleware
    # ...
]
```

## Static Files

```python
# settings.py
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]

# In templates
{% load static %}
<link rel="stylesheet" href="{% static 'css/style.css' %}">
<img src="{% static 'img/logo.png' %}" alt="Logo">
```

```bash
# Collect static files for production
python manage.py collectstatic
```

## Django REST Framework

```python
# Install
pip install djangorestframework

# settings.py
INSTALLED_APPS = [
    # ...
    'rest_framework',
]

# serializers.py
from rest_framework import serializers
from .models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['id', 'title', 'slug', 'content', 'status', 'created_at']

# views.py
from rest_framework import viewsets
from .serializers import PostSerializer

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer

# urls.py
from rest_framework.routers import DefaultRouter
from .views import PostViewSet

router = DefaultRouter()
router.register(r'posts', PostViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]
```

## Management Commands

```bash
# Create superuser
python manage.py createsuperuser

# Shell
python manage.py shell

# Database shell
python manage.py dbshell

# Check project
python manage.py check

# Clear sessions
python manage.py clearsessions
```

## Resources

- [Official Documentation](https://docs.djangoproject.com/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Awesome Django](https://github.com/wsvincent/awesome-django)
