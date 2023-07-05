---
layout: blog
title: "What makes django amazing!"
subtitle: "My favourite Django-apps and packages that i use in almost every Django project."
cover_image: images/blog/django.png
cover_image_caption: "Django"
tags: [django, python, lists]
---

In this post I will cover some amazing Django-apps and packages that i use in almost every Django project.

- [psycopg](https://pypi.python.org/pypi/psycopg2):

Psycopg is the most popular PostgreSQL adapter for the Python programming language. At its core it fully implements the Python DB API 2.0 specifications.

- [django-debug-toolbar](https://pypi.python.org/pypi/django-debug-toolbar):

Really nice to have when developing Django project, not using it is insane.

- [django-extensions](http://django-extensions.readthedocs.org/en/latest/) :

collection of custom extensions that would save you a lot of time.

- [South](http://south.aeracode.org/):

Data migration in Django is free, it’s up to you to choose how to migrate your data. I think the best way you want to do it is via South.

- [django-registration](https://pypi.python.org/pypi/django-registration):

Simple and solid registration app. Although, you do have to make your own templates.

- [django-endless-pagination](http://django-endless-pagination.readthedocs.org/en/latest/index.html):

This app can be used to provide Twitter-style or Digg-style pagination.

- [django-social-auth](http://django-social-auth.readthedocs.org/en/latest/index.html):

Easy to setup social authentication/authorization mechanism for Django projects.

- [haystack](http://haystacksearch.org/):

This application add full text search capability to your application. Currently it support Solr, Whoosh and Xapian.

Do not copy/paste this libraries in the project directory. Do it the right way; a python dependency management.

Put this list in a requirements.txt file, then install them as proper dependencies in your virtualenv:

```bash
pip install -r requirements.txt
```
