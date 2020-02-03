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
