---
layout: blog
title: "End to end web app with Django-Rest-Framework & AngularJS."
subtitle: "[Part 4]."
cover_image: images/blog/django+angular.png
cover_image_caption: "django+angular"
tags: [angular, django, ui]
---

In this part we will create a tags app for our project.

```bash
>  git checkout part4
```

### Backend

Let’s create a simple model for our tags,

```python
from django.db import models
from django.utils.translation import ugettext_lazy as _
from django.template.defaultfilters import slugify


class Tag(models.Model):
    slug = models.SlugField(max_length=100, verbose_name=_('slug'), blank=True)
    name = models.CharField(max_length=100)

    def __unicode__(self):
        return "%s" % self.name

    def save(self, *args, **kwargs):
        if not self.pk:
            self.slug = slugify(self.name)
        super(Tag, self).save(*args, **kwargs)
```

Create post_syncdb signal handler to create some tags

```python
from django.db.models import signals
from tags.models import Tag


def create_tags(app, created_models, **kwargs):
        Tag.objects.get_or_create(name="Architecture")
        Tag.objects.get_or_create(name="Art")
        Tag.objects.get_or_create(name="Books")
        Tag.objects.get_or_create(name="Cars & Motorcycles")
        Tag.objects.get_or_create(name="DIY & Crafts")
        Tag.objects.get_or_create(name="Design")
        Tag.objects.get_or_create(name="Drink")
        Tag.objects.get_or_create(name="Education")
        Tag.objects.get_or_create(name="Fashion")
        Tag.objects.get_or_create(name="Film, Music")
        Tag.objects.get_or_create(name="Food")
        Tag.objects.get_or_create(name="Games")
        Tag.objects.get_or_create(name="Gardening")
        Tag.objects.get_or_create(name="Geeg")
        Tag.objects.get_or_create(name="Hair & beauty")
        Tag.objects.get_or_create(name="Health & fitness")
        Tag.objects.get_or_create(name="History")
        Tag.objects.get_or_create(name="Holidays & Events")
        Tag.objects.get_or_create(name="Home Decor")
        Tag.objects.get_or_create(name="Humor")
        Tag.objects.get_or_create(name="Illustrations")
        Tag.objects.get_or_create(name="Kids")
        Tag.objects.get_or_create(name="Men")
        Tag.objects.get_or_create(name="Outdoors")
        Tag.objects.get_or_create(name="Photography")
        Tag.objects.get_or_create(name="Products")
        Tag.objects.get_or_create(name="Quotes")
        Tag.objects.get_or_create(name="Science a Nature")
        Tag.objects.get_or_create(name="Sports")
        Tag.objects.get_or_create(name="Technology")
        Tag.objects.get_or_create(name="Travel")
        Tag.objects.get_or_create(name="Weddings")
        Tag.objects.get_or_create(name="Women")
        Tag.objects.get_or_create(name="Videos")

signals.post_syncdb.connect(create_tags, dispatch_uid=Tag)
```

the app contains a migration, use south to migrate this app.

Let’s add views to query tags, list tags, search and autocomplete

```python
from rest_framework import generics
from rest_framework import permissions
from tags.models import Tag
from tags.serializers import TagSerializer


class TagList(generics.ListAPIView):
    model = Tag
    serializer_class = TagSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly,)

    def get_queryset(self):
        """
        This view should return a list of
            if q, all tags that contain q
            else, all tags
        """
        queryset = Tag.objects.all()
        query = self.request.QUERY_PARAMS.get('q', None)
        if query is not None:
            queryset = queryset.filter(name__icontains=query)
        return queryset


class TagDetail(generics.RetrieveUpdateDestroyAPIView):
    """
    Retrieve, update or delete a tag instance.
    """
    model = Tag
    serializer_class = TagSerializer
    permission_classes = (permissions.IsAuthenticatedOrReadOnly,)
```

Now we should update the post model to include tags field

```python
tags = models.ManyToManyField('tags.Tag', related_name='posts')
```

We should also add a tags field in the post serializer

```python
from rest_framework import serializers
from posts.models import Post
from tags.serializers import TagSerializer


class PostSerializer(serializers.HyperlinkedModelSerializer):
    author = serializers.Field(source='author.username')
    tags_details = TagSerializer(source='tags', read_only=True)
    api_url = serializers.SerializerMethodField('get_api_url')

    class Meta:
        model = Post
        fields = ('id', 'title', 'description', 'created_on', 'author', 'tags',
                  'tags_details', 'url', 'api_url')
        read_only_fields = ('id', 'created_on')

    def get_api_url(self, obj):
        return "#/post/%s" % obj.id
```

### Frontend

We should start first by creating a service to manage our tags

```js
Blog.factory('TagService', function ($http, $q) {
    var api_url = "/tags/";
    var api_suffixe = "/posts/";
    return {
        get: function(tag_id){
            var url = api_url + tag_id+ "/";
            var defer = $q.defer();
            $http({method: 'GET', url: url}).
                success(function(data, status, headers, config) {
                    defer.resolve(data);
                }).
                error(function(data, status, headers, config) {
                    defer.reject(status);
                });
            return defer.promise;
        },
        list: function(){
            var defer = $q.defer();
            $http({method: 'GET', url: api_url}).
                success(function(data, status, headers, config) {
                    defer.resolve(data);
                }).
                error(function(data, status, headers, config) {
                    defer.reject(status);
                });
            return defer.promise;
        },
        query: function(text){
            var url = api_url + '?q=' + text;
            var defer = $q.defer();
            $http({method: 'GET', url: url}).
                success(function(data, status, headers, config) {
                    defer.resolve(data);
                }).
                error(function(data, status, headers, config) {
                    defer.reject(status);
                });
            return defer.promise;
        }
    }
});
```

Now we can use the tags service in the controllers to query tags, make an autocomplete, and associate tags to posts

```js
Blog.controller('PostController', function ($scope, $routeParams, $location, PostService, TagService, GlobalService, post) {
    $scope.post = post;
    $scope.globals = GlobalService;
    var failureCb = function (status) {
        console.log(status);
    }
    //options for modals
    $scope.opts = {
        backdropFade: true,
        dialogFade: true
    };
    //open modals
    $scope.open = function (action) {
        if (action === 'edit'){
            $scope.postModalEdit = true;
        };
    };
    //close modals
    $scope.close = function (action) {
        if (action === 'edit'){
            $scope.postModalEdit = false;
        };
    };
    //calling board service
    $scope.update = function () {
        PostService.update($scope.post).then(function (data) {
            $scope.post = data;
            $scope.postModalEdit = false;
        }, failureCb);
    };
    $scope.getTag = function (text) {
        return TagService.query(text).then(function (data) {
            return data;
        }, function (status) {
            console.log(status);
        });
    };
    $scope.selectTag = function () {
        if (typeof $scope.selectedTag === 'object') {
            $scope.post.tags.push($scope.selectedTag.url);
            $scope.post.tags_details.push($scope.selectedTag);
            $scope.selectedTag = null;
        }
    };
    $scope.removeTag = function (category) {
        var index = $scope.post.tags_details.indexOf(category);
        $scope.post.tags_details.splice(index, 1);
        var index = $scope.post.tags.indexOf(category.url);
        $scope.post.tags.splice(index, 1);
    };
});
```

Finnaly we can update our templates to show the list of tags for each post.

```html
<p>
   <ul class="inline">
      <li ng-repeat="tag in post.tags_details">
          <span class="label"><i class="icon-white icon-tag"></i> {[{tag.name}]}</span>
       </li>
   </ul>
</p>
```
