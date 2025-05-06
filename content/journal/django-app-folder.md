---
title: "Project Structure"
date: "2025-04-30" 
author: "Mateusz Dubiel"
summary: ""
description: ""
readTime: true
autonumber: true
showTags: true
hideBackToTop: false
draft: true
tags: []
commits: []
prev_page: "/journal/project-structure" 
---

## Django project folder for multiple apps

Django offers a very flexible ways to create a project structure, when browsing the web for any directions (or just simply follow the official documentation), you can easy find number of solutions.

The first thought might be to use a recommendation from Django, what is perfect start and would work for many projects very well, however in this one I'd like to "complicate" things a bit, as I want to have one project but many apps, serving for different purposes (I name it "Domains"). Here is the idea.

The key is to separate functional domains into different Django applications while keeping shared logic (like models) in a dedicated place.

### The Core

One of the "apps" in Django terminology will be the core, the place where I'd like to define all models, all managers, and put all utilities and any other means of common code base. Reasoning for this is, to have a common structure for all domains I'll create later - this let me simply use  `core.*` reference to access this code.
This is enormous simplification, and I write code once!

### Webapp

The most important part of this project now is this monolithic webapplication, saying monolithic I mean frontend and backend are build in one app (and then later deployed in one container) rather to be separated, what could be done later if needed. This application will use models definied in `core` and all utilities definied there as well. The code here is pure application related. This is my first domain:The Application

### API

Later, I'd like to introduce API, that user can use to access data. This will be also used by mobile application, and from other web frontends. This of course would utilize same models from `core`, and would keep the same structure as other components in this project. The major thing here would be applicaiton api views. This is second domain of this application: "API domain"

### Analitics

For marketing purposes (this is not a scope of my application now) is that I pin marketing tools to this app thru separate domain, but it would use exactly same data. Only frontend will change and will have dedicated views. Let's call it third domain: "Analytics domain".

With this approach, I can add as many functional domains as I like, still keeping the core structure intacted and aligned across entire project.

### TailwindCSS source

As I'm going to use TailwindCSS for the frontend, I need to have a source directory where I would place all inputs for Tailwind and its utilities (like npm packages).

### Statics source and local CDN

One of other considerations is how to serve static files. As on production they would be delegated to file bucket, in local development it is need to simulate this behaviour.

The first directory is the source, where I put all required CSS, JS, and image files. Second directory is `local_cdn` from where files would be served, and for synchronization fo that folder `collectstatics` tools is used.

In the devlopment static files will be server thru the Django (read more about this in django documentation) but in production all statics (whatever match `/static/`) will be server by separate HTTPD server to avoid performance issues.

How to work with statics (and media files) would be described in separate entry.

## Benefits of this approach

- **Separation** of functional domains: The data layer (`core`), web application (`webapp`), and API  (`api`) are distinct, making the codebase organized and easy to understand.
- **Reusability**: Models defined in core are easily imported and used by all views.
- **Maintainability**: Data model is centralized in the `core`. I can modify an application view without affecting the API views.
- **Scalability**: It's clear where to add new features should go. It would be beneficial later to create separate containers for each domain and deploy them on Production separatelly (though they'd still share the database).

## The structure

Project structure is the following:

```
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
│   ├── models.py       # ALL  database models go here
│   ├── tests.py        # Tests for core logic
│   └── utils.py        # Optional: Shared utility functions
├── web/                #  main web application
│   ├── __init__.py
│   ├── admin.py        # Admin configurations specific to webapp (if any)
│   ├── apps.py
│   ├── migrations/     # Migrations specific to webapp
│   │   └── __init__.py
│   ├── models.py       # empty or minimal - models are in core
│   ├── tests.py        # Tests for webapp views/forms etc.
│   ├── views.py        # Views that render templates, handle form submissions, etc.
│   ├── urls.py         # URL patterns specific to the webapp
│   └── templates/      # HTML templates for the webapp
│       └── webapp/
│           └── ...     # Your template files
├── api/                # Your future API application
│   ├── __init__.py
│   ├── admin.py        # Admin configurations specific to API
│   ├── apps.py
│   ├── migrations/     # Migrations specific to API
│   │   └── __init__.py
│   ├── models.py       # empty or minimal - models are in core
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

For this basic folder structure, let's use django management tools to do that, it will generate all necessary files accordingly depending of ised django version. You can create app folders manually but is is not recommended way to do that.

ensure venv is loaded
which django-admin

django-admin startproject webapp .
python manage.py startapp core
python manage.py startapp webapp
# When ready for API:
# python manage.py startapp api

Configure settings.py:
Add your new apps to INSTALLED_APPS:


Python
# myproject/myproject/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core',      # Add your core app
    'webapp',    # Add your webapp
    # 'api',     # Add when you create it
]

# myproject/myproject/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('webapp.urls')), # Include webapp urls at the root
    # When ready for API:
    # path('api/', include('api.urls')), # Include api urls, typically with a prefix
]

Setup Node.js Dependencies (Tailwind & DaisyUI)
npm init -y
npm install -D tailwindcss@latest daisyui
npx tailwindcss init
mkdir src
mv tailwind.config.js src/

Update src/tailwind.config.js: Edit the file to configure Tailwind and add DaisyUI.
JavaScript
// src/tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    // Scan all HTML files in any 'templates' directory within any subdirectory
    "../**/templates/**/*.html",
    // If you have JS files that dynamically add classes, add them here too:
    // "../**/static/js/**/*.js",
  ],
  theme: {
    extend: {},
  },
  plugins: [
    require('daisyui'), // Add DaisyUI plugin
  ],

  // Optional: DaisyUI configuration
  daisyui: {
    themes: ["light", "dark", "cupcake"], // Example themes
    // Other options: styled, base, utils, logs
  },
}

/* src/input.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* You can add your own global custom styles here */




