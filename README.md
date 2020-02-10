# Django学習

## セットアップ

### 仮想環境

- 仮想環境の用意

```bash
python3 -m venv myvenv
```

- 仮想環境のアクティベート

```bash
. myvenv/bin/activate
```

### Djangoのインストール

- requirementsファイルによるパッケージのインストール

```txt:requirements.txt
Django~=3.0.2
```

```bash
pip install -r requirements.txt
```

## プロジェクトの作成

```bash
django-admin startproject mysite .
```

```txt
├── manage.py
├── mysite
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── requirements.txt
```

### 設定変更

- mysite/settings.py

```python:mysite/settings.py

# タイムゾーン
TIME_ZONE = 'Asia/Tokyo'

# 言語
LANGUAGE_CODE = 'ja'

# 静的ファイルのパス
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

# DBの設定（デフォルトのまま）
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

```

### DBの作成

```bash
python manage.py migrate
```

### Webサーバの起動

```bash
python manage.py runserver
```

## アプリケーションの作成

- 例としてblogアプリケーションを作成

```bash
python manage.py startapp blog
```

```txt
├── blog
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
├── mysite
│   └── ...
└── requirements.txt
```

- プロジェクトの設定変更（mysite/settings.py）

```python:mysite/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog.apps.BlogConfig', # 追加
]
```

### モデルの作成

- blog/models.py
  - models.ModelがDjangoのモデルだという意味でDjangoがDBに保存すべきものと認識する
  - フィールドのタイプ <https://docs.djangoproject.com/ja/2.2/ref/models/fields/#field-types>

```python:blog/models.py
from django.conf import settings
from django.db import models
from django.utils import timezone

class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    text = models.TextField()
    create_date = models.DateTimeField(default=timezone.now)
    publish_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.publish_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title
```

### テーブルの作成

```bash
python manage.py makemigrations blog
```

### 管理画面

- blog/admin.py

```python:blog/admin.py
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

- super userの作成

```bash
python manage.py createsuperuser
```

### viewの作成

- blog/views.py

```python:blog/views.py
from django.shortcuts import render
from django.utils import timezone
from .models import Post

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})
```

- テンプレートの作成(blog/templates/blog/post_list.html)

```html:blog/templates/blog/post_list.html
<html>
    <head>
        <title>Django blog</title>
    </head>
    <body>
        <header>
            <h1><a href="/">Django blog</a></h1>
        </header>
{% for post in posts %}
        <section>
            <p>published: {{ post.published_date }}</p>
            <h2><a href="">{{ post.title }}</a></h2>
            <p>{{ post.text|linebreaksbr }}</p>
        </section>
{% endfor %}
    </body>
</html>
```

### URLの設定

- アプリケーションのURLを定義(blog/urls.py)

```python:blog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.post_list, name='post_list'),
]
```

- アプリケーションのURLをインポート(mysite/urls.py)

```python:mysite/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')), # 追加
]
```

### ORMとクエリセット

- 参考 <https://docs.djangoproject.com/ja/2.2/ref/models/querysets/>

- Django shellの起動

```bash
python manage.py shell
```

- オブジェクトの取得(全て)

```python
from blog.models import Post

Post.objects.all()
```

- オブジェクトの作成

```python
Post.create(author=User.objects.get(username='abea'), title='Sample Title', text='Sample Text')
```

- オブジェクトの絞り込み

```python
Post.objects.filter(author=User.objects.get(username='abea'))
Post.objects.filter(title__contains='title')

from django.utils import timezone
Post.objects.filter(published_date__lte=timezone.now())
Post.objects.filter(published_date__lte=timezone.now()).order_by('created_date')  # 昇順
Post.objects.filter(published_date__lte=timezone.now()).order_by('-created_date') # 降順
```

### 静的ファイル

- 静的ファイルの配置

```txt
└── blog
    └── static
        └── css
            └── blog.css
```

- 静的ファイルのロード

```html
{% load static %}
<html>
    <head>
        <link rel="stylesheet" href="{% static 'css/blog.css' %}">
    </head>
```

### テンプレートの拡張

- 構成

```txt
└── blog
    └── templates
        └── blog
            ├── base.html
            └── post_list.html
```

- templates/blog/base.html

```html:templates/blog/base.html
{% load static %}
<html>
    <head>
        <title>Django blog</title>
        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css">
        <link href="//fonts.googleapis.com/css?family=Lobster&subset=latin,latin-ext" rel="stylesheet" type="text/css">
        <link rel="stylesheet" href="{% static 'css/blog.css' %}">
    </head>
    <body>
        <header>
            <h1><a href="/">Django blog</a></h1>
        </header>
        <div class="content container">
            <div class="row">
                <div class="col-md-8">
{% block content %}
{% endblock %}
                </div>
            </div>
        </div>
    </body>
</html>
```

- templates/blog/post_list.html

```html:templates/blog/post_list.html
{% extends 'blog/base.html' %}

{% block content %}
    {% for post in posts %}
                    <section class="post">
                        <div class="date">
                            <p>published: {{ post.published_date }}</p>
                        </div>
                        <h2><a href="">{{ post.title }}</a></h2>
                        <p>{{ post.text|linebreaksbr }}</p>
                    </section>
    {% endfor %}
{% endblock %}
```

#### 詳細ページを作る

- テンプレートへのリンク
  - プライマリキーを指定しなかった場合は `xxx.pk` という形式で取得可能(Djangoが自動で生成)

```html:templates/blog/post_list.html
                        <h2><a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a></h2>
```

- URL追加(blog/urls.py)

```python:blog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('post/<int:pk>/', views.post_detail name='post_detail'), # 追加
]
```

- viewの追加(blog/views.py)
  - get_object_or_404 メソッドを利用してオブジェクトが存在しない場合は404ページに遷移するようにしている

```python:blog/views.py
from django.shortcuts import render, get_object_or_404
from django.utils import timezone
from .models import Post

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})

# 追加
def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'blog/post_detail.html', {'post': post})
```

- テンプレートの追加(blog/templates/blog/post_detail.html)

```html:blog/templates/blog/post_detail.html
{% extends 'blog/base.html' %}

{% block content %}
                    <section class="post">
    {% if post.published_date %}
                        <div class="date">
                            <p>published: {{ post.published_date }}</p>
                        </div>
    {% endif %}
                        <h2>{{ post.title }}</h2>
                        <p>{{ post.text|linebreaksbr }}</p>
                    </section>
{% endblock %}
```

### フォーム

- 構成

```txt
└── blog
    ├── forms.py
    ├── urls.py
    └── views.py
```

- blog/forms.py

```python:blog/forms.py
from django import forms
from .models import Post

class PostForm(forms.ModelForm):

    class Meta:
        model = Post
        fields = ('title', 'text',)
```

- blog/views.py

```python:blog/views.py
from django.shortcuts import render, get_object_or_404, redirect

def post_new(request):

    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()

            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()

    return render(request, 'blog/post_edit.html', {'form': form})
```

- blog/templates/blog/post_edit.html

```html:post_edit.html
{% extends 'blog/base.html' %}

{% block content %}
    <h2>New Post</h2>
    <form method="POST" class="post-form">{% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="save btn btn-default">Save</button>
    </form>
{% endblock %}
```

- blog/urls.py

```python:blog/urls.py
urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('post/new/', views.post_new, name='post_new'), # 追加
]
```

- blog/templates/blog/base.html

```html:blog/templates/blog/base.html
            <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
```