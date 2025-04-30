---
title: "Technology stack selection for Beryl"
date: "2025-04-18"
author: "Mateusz Dubiel"
summary: "Exploring the core technologies selected for the Beryl project, including Django, HTMX, TailwindCSS, PostgreSQL, Docker, and more, with rationale for each choice."
description: "A detailed look at the technology stack chosen for the Beryl full-stack application. Covers frontend (HTMX, TailwindCSS), backend (Django, Python, PostgreSQL, Celery), development tools, and infrastructure plans (Docker, Redis)."
readTime: true
autonumber: true
showTags: true
hideBackToTop: false
draft: false
tags: ["Django", "Full-Stack", "Open-Source", "Beryl", "Development", "Project Chronicle", "Tech Stack", "Python", "HTMX", "TailwindCSS", "PostgreSQL", "Docker", "Redis", "Celery", "django-ninja"]
commits: []
prev_page: "/journal/user-stories"
next_page: "/journal/project-structure"
---

## Technology stack for the app

This covers the majority of technologies I plan to use when building this application. There will likely be minor additions around libraries and frameworks later; however, here I'd like to focus on the major elements.

The list below is mostly based on my experience. I'm aware that there could be other options which might suit the project better or perhaps be easier to work with initially. My opinion is that familiar technologies, even if not objectively the "best", can yield better results when you have experience and know how to leverage them effectively.

### Frontend

For the frontend, I plan to use server-rendered HTML enhanced with HTMX. I'm a fan of minimizing the JavaScript footprint in web apps, as heavy JS frameworks tend to overload the application, making them feel less responsive, rather than smooth. My goal is a lighter, more nimble experience.

The look and feel will be based on DaisyUI, a component library built on top of TailwindCSS. TailwindCSS is a utility-first CSS framework that allows for rapid UI development with minimal custom, complemented by the Lucide icon library.

**HTMX** will provide the necessary interactivity for features like autocomplete or dynamic notifications, integrating well with the server-rendered approach.

 - [DaisyUI](https://daisyui.com/)
 - [TailwindCSS](https://tailwindcss.com/)
 - [Lucide](https://lucide.dev/)
 - [HTMX](https://htmx.org/)

### Backend

My choice here is the Python and Django combination – very mature and commonly known technologies with huge community support and a vast ecosystem of additional libraries that help streamline parts of the application build.

As the database, I will use PostgreSQL. Firstly, because I have more experience with it. Secondly, it is very well supported in containerized environments like Kubernetes, which is a potential target infrastructure.

For background and scheduled tasks, Celery will be used, as it integrates well with Django.

For potential API endpoints, I plan to use django-ninja. It's a well-maintained, modern alternative for building APIs within Django, offering flexibility if needed.

A detailed breakdown of the specific Django libraries used will be covered in a future post.

 - [Python 3](https://www.python.org/)
 - [Django 5](https://www.djangoproject.com/)
 - [PostgreSQL](https://www.postgresql.org/)
 - [Django-ninja](https://django-ninja.rest-framework.com/)
 - [Celery](https://docs.celeryq.dev/en/stable/) 

### Development

My personal choice for an editor is VSCode, as I often do remote development over SSH, and it works extremely well in this scenario.

I will use pip and venv for managing Python dependencies and npm for frontend build tools (like TailwindCSS).

 - [VSCode](https://code.visualstudio.com/)
 - [pip](https://pip.pypa.io/en/stable/)
 - [npm](https://www.npmjs.com/)

### Infrastructure

While I'm far from building the production infrastructure at this stage, it's the perfect moment to consider how this software will eventually be delivered. For deployment flexibility, the application needs to be **containerized**. This allows it to run smoothly on any major cloud provider, connected to other required services like the database or Celery workers.

The CI/CD pipeline will be implemented using GitHub Actions.

Caching will be handled by Redis, which will also serve as the message broker for Celery. Local development orchestration will likely be managed using Docker Compose.

 - [Docker](https://www.docker.com/) / [containerd](https://containerd.io/)
 - [Docker Compose](https://docs.docker.com/compose/)
 - [GitHub Actions](https://github.com/features/actions)
 - [Redis](https://redis.io/)

 ## Conclusion

This stack represents a **solid foundation** for Beryl. It may evolve or be extended (perhaps for a mobile app someday), should the need arise.

Choosing a tech stack is rather like equipping an expedition. You pack the sturdy boots (Django, Python), the all-weather gear (Docker), the compass (HTMX), and a few experimental gadgets (django-ninja). You *hope* you've packed correctly for the journey ahead, but experience suggests you'll inevitably wish you'd brought a different kind of rope, or perhaps more biscuits. Nevertheless, the journey must begin, map in hand, ready for whatever dragons, swamps, or unexpected banjo-playing trolls the landscape of development might reveal.
