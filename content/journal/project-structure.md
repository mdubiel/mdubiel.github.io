---
title: "Project Structure"
date: "2025-04-30" 
author: "Mateusz Dubiel"
summary: "An initial attempt to define a sensible directory structure for the Beryl project, aiming to establish a clear organization from the outset."
description: "Details the proposed directory layout for the Beryl project, considering the needs of components like Django, potential mobile apps, Docker configurations, and supporting scripts."
readTime: true
autonumber: true
showTags: true
hideBackToTop: false
draft: false
tags: ["Django", "Full-Stack", "Open-Source", "Beryl", "Development", "Project Chronicle", "Project Structure", "Architecture"]
commits: []
prev_page: "/journal/tech-stack"
next_page: "/journal/django-app-folder"
---


## Defining the project structure

Right, let's talk about organization. Before getting carried away with the actual coding – which inevitably brings its own set of challenges – it's a good idea to plan where everything will go. A well-defined project structure acts as a map, helping to keep track of all the different parts and preventing things from getting lost later on.

Without a clear structure, a project can quickly become rather difficult to navigate. Finding specific files might turn into a frustrating exercise, maintenance becomes a chore, and you risk stumbling over bits of code you wrote weeks ago and no longer understand – perhaps even being metaphorically tripped up by a rogue semicolon. This sort of disarray, while perhaps character-building, is generally counterproductive when the goal is to create functional software without unnecessary hassle.

Therefore, what follows is an initial proposal for organising the Beryl project's directories. It's presented with the full understanding that reality often has other plans, but consider it a starting point – a gentle nudge towards organisation, hoping the codebase doesn't immediately descend into an unmanageable tangle.

## Choosing the right tools

To get started, we first need to manually install the core tooling that will help manage the project.

I'm going to use **[UV](https://docs.astral.sh/uv/)** to manage this project. There are several alternatives; traditionally, working with **Django** often involves using *`pip`* and a *`venv`* (virtual environment) to manage dependencies effectively. However, `uv` aims to be a faster, unified replacement for tools like *`pip`*, *`pip-tools`*, *`poetry`*, and *`venv`*.

Given that this project aims to be reasonably modern and will eventually include several distinct parts (like the web application, mobile components, Docker configurations, and utility scripts), using a tool that can manage this complexity effectively makes sense. `uv`'s **`workspace`** feature is particularly helpful here.

There is a lot of advantages using `uv`, one is that I can set entire project from my existing GitHub repository, and add sub projects called here `workspaces` and use them separatelly for each of my components. Saying that - I would have different sets of libraries for webapp and different for mobile, and they would not collide.

It also uses `requirements.txt` and `pyproject.toml` files making it compatible with pip and poetry.

The structure of this project consists of following folders:
 - main directory, with `README.md`, `pyproject.toml` and similar files, creating the solid root for this project.
 - `webapp` - containing django web application
 - `mobile` - for future mobile application development
 - `docker` - componentes related to contenarization, with development environment setup and docker build files
 - `scripts` - supporting executables

### Project bootstrap

`uv` can be installed using snap:

{{< codeblock lang="bash" >}}
$ sudo snap install --classic astral-uv
{{< /codeblock >}}

From now, I will operate exclusivelly in `beryl3` directory, which in my case is a local github repository.

Prepare environment:

{{< codeblock lang="bash" >}}
$ uv init \
  --name beryl3 \
  --bare \
  --no-package \
  --description "Beryl, collection management system" \
  --author-from git
{{< /codeblock >}}

Setup first component which is Web Application:

{{< codeblock lang="bash" >}}
$ uv init \
  --name beryl3-webapp \
  --bare \
  --no-package \
  --description "Beryl, collection management system. Web application" \
  --author-from git \
  webapp/

Adding `beryl3-webapp` as member of workspace `/home/mdubiel/github/beryl3`
Initialized project `beryl3-webapp` at `/home/mdubiel/github/beryl3/webapp/`
{{< /codeblock >}}

Create addtional folders for docker components and scripts folder:

{{< codeblock lang="bash" >}}
$ mkdir docker scripts
{{< /codeblock >}}

Now, the folder structure should look like this:

{{< codeblock lang="bash" >}}
$ tree --dirsfirst -F
./
├── docker/
├── scripts/
├── webapp/
│   └── pyproject.toml
├── LICENSE
├── pyproject.toml
├── README.md
└── uv.lock

3 directories, 5 files
{{< /codeblock >}}

Let's add necessary elements to that, we need python3.12 for entire project, and django for webapp component:

{{< codeblock lang="bash" >}}
$ uv python install 3.12
Installed Python 3.12.10 in 1.75s
 + cpython-3.12.10-linux-x86_64-gnu
{{< /codeblock >}}

Set this python as default:
{{< codeblock lang="bash" >}}
$ uv python pin 3.12
Pinned `.python-version` to `3.12`
{{< /codeblock >}}

Install django:

{{< codeblock lang="bash" >}}
$ cd webapp/ && uv add "django>=5.2,<6.0"
{{< /codeblock >}}

Verify installation:

{{< codeblock lang="bash" >}}
$ uv run django-admin --version
5.2

$ uv run python --version
Python 3.12.10

$ uv tree
Resolved 7 packages in 1ms
beryl3-webapp v0.1.0
└── django v5.2
    ├── asgiref v3.8.1
    └── sqlparse v0.5.3
beryl3 v0.1.0
└── beryl3-webapp v0.1.0 (*)
(*) Package tree already displayed
{{< /codeblock >}}

## Next steps

The next step is to structure Django web application folder, what is explained in the next chapter.