---
layout: blog
title: "End to end web app with Django-Rest-Framework & AngularJS."
subtitle: "[Part 3]."
cover_image: images/blog/django+angular.png
cover_image_caption: "django+angular"
tags: [angular, django, ui]
---

In the 2 previous posts we built a backend API with DRF and a client with AngularJs.

In this part, we will add authentication and permission to our app. We will add some restrictions on who can edit and delete posts.

 * Authenticated users can create blog posts
 * Posts are tied to their author (edit/delete permissions)
 * Posts are read only for unauthenticated users

Since we are going to make some changes in our models, we need to install south. Update the environment

```bash
>  git checkout part3
>  pip install -r requirements.pip
```

And add south to the installed apps.

N.B. Because we are updating tables that are already in the db the initial schemamigration won’t work instead we use:

```bash
>  ./blog/manage.py  convert_to_south posts
```

this gets it in the right state for further alterations.

Let’s add a field to represent the author of this post.

```python
author = models.ForeignKey('auth.User', related_name='posts')
```

All we need to do now is to go back and run :

```bash
>  ./blog/manage.py schemamigration posts --auto
>  ./blog/manage.py migrate posts
```

You will be asked to specify a default value. enter 1 (the id of the first user by default)

### Associating Posts with Users

Right now, if a user created a post, there’d be no way of associating the user that created the post, with the post instance. The user isn’t sent as part of the serialized representation, but is instead a property of the incoming request.

The way we deal with that is by overriding a .pre_save() method on our posts views, that allows us to handle any information that is implicit in the incoming request or requested URL.

```python
def pre_save(self, obj):
    obj.author = self.request.user
```

### Updating our serializer

Now that posts are associated with the author that created them, let’s update our PostSerializer to reflect that. Add the following field to the serializer definition:

```python
from rest_framework import serializers
from posts.models import Post


class PostSerializer(serializers.HyperlinkedModelSerializer):
    author = serializers.Field(source='author.username')
    api_url = serializers.SerializerMethodField('get_api_url')

    class Meta:
        model = Post
        fields = ('id', 'title', 'description', 'created_on', 'author', 'url', 'api_url')
        read_only_fields = ('id', 'created_on')

    def get_api_url(self, obj):
        return "#/post/%s" % obj.id
```

### Adding required permissions to views

Now that code posts are associated with users, we want to make sure that only authenticated users are able to create, update and delete posts.

REST framework includes a number of permission classes that we can use to restrict who can access a given view. In this case the one we’re looking for is IsAuthenticatedOrReadOnly, which will ensure that authenticated requests get read-write access, and unauthenticated requests get read-only access.

We’d also like all posts to be visible to anyone, but make sure that only the author is able to update or delete it.

To do that we’re going to need to create a custom permission.

In the posts app, create a new file, `permissions.py`

```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow owners of an object to edit it.
    """

    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request
        if request.method in permissions.SAFE_METHODS:
            return True

        # Write permissions are only allowed to the owner of the tip
        return obj.author == request.user
```

### Setting authentication

The default authentication schemes may be set globally, using the `DEFAULT_AUTHENTICATION` setting. For example.

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
           'rest_framework.authentication.SessionAuthentication',
    )
}
```

We will make use of the DRF auth-api to authenticate our users. Add a pattern to include the login and logout in the browsable API.

```python
urlpatterns += patterns('',
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework')),
)
```

Finally we need to add some conditions in our templates, for both feeds and post view, to check if the user is authenticated or not. For this purpose, we are going to use the GlobalService that we initialize when loading the landing page of our app.

```html
<body data-spy="scroll" id="index" style="zoom: 1;" ng-controller="AppController" ng-init="initialize('{{ user.is_authenticated }}')">
```

And to use this global variable we can do something like this

```html
<a ng-show="globals.is_authenticated" class="btn" ng-click="open('create')"> Create</a>
<a ng-hide="globals.is_authenticated" class="btn" href="/api-auth/login/"> Create</a>
```

Also we update our templates to include the author informations

```html
<small><b>by</b> {[{post.author}]}</small>
```
