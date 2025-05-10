---
title: "Initial run and database setup"
date: "2025-05-10" 
author: "Mateusz Dubiel"
summary: "This post details the initial steps of running a Django application, including compiling Tailwind CSS, starting the development server, setting up a PostgreSQL database with Docker Compose, managing environment variables for security with django-environ, and applying initial database migrations."
description: "A step-by-step guide to the initial run and database configuration for a Django project. Covers Tailwind CSS, Django dev server, secure environment variable management with django-environ, PostgreSQL setup using Docker Compose, and initial database migrations with psycopg2."
readTime: true
autonumber: true
showTags: true
hideBackToTop: false
draft: false
tags: ["Django", "Tailwind CSS", "Docker", "PostgreSQL", "django-environ", "Python", "Web Development", "Setup", "Configuration", "Migrations"]
commits: []
prev_page: "/journal/django-app-folder"
---
## Initial run

In the previous entry, I established the entire structure for this Django application. Now, let's test if everything runs as expected.

### Tailwind

First, let's test that the TailwindCSS setup compiles the CSS files and places them in the expected output directory:

```bash 
$ npx @tailwindcss/cli -i src/input.css -o static/css/tailwind.css
/*! 🌼 daisyUI 5.0.35 */
≈ tailwindcss v4.1.5

Done in 247ms
```

It compiles successfully! It takes the configuration from `src/input.css`, compiles it, and writes the output to the `static/css/tailwind.css` file. We will source this CSS file later in our Django templates.

### Django server

Second, let's test our Django development server. The goal is to verify that the initial setup is correct and allows us to start the server. I need to specify a host and port because the default port `8000` might be busy on my development machine:

```bash
$ python manage.py runserver localhost:8763
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
May 08, 2025 - 13:45:11
Django version 5.2, using settings 'webapp.settings'
Starting development server at http://localhost:8763/
Quit the server with CONTROL-C.
```

The server runs as expected, but the output highlights a key issue we need to address: unapplied database migrations.

## Database setup and initial migrations

The next crucial step is setting up the database. While Django defaults to SQLite, I'll opt for a PostgreSQL database running in a Docker container. This approach, though slightly more complex to set up initially, more closely simulates a production environment and prepares the application for loads beyond what SQLite typically handles.

An additional aspect to cover here is how to manage database secrets (like passwords). The answer is to use environment variables and/or a secrets management system. Let's delve into this.

### Environment variables

A quick look into a newly generated Django settings.py file often reveals an entry like this:

```python
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'django-insecure-j_w53%gokd#1y8*!^l*-j61jwfid=o+pj39vv+qimh*6u_mc$3'
```

This is an immediate red flag. Storing secrets directly in version-controlled files means the same secret is used across all deployment environments, posing a significant security risk. To mitigate this, secrets should be stored externally and loaded by the application based on its environment. Fortunately, Python and Django offer straightforward ways to achieve this.

The core idea is to read environment variables at application startup and use their values instead of hardcoded ones. This can be managed in various ways: locally, a `.env` file is common for development, while cloud platforms and Docker orchestrators provide dedicated mechanisms for injecting environment variables and secrets.

For this project, I'll use the `django-environ` library to manage environment variables. Let's install it:

```bash
$ uv add "django-environ"
Resolved 8 packages in 512ms
Prepared 1 package in 122ms
Installed 1 package in 1ms
 + django-environ==0.12.0
```

Modify `settings.py` slightly to integrate `django-environ` and read these variables:

```python
from pathlib import Path
import os

import environ


# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

# Initialize environ
env = environ.Env(
    # set casting and default value for DEBUG
    DEBUG=(bool, False)
)

# Read variables from .env file
environ.Env.read_env(os.path.join(BASE_DIR, '.env'))

SECRET_KEY = env('SECRET_KEY')
DEBUG = env('DEBUG')
ALLOWED_HOSTS = env.list('ALLOWED_HOSTS', default=['127.0.0.1'])
```

Finally, create a `.env` file in the root directory of your Django project (alongside `manage.py`). Crucially, add `.env` to your `.`gitignore` file to prevent committing secrets to your repository. It should look something like this (use your own secure values):

```bash
SECRET_KEY='django-insecure-j_w53%gokd#1y8*!^l*-j61jwfid=o+pj39vv+qimh*6u_mc$3'
DEBUG=True
ALLOWED_HOSTS=.localhost,127.0.0.1
```

### Install database with docker-compose

The final setup piece is running our PostgreSQL database within a Docker container. I'm using Docker Compose for this, rather than a simple docker run command, because it simplifies managing multiple services. As the project grows, other components might also be containerized, and Docker Compose allows for consistent deployment and management from a single configuration file.

I'll assume Docker and Docker Compose are already installed; please refer to their official documentation if you need installation guides (Docker Desktop, Docker Compose).

First, let's add the necessary database credentials to our `webapp/.env` file (the same `.env` file we created earlier):

```bash
PG_USER=myuser
PG_PASSWORD=mypassword
PG_DB=mydatabase
PG_HOST=localhost
PG_PORT=5432
```

Create a `docker-compose.yaml` file, for example, in a `docker/` directory at the project root (i.e., one level above your webapp Django project directory), with the following content:

```yaml
services:
  postgres:
    image: postgres:15
    container_name: my_postgres
    environment:
      POSTGRES_USER: ${PG_USER}
      POSTGRES_PASSWORD: ${PG_PASSWORD}
      POSTGRES_DB: ${PG_USER}
    ports:
      - "${PG_PORT}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```


Now, run the Docker Compose setup. The `--env-file` flag directs Docker Compose to load variables from our `webapp/.env` file. Execute this command from your project's root directory (the one containing the docker and webapp folders):

```bash
$ docker compose --env-file webapp/.env --file docker/docker-compose.yaml --project-name beryl3 up -d
[+] Running 3/3
 ✔ Network beryl3_default         Created                                                                                                      0.2s 
 ✔ Volume "beryl3_postgres_data"  Created                                                                                                      0.0s 
 ✔ Container beryl3_postgres      Started    
```

At this moment, your PostgreSQL database container should be running and ready to accept connections.

### Initial migrations

To complete the database integration, Django needs to be configured to connect to our new PostgreSQL instance.

Since we're using PostgreSQL, Django requires a database adapter. The `psycopg2` library is the standard choice. To avoid compilation issues that can arise from the source distribution (which might require a full build toolchain), we'll install the pre-compiled binary version, `psycopg2-binary`:


```bash
$ uv add psycopg2-binary
Resolved 9 packages in 12ms
Installed 1 package in 29ms
 + psycopg2-binary==2.9.10
```


Next, update the `DATABASES` setting in your `webapp/settings.py` to use the environment variables for the PostgreSQL connection:

```python
DATABASES = {
    'default': {
        'ENGINE': env('DB_ENGINE', default='django.db.backends.postgresql'),
        'NAME': env('PG_DB'),
        'USER': env('PG_USER'),
        'PASSWORD': env('PG_PASSWORD'),
        'HOST': env('PG_HOST'),
        'PORT': env.int('PG_PORT'),  
    }
}
```

With the settings configured, let's test if Django can connect to the database using the `dbshell` management command. This command drops you into the native command-line client for your configured database.

```bash
$ python manage.py dbshell
psql (14.17 (Ubuntu 14.17-0ubuntu0.22.04.1), server 15.13 (Debian 15.13-1.pgdg120+1))
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
Type "help" for help.

beryl3_dev=# \dt
Did not find any relations.
```

Success! The connection works. The `\dt` command (for PostgreSQL) lists tables; it shows none because the database is fresh. Django requires initial migrations to set up its necessary tables (for authentication, sessions, admin interface, etc.). Let's apply these migrations and then re-check the tables.

Apply the initial migrations:

```bash
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
```

Now, let's query the tables again using `dbshell`:

```bash
$ python manage.py dbshell
psql (14.17 (Ubuntu 14.17-0ubuntu0.22.04.1), server 15.13 (Debian 15.13-1.pgdg120+1))
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
Type "help" for help.

beryl3_dev=# \dt
                    List of relations
 Schema |            Name            | Type  |   Owner    
--------+----------------------------+-------+------------
 public | auth_group                 | table | beryl3_dev
 public | auth_group_permissions     | table | beryl3_dev
 public | auth_permission            | table | beryl3_dev
 public | auth_user                  | table | beryl3_dev
 public | auth_user_groups           | table | beryl3_dev
 public | auth_user_user_permissions | table | beryl3_dev
 public | django_admin_log           | table | beryl3_dev
 public | django_content_type        | table | beryl3_dev
 public | django_migrations          | table | beryl3_dev
 public | django_session             | table | beryl3_dev
(10 rows)
```

Excellent! The necessary Django tables have been created. The application is now configured with a PostgreSQL database, and we're ready to proceed with development.
