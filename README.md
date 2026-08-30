# Table of contents <!-- omit in toc -->

- [1. Run the existing starter project](#1-run-the-existing-starter-project)
- [2. Step by step guide from scratch (for Ubuntu or wsl)](#2-step-by-step-guide-from-scratch-for-ubuntu-or-wsl)
  - [2.1. Create the Django project using `uv`](#21-create-the-django-project-using-uv)
  - [2.2. Database setup for PostgreSQL](#22-database-setup-for-postgresql)
    - [2.2.1. Install `psycopg` package (https://github.com/psycopg/psycopg/)](#221-install-psycopg-package-httpsgithubcompsycopgpsycopg)
    - [2.2.2. Create database](#222-create-database)
      - [2.2.2.1. enter postgress psql shell](#2221-enter-postgress-psql-shell)
    - [2.2.3. Settings file](#223-settings-file)
    - [2.2.4. Environ variables](#224-environ-variables)
    - [2.2.5. Git commit](#225-git-commit)
  - [2.3. Configure templates, static files and media](#23-configure-templates-static-files-and-media)
  - [2.4. (Optional) TailwindCSS and DaisyUI setup for frontend](#24-optional-tailwindcss-and-daisyui-setup-for-frontend)

# 1. Run the existing starter project

- Install [uv](https://tinyurl.com/5bdmvhn4) globaly if not already installed
- Install dependencies in the venv

  `terminal`

  ```bash
  uv sync
  ```

- Create a db ([follow this steps](#database-setup-for-postgresql))

- Run `migrate`

  `terminal`

  ```bash
  uv run manage.py migrate
  ```

# 2. Step by step guide from scratch (for Ubuntu or wsl)

Create your own project directory to start from scratch and follow this guide.

## 2.1. Create the Django project using `uv`

1. Install [uv](https://tinyurl.com/5bdmvhn4) globaly if not already installed.
2. Create a project directory (e.g., `django-starter`) and navigate to it.
3. Initialize the project in the current directory, pinned to `Python 3.15`:

   ```bash
   uv init --python 3.15 .
   ```

   > [!TIP]
   >
   > You can check available python versions using `uv python list`

4. Add Django to the project venv, then scaffold the Django project:

   ```bash
   uv add django
   uv run django-admin startproject config .
   ```

   To verify that Django can be seen by Python, try this:

   ```bash
   python -c "import django; print(django.get_version())"
   ```

   output:

   ```
   6.1
   ```

   - Or, run this using `uv`:

   ```bash
   uv run python -m django --version
   ```

5. Create a `.gitignore` file.

[Follow this link](https://github.com/ttanvirr/django-notes/blob/main/.gitignore) and add common things to `.gitignore`.

Your directory should now contain the following files:

```
├── .python-version
├── src/
│ └── django_starter/
│   └── __init__.py
├── manage.py
├── config/
│ ├── __init__.py
│ ├── asgi.py
│ ├── settings.py
│ ├── urls.py
│ └── wsgi.py
├── pyproject.toml
├── .gitignore
├── uv.lock
└── README.md
```

## 2.2. Database setup for PostgreSQL

- Django comes with `sqlite` db by defalut. But if we want to setup big db engines like PostgreSql, we need to set it up.
- This can be done at the end, but recommended to do at the beginning to avoid any issue

### 2.2.1. Install `psycopg` package (https://github.com/psycopg/psycopg/)

- In the project's venv (activating venv), install following

```bash
pip install "psycopg[binary,pool]"
pip freeze > requirements.txt
```

### 2.2.2. Create database

#### 2.2.2.1. enter postgress psql shell

open wsl

```bash
sudo -u postgres psql
```

(Enter password for `sudo`)

- Create db for an existing postgres user:

`(DON'T FORGET SEMICOLON FOR POSTGRES SHELL COMMANDS)`

```psql
CREATE DATABASE <db_name> OWNER <pg_username>;
```

This will grant all privileges to the user by default

- check if the db is created

```psql
\l
```

### 2.2.3. Settings file

- Install `dj-database-url` for convenience (https://pypi.org/project/dj-database-url/)

**terminal**

```bash
(.venv)$ pip install dj-database-url
pip freeze > requirements.txt
```

**config/settings.py**

```py
import dj_database_url

# modify
DATABASES = {
    "default": dj_database_url.config(
        default="postgres://<db_owner>:<owner_password>@<host>:5432/<db_name>",
        conn_max_age=600,
    )
}
```

- for local dev `<host>` will be `localhost`

- DELETE `db.sqlite3` file
- Run migrate and runserver. See if everything is okay

- For extra check, check if database tables (auth, etc.) are created.

**terminal**

```bash
psql -U <db_user> -d <db_name>
```

**psql shell**

```psql
\dt
```

This tables are created based on INSTALLED_APPS listed in settings.py

- run `\q` to exit psql shell

### 2.2.4. Environ variables

- create `.env` file in the root

```
DEBUG=True
SECRET_KEY=<django_secret_key>
DATABASE_URL=postgres://<db_owner>:<owner_password>@<port>:5432/<db_name>
```

- immediately add .env to .gitignore file

- Generate secret key for django:

**terminal**

```bash
python -c "import secrets; print(secrets.token_urlsafe(50))"
```

- add the secret key to .env file as value of `SECRET_KEY`.
- update `DATABASE_URL` in .env file with real values.

- Install `django-environ` in the venv:

```bash
pip install django-environ
pip freeze > requirements.txt
```

- Modify settings.py to use the envs

**settings.py**

```py
import environ
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# Initialize environ
env = environ.Env(
    # set casting, default value
    DEBUG=(bool, False)
)

# Read the .env file
environ.Env.read_env(os.path.join(BASE_DIR, '.env'))

# Use the variables
SECRET_KEY = env('SECRET_KEY')
DEBUG = env('DEBUG')

DATABASES = {
    'default': env.db(), # Parses DATABASE_URL using django-environ's built-in dj-database-url support.
}
DATABASES["default"]["CONN_MAX_AGE"] = 600
```

- Stop the server, run migrate and server. Check if everything is okay

### 2.2.5. Git commit

- As initial setups have completed, do your first commit (optionally push to a github repo)

## 2.3. Configure templates, static files and media

If you want to use project-level `templates` directory, add this settings in `settings.py`

```py
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [BASE_DIR / "templates"], # new
        "APP_DIRS": True,
        # ...
    },
]
```

If you want your static files to be served by nginx (usually in production), add these settings to `settings.py`

```py
STATIC_URL = "static/"
STATIC_ROOT = BASE_DIR / "staticfiles"  # new
```

> [!NOTE]
> The command `python manage.py collectstatic` will collect all static files to this `staticfiles` directory. Run this command only when you want nginx to serve static files (usually in production).

If your project needs user to upload files/media, add these settings to `settings.py`

```py
MEDIA_URL = "media/"
MEDIA_ROOT = BASE_DIR / "media"
```

With default configuration, the development server (`runserver`) automatically serves static files during development but it can't automatically serve media files. So, we need to add the following URL pattern for development.:

`config/urls.py`

```py
from django.conf import settings # new
from django.conf.urls.static import static # new

urlpatterns = [
    # ...
]

# new
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

```

If your project handles image uploads, install `Pillow`:

```bash
pip install pillow
pip freeze > requirements.txt
```

## 2.4. (Optional) TailwindCSS and DaisyUI setup for frontend

- will be updated later

<!-- ============================END INITIAL DJANGO SETUPS============================== -->

**NOW YOU ARE READY TO CREATE PROJECT-SPECIFIC APPS AND DO OTHER SETUPS**
