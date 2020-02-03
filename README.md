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
