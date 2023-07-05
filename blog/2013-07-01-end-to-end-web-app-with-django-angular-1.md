---
layout: blog
title: "End to end web app with Django-Rest-Framework & AngularJS."
subtitle: "[Part 1]."
cover_image: images/blog/django+angular.png
cover_image_caption: "django+angular"
tags: [angular, django, ui]
---

A REST API allows your users to interact with your website from anything that can send an HTTP request. In this post we will create a RESTful api in Django using the Django-Rest-Framework. And we will access this api using a client developed under AngularJS.

To utilize the API developed in Django, we are going to use $http & $q services in Angular. The $http service is a core Angular service that facilitates communication with the remote HTTP servers via browser’s XMLHttpRequest object or via JSONP. $q is promise implementation that comes with Angular.

### Preparing the environment

First thing to do is to prepare the environment, I suppose that most of you use virtualenv.

```bash
>  virtualenv django-angular
>  cd django-angular
>  source bin/activate  
```

Clone this repos :

[https://github.com/mouradmourafiq/django-angular-blog](https://github.com/mouradmourafiq/django-angular-blog)

Checkout to part1 branch.

```bash
>  git clone https://github.com/mouradmourafiq/django-angular-blog
>  cd django-angular-blog
>  git checkout part1
```

Install the packages.

```bash
>  pip install -r blog/requirements.pip
```

Copy the `local_settings.py` from `templates/template_local_settings.py`

```bash
>  cp blog/templates/template_local_settings.py blog/blog/local_settings.py
```

and update the file with the correct informations.

### The backend

We will create a simple model for our posts app.

```python
from django.db import models
from django.template.defaultfilters import slugify


class Post(models.Model):
    """
    An item created by a user.
    """
    title = models.CharField(max_length=100)
    slug = models.SlugField()
    description = models.TextField(blank=True, help_text=(
        "If omitted, the description will be the post's title."))
    is_active = models.BooleanField(default=True, blank=True)
    created_on = models.DateTimeField(auto_now_add=True)
    updated_on = models.DateTimeField(auto_now=True)

    def save(self, *args, **kwargs):
        self.do_unique_slug()
        if not self.description:
            self.description = self.title
        super(Post, self).save(*args, **kwargs)

    def do_unique_slug(self):
        """
        Ensures that the slug is always unique for this post
        """
        if not self.id:
            # make sure we have a slug first
            if not len(self.slug.strip()):
                self.slug = slugify(self.title)

            self.slug = self.get_unique_slug(self.slug)
            return True

        return False

    def get_unique_slug(self, slug):
        """
        Iterates until a unique slug is found
        """
        orig_slug = slug
        counter = 1

        while True:
            posts = Post.objects.filter(slug=slug)
            if not posts.exists():
                return slug

            slug = '%s-%s' % (orig_slug, counter)
            counter += 1
```

Now we have our model we can create a serializer in order to provide a way of serializing and deserializing the post instances into representations such as json.

DRF comes with a `HyperlinkedModelSerializer` class, which is a serializer classe that map closely to model definitions. rather than primary keys, it uses hyperlinks to represent relationships.

```python
from rest_framework import serializers
from posts.models import Post


class PostSerializer(serializers.HyperlinkedModelSerializer):
    api_url = serializers.SerializerMethodField('get_api_url')

    class Meta:
        model = Post
        fields = ('id', 'title', 'description', 'created_on', 'url', 'api_url')
        read_only_fields = ('id', 'created_on')

    def get_api_url(self, obj):
        return "#/post/%s" % obj.id
```

Finally, we need to write our views and urls. We will use the class based views, rather than function based views.

```python
from rest_framework import generics
from posts.models import Post
from posts.serializers import PostSerializer


class PostList(generics.ListCreateAPIView):
    """
    List all boards, or create a new board.
    """
    model = Post
    serializer_class = PostSerializer


class PostDetail(generics.RetrieveUpdateDestroyAPIView):
    """
    Retrieve, update or delete a board instance.
    """
    model = Post
    serializer_class = PostSerializer
```

```python
from django.conf.urls import patterns, include, url
from rest_framework.urlpatterns import format_suffix_patterns
from posts import views

urlpatterns = patterns('',
    url(r'^$', views.PostList.as_view(), name='post-list'),
    url(r'^(?P<pk>[0-9]+)/$', views.PostDetail.as_view(), name='post-detail'),
)


urlpatterns = format_suffix_patterns(urlpatterns)
```

Django REST Framework supports generating human-friendly HTML output for each resource when the HTML format is requested. These pages allow for easy browsing of resources, as well as forms for submitting data to the resources using POST, PUT, and DELETE.

You can test for yourself : http://localhost:8000/posts/

The next part of this tutorial will deal with the front end of this app with angularJs.
