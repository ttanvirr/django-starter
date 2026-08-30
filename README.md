# Table of contents <!-- omit in toc -->

- [1. Run the existing starter project](#1-run-the-existing-starter-project)
- [2. Step by step guide from scratch (for Ubuntu or wsl)](#2-step-by-step-guide-from-scratch-for-ubuntu-or-wsl)
  - [2.1. Install python (if still not)](#21-install-python-if-still-not)
  - [2.2. Create venv (virtual environment)](#22-create-venv-virtual-environment)
  - [2.3. Add common things to `.gitignore` file](#23-add-common-things-to-gitignore-file)
  - [2.4. Install Django](#24-install-django)
  - [2.5. Create a django project](#25-create-a-django-project)
  - [2.6. Database setup for PostgreSQL](#26-database-setup-for-postgresql)
    - [2.6.1. Install `psycopg` package (https://github.com/psycopg/psycopg/)](#261-install-psycopg-package-httpsgithubcompsycopgpsycopg)
    - [2.6.2. Create database](#262-create-database)
      - [2.6.2.1. enter postgress psql shell](#2621-enter-postgress-psql-shell)
    - [2.6.3. Settings file](#263-settings-file)
    - [2.6.4. Environ variables](#264-environ-variables)
    - [2.6.5. Git commit](#265-git-commit)
  - [2.7. Configure templates, static files and media](#27-configure-templates-static-files-and-media)
  - [2.8. (Optional) TailwindCSS and DaisyUI setup for frontend](#28-optional-tailwindcss-and-daisyui-setup-for-frontend)

# 1. Run the existing starter project

- Create a venv and activate it

  ```bash
  python -m venv .venv
  source .venv/bin/activate
  ```

- Install dependencies inside the venv

  ```bash
  pip install -r requirements.txt
  ```

- create a db ([follow this steps](#database-setup-for-postgresql))

- Run `migrate`

  ```bash
  python manage.py migration
  ```

- Run `runserver`

  ```bash
  python manage.py runserver
  ```

- Visit `http://127.0.0.1:8000/`

# 2. Step by step guide from scratch (for Ubuntu or wsl)

Create your own project directory to start from scratch and follow this guide.

## 2.1. Install python (if still not)

- for ubuntu (wsl)

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

- OR, check if they are already installed (globally)

```bash
apt list --installed python3 python3-pip python3-venv
```

## 2.2. Create venv (virtual environment)

- Create a venv inside the project directory

  ```bash
  python3 -m venv .venv
  ```

- immediately add it to `.gitignore` file:

  ```
  .venv/
  ```

- Activate venv

  ```bash
  source .venv/bin/activate
  ```

## 2.3. Add common things to `.gitignore` file

[Follow this link](https://github.com/ttanvirr/django-notes/blob/main/.gitignore) and add common things to `.gitignore`.

## 2.4. Install Django

```bash
python -m pip install django
pip freeze > requirements.txt
```

- To verify that Django can be seen by Python, type python from your shell. Then at the Python prompt, try
  to import Django:

```bash
python
>>> import django
>>> print(django.get_version())

//output
6.0.6

>>> exit()
```

- Or, run this while activating the venv

```bash
python -m django --version
```

## 2.5. Create a django project

```bash
django-admin startproject config .
```

- This will create a core project directory named 'config' and a file 'manage.py' inside current directory

- Run server to test if everything is okay

```bash
python manage.py runserver
```

- ignore the 'unapplied migration' warning for now.

## 2.6. Database setup for PostgreSQL

- Django comes with `sqlite` db by defalut. But if we want to setup big db engines like PostgreSql, we need to set it up.
- This can be done at the end, but recommended to do at the beginning to avoid any issue

### 2.6.1. Install `psycopg` package (https://github.com/psycopg/psycopg/)

- In the project's venv (activating venv), install following

```bash
pip install "psycopg[binary,pool]"
pip freeze > requirements.txt
```

### 2.6.2. Create database

#### 2.6.2.1. enter postgress psql shell

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

### 2.6.3. Settings file

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

### 2.6.4. Environ variables

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

### 2.6.5. Git commit

- As initial setups have completed, do your first commit (optionally push to a github repo)

## 2.7. Configure templates, static files and media

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

## 2.8. (Optional) TailwindCSS and DaisyUI setup for frontend

- will be updated later

<!-- ============================END INITIAL DJANGO SETUPS============================== -->

**NOW YOU ARE READY TO CREATE PROJECT-SPECIFIC APPS AND DO OTHER SETUPS**
