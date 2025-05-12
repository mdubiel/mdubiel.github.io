---
title: "Hello world!"
date: "2025-05-12" 
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
prev_page: "/journal/initial-run"
---

## Hello world!

It is now time to say "Hello World" from your Django application!

### 404!

Let's start the dev server and see how it looks at the begining:

```bash
python manage.py runserver localhost:8763
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
May 12, 2025 - 10:58:49
Django version 5.2, using settings 'webapp.settings'
Starting development server at http://localhost:8763/
Quit the server with CONTROL-C.

WARNING: This is a development server. Do not use it in a production setting. Use a production WSGI or ASGI server instead.
For more information on production servers see: https://docs.djangoproject.com/en/5.2/howto/deployment/
```

And point your browser to highlighted URL: http://localhost:8763/ and you can immadiatelly see `404` HTTP Error :

![Page not found!](page-not-found.png)

This is because we indeed have defaults definied in `urls.py` but currently the call to the `/` points nowhere. It is dispatched first by `webapp/webapp/urls.py` and redirected (or included) to the module specific configuration:

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('web.urls')),
    path('api/', include('api.urls')),
]
```

However, this file `webapp/web/urls.py` has no paths definied as we set it empty at the begining:

```python
urlpatterns = []
```

Let's fix it and set our "hello world".

### Add first path and first function view

In the `urlpatterns` list add a new entry, and specify:
 - `''` - to follow everything if unspecified, remeber here that it will concenate with everything else what "included" this file, however - because our include call is refering to empty string that, it this particular case concenate two empty strings. 
 - `views.index` - to use that view function to handle this request
 - `name='index'` - to give it a name we will use later to refer to it (eg.: when creating links)

Also, import necessary libraries:
 - `path` - to handle your URL request
 - `web.views` to import your views

```python
from django.urls import path
from web.views import index

urlpatterns = [
  path('', index, name='index'),
]
```

Now, I need to write the function `views.index`.
Import necessary libraries (`HttpResponse` from `django.http`) which is responsible to return the content to the client. Write the function itself in `webapp/web/views.py`:

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello from django!")

```

Function views are using `request` argument to handle data from the HTTP request, and return `HttpResonse`. You as a programer are only responsible to provide content, Django will handle the rest for you. In this example, function returns text, what would result in displaying "Hello from django!" in your browser.

![Hello world!](hello-world.png)

### Spining up with templates

 write about how to move this return text to template, and simple inheritance.


## Conclusion

`urls.py` is the place where you define you paths and match the function which will handle this request.

`views.py` is the file where you write most of the code, all view function will live here.

`templates` is a directory where all templates are located, remeber about specific order how system looks for template file to parse them.
