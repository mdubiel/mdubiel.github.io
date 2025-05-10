---
title: "Structuring Django Project: A Multi-App Approach"
date: "2025-05-08" 
author: "Mateusz Dubiel"
summary: "Outlines the Django project structure for Beryl, detailing a multi-app architecture with 'core', 'web', and 'api' applications to promote separation of concerns and scalability."
description: "Explains the rationale and implementation of a multi-app Django project structure for Beryl. Covers the roles of 'core', 'web', and 'api' apps, static file management with TailwindCSS, and initial setup steps for a maintainable and scalable codebase."
readTime: true
autonumber: true
showTags: true
hideBackToTop: false
draft: false
tags: ["Django", "Project Structure", "Architecture", "Beryl", "Development", "Django Apps", "Core App", "API Design", "Static Files", "TailwindCSS", "Scalability", "Maintainability"]
commits: []
prev_page: "/journal/project-structure"
next_page: "/journal/initial-run"
---

## Django project folder for multiple apps

**Django** offers very flexible ways to create a project structure. When browsing the web for guidance (or simply following the [official Django documentation](https://docs.djangoproject.com/en/stable/)), you can easily find numerous solutions.

The first thought might be to use the recommendation from Django, which is a perfect start and would work very well for many projects. However, in this project, I'd like to "complicate" things a bit. My goal is to have one project with multiple **Django apps**, each serving a distinct purpose (which I'll refer to as "**domains**"). The key is to separate these functional domains into different Django applications while keeping shared logic (like **models**) in a dedicated place.

This is the idea.

### The Core

One of the "apps" in Django terminology will be the **`core` app**. This is where I plan to define all **models**, **managers**, and place all **utilities** and any other common codebase components. The reasoning for this is to establish a common structure for all domains I'll create later. This allows me to simply use a `core.*` reference to access this shared code.

This offers an enormous simplification, as I write the code once!

### Webapp

The most important part of this project, for now, is the **monolithic web application**. By monolithic, I mean the frontend and backend are built within a single app (and later deployed in one container), rather than being separated—which could be an option for a future revision. This application will use models defined in `core` and all utilities defined there as well. The code here is purely application-related. This constitutes my first domain: **The Application**.

### API

Later, I plan to introduce an **API** that users can use to access data. This API will also be used by a potential mobile application and other web frontends. Naturally, it will utilize the same models from `core` and maintain the same structure as other components in this project. The major component here will be the application's API views. This forms the second domain of this application: the **API Domain**.

### Analytics

For marketing purposes (though not within the current scope of my application), I envision pinning marketing tools to this app through a separate domain. This domain would use the exact same data. Only the frontend will differ, featuring dedicated views. Let's call this the third domain: the **Analytics Domain**.

With this approach, I can add as many functional domains as needed, while keeping the core structure intact and aligned across the entire project.

### TailwindCSS source

Since I'm going to use **TailwindCSS** for the frontend, I need a source directory (e.g., `src/`) to place all inputs for Tailwind and its utilities (like **npm** packages).

### Statics source and local CDN

Another important consideration is how to serve **static files**. While in production they would be delegated to a file bucket (like Google Cloud Storage (GCS) or Amazon S3), local development requires simulating this behavior.

The first directory is the **source** (e.g., `static/` for compiled assets, `src/` for source assets), where I'll place all required CSS, JS, and image files. The second directory, `local_cdn/`, is where files will be served from after being collected by Django's `collectstatic` command, simulating a CDN locally.

In development, static files will be served through Django (refer to the Django documentation on static files for more details). However, in production, all static files (matching `/static/`) will be served by a separate HTTPD server (e.g., Nginx, Apache) to avoid performance issues.

How to work with static and media files will be described in a separate entry.

## Benefits of this approach

- **Separation**: Functional **domains** such as the data layer (`core`), web application (`web`), and API (`api`) are distinct. This makes the codebase more organized and easier to understand.
- **Reusability**: **Models** defined in `core` are easily imported and utilized by all views across different applications.
- **Maintainability**: The data model is centralized in the **`core` app**. I can modify an application view without inadvertently affecting API views (as long as the underlying model contracts are respected).
- **Scalability**: It's clear where new features should be added. This structure also paves the way for future scalability, such as creating separate containers for each domain and deploying them independently in production (though they would still share the same database).

## The structure

Project structure is the following:

```bash
webapp/                 # Project root directory
├── manage.py           # Django management script
├── webapp/             # project configuration directory
│   ├── __init__.py
│   ├── settings.py     # Shared settings
│   ├── urls.py         # Main URL dispatcher
│   └── wsgi.py         # WSGI entry point
├── core/               # Application for shared models, managers, utilities
│   ├── __init__.py
│   ├── admin.py        # Admin configurations for core models
│   ├── apps.py
│   ├── migrations/     # Database migrations for core models
│   │   └── __init__.py
│   ├── models.py       # ALL database models go here
│   ├── tests.py        # Tests for core logic
│   └── utils.py        # Optional: Shared utility functions
├── web/                #  main web application
│   ├── __init__.py
│   ├── admin.py        # Admin configurations specific to web
│   ├── apps.py
│   ├── migrations/     # Migrations specific to webapp
│   │   └── __init__.py
│   ├── models.py       # empty or minimal
│   ├── tests.py        # Tests for web views/forms etc.
│   ├── views.py        # Views that render templates, handle form submissions, etc.
│   ├── urls.py         # URL patterns specific to the webapp
│   └── templates/      # HTML templates for the webapp
│       └── webapp/
├── api/                # API application
│   ├── __init__.py
│   ├── admin.py        # Admin configurations specific to API
│   ├── apps.py
│   ├── migrations/     # Migrations specific to API
│   │   └── __init__.py
│   ├── models.py       # empty or minimal
│   ├── tests.py        # Tests for API endpoints
│   ├── views.py        # Views that handle API requests (return JSON/data)
│   └── urls.py         # URL patterns specific to the API (often prefixed, e.g., /api/)
├── src/                # Development files
├── static/             # Your development static files
│   └── css/
│       └── tailwind.css
├── local_cdn/          # The directory created by collectstatic
│   ├── css/
│   │   └── tailwind.css
│   └── admin/          # Static files from django.contrib.admin, etc.
│       ├── css/
│       └── ...
└── pyproject.toml      # List of project dependencies (Django, etc.)
```

## Execution

To create this basic folder structure, let's use Django's management tools. These tools will generate all necessary files according to the Django version being used. While you can create app folders manually, it is not the recommended approach.

Ensure the correct Python virtual environment is activated and verify the installed Django version:

```bash
$ source .venv/bin/activate
$ which django-admin
/home/mdubiel/github/beryl3/.venv/bin/django-admin
$ django-admin --version
5.2
```

### Create Django project and applications:

```bash
cd webapp/
django-admin startproject webapp .
python manage.py startapp core
python manage.py startapp web
python manage.py startapp api
```

### Configure settings.py

Add your new apps to `INSTALLED_APPS`:

```python
# webapp/webapp/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core.apps.CoreConfig',      # Add Core app
    'web.apps.WebConfig',        # Add Web Application
    'api.apps.ApiConfig',        # Add API Application

]
```

A word on URL structure: All default requests to the application domain (e.g., `https://beryl.app/`) will be served by the **Web Application** module (web app). All other modules, like the API, must have a defined prefix. For instance, to call the API, one would use `https://beryl.app/api/`.

The only potential pitfall is to avoid using the same top-level prefix (e.g., `/api/`) anywhere else in other applications, particularly in the web application. This means I must avoid defining paths like `/api/...` within the web application's `urls.py`.

```python
# webapp/webapp/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('web.urls')), # Include web urls at the root
    path('api/', include('api.urls')), # Include api urls, typically with a prefix
]
```

Create minimal `urls.py` files for the `web` and `api` apps. Remember how Python's import system works here: `web.` refers to the `web/` directory (making it a package), and `urls` refers to the `urls.py` file within it.

```bash
$ echo 'urlpatterns = []' > web/urls.py
$ echo 'urlpatterns = []' > api/urls.py
```

### Setup Node.js Dependencies

**TailwindCSS** and **DaisyUI** require an additional development folder and build tools, which are typically managed using `npm` (Node Package Manager).

Check the official TailwindCSS documentation for various ways to install this component. Here, I'm using my preferred method, the Tailwind CSS CLI.

```bash
$ npm init -y
$ npm install -D tailwindcss@latest @tailwindcss/cli daisyui@latest
$ mkdir src
```

Next, create your Tailwind input CSS file (e.g., `src/input.css`) with basic utilities. Keep in mind that there are significant differences between Tailwind CSS v3 and v4. Always refer to the official TailwindCSS documentation for version-specific details.

```css
@import "tailwindcss";
@plugin "daisyui" {
  themes: nord --default, sunset --prefersdark;
}
```

That's all for the structural setup. In the next entry, we'll initialize the project and verify that everything is working as expected.