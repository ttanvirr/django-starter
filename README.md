# Table of contents <!-- omit in toc -->

- [1. Overview: Django starter](#1-overview-django-starter)
- [2. Run the existing starter project](#2-run-the-existing-starter-project)
- [3. Step by step guide from scratch (for Ubuntu or wsl)](#3-step-by-step-guide-from-scratch-for-ubuntu-or-wsl)
  - [3.1. Create the Django project using `uv`](#31-create-the-django-project-using-uv)
  - [3.2. Database setup for PostgreSQL](#32-database-setup-for-postgresql)
    - [3.2.1. Install `psycopg` package (https://github.com/psycopg/psycopg/)](#321-install-psycopg-package-httpsgithubcompsycopgpsycopg)
    - [3.2.2. Create the PostgreSQL database](#322-create-the-postgresql-database)
    - [3.2.3. Configure PostgreSQL and environment variables](#323-configure-postgresql-and-environment-variables)
    - [3.2.4. Configure `settings.py`](#324-configure-settingspy)
    - [3.2.5. Verify the PostgreSQL configuration](#325-verify-the-postgresql-configuration)
    - [3.2.6. Git commit](#326-git-commit)
  - [3.3. Configure templates, static files and media](#33-configure-templates-static-files-and-media)
  - [3.4. (Optional) TailwindCSS and DaisyUI setup for frontend](#34-optional-tailwindcss-and-daisyui-setup-for-frontend)

# 1. Overview: Django starter

This repository is about how to initialize any Django project from scratch.

This repository has 2 versions: 1. using `uv` and 2. using typical `pip`.
The `main` branch and the `uv` branch uses `uv` and the `pip` branch uses `pip`.

# 2. Run the existing starter project

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

# 3. Step by step guide from scratch (for Ubuntu or wsl)

Create your own project directory to start from scratch and follow this guide.

## 3.1. Create the Django project using `uv`

1. Install [uv](https://tinyurl.com/5bdmvhn4) globaly if not already installed.
2. Create a project directory (e.g., `django-starter`) and navigate to it.
3. Initialize the project in the current directory, pinned to `Python 3.14`:

   ```bash
   uv init --python 3.14 .
   ```

   > [!TIP]
   >
   > You can check available python versions using `uv python list`
   >
   > An important consideration is that `psycopg` adapter has currently supports for python `3.10` to `3.14` ([see current supports](https://tinyurl.com/j732cduk)).

4. Add Django to the project venv, then scaffold the Django project:

   ```bash
   uv add django
   uv run django-admin startproject config .
   ```

   This will create a core project directory named 'config' and a file 'manage.py' inside current directory

   To verify that Django can be seen by Python, try this:

   ```bash
   uv run python -m django --version
   ```

5. Create a `.gitignore` file.

[Follow this link](https://github.com/ttanvirr/django-notes/blob/main/.gitignore) and add common things to `.gitignore`.

6. Run the development server to verify that everything is working:

   ```bash
   uv run manage.py runserver
   ```

   Ignore the unapplied migration warning for now.

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
├── db.sqlite3
├── .gitignore
├── pyproject.toml
├── uv.lock
└── README.md
```

## 3.2. Database setup for PostgreSQL

- Django comes with `sqlite` db by defalut. But if we want to setup big db engines like PostgreSql, we need to set it up.
- This can be done at the end, but recommended to do at the beginning to avoid any issue

### 3.2.1. Install `psycopg` package (https://github.com/psycopg/psycopg/)

`psycopg` is the PostgreSQL adapter that allows Python/Django to communicate with PostgreSQL.

Add it to the project using `uv`:

```bash
uv add "psycopg[binary,pool]"
```

### 3.2.2. Create the PostgreSQL database

Open wsl and run the following command to enter postgress `psql` shell

```bash
sudo -u postgres psql
```

(Enter password for `sudo`)

- Create db for an existing postgres user:

> DON'T FORGET SEMICOLON (`;`) FOR POSTGRES SHELL COMMANDS

```psql
CREATE DATABASE <db_name> OWNER <pg_username>;
```

This will grant all privileges to the user by default

- check if the db is created

```psql
\l
```

### 3.2.3. Configure PostgreSQL and environment variables

Install `django-environ` using `uv`:

```bash
uv add django-environ
```

Create a `.env` file in the project root:

```
DEBUG=True
SECRET_KEY=<django_secret_key>
DATABASE_URL=postgresql://<db_owner>:<owner_password>@<host>:5432/<db_name>
```

Replace `<db_owner>` and `<owner_password>` with your postgres user and password. For local dev `<host>` will be `localhost`

> [!IMPORTANT]
> Check that the `.env` is added to `.gitignore`

Generate a secret key for Django:

```bash
uv run python -c "import secrets; print(secrets.token_urlsafe(50))"
```

Copy the generated value and use it as the value of `SECRET_KEY` in `.env`.

### 3.2.4. Configure `settings.py`

Modify `config/settings.py` to read the environment variables:

```py
import environ
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# Initialize environ
env = environ.Env(
    # set casting, default value
    DEBUG=(bool, False)
)

# Read the .env file
env.read_env(BASE_DIR / '.env')

# Use the variables
SECRET_KEY = env('SECRET_KEY')
DEBUG = env('DEBUG')

DATABASES = {
    'default': env.db(), # django-environ reads DATABASE_URL and converts it into Django's DATABASES configuration.
}
DATABASES["default"]["CONN_MAX_AGE"] = 600
```

### 3.2.5. Verify the PostgreSQL configuration

Delete the default SQLite database if it was created:

```bash
rm db.sqlite3
```

Run the migrations:

```bash
uv run manage.py migrate
```

Then start the development server:

```bash
uv run manage.py runserver
```

Check that the application works correctly.

For an additional check, verify that Django's database tables were created in PostgreSQL:

```bash
psql -U <db_user> -d <db_name>
```

Inside the PostgreSQL shell:

```psql
\dt
```

You should see tables such as `auth_user`, `django_migrations`, `django_session`, etc.

These tables are created by the migrations associated with the applications listed in `INSTALLED_APPS`.

Exit the PostgreSQL shell with:

```psql
\q
```

### 3.2.6. Git commit

- As initial setups have completed, do your first commit (optionally push to a github repo)

## 3.3. Configure templates, static files and media

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
uv add pillow
```

## 3.4. (Optional) TailwindCSS and DaisyUI setup for frontend

- will be updated later

<!-- ============================END INITIAL DJANGO SETUPS============================== -->

**NOW YOU ARE READY TO CREATE PROJECT-SPECIFIC APPS AND DO OTHER SETUPS**
