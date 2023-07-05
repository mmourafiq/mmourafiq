---
layout: blog
title: "End to end web app with Django-Rest-Framework & AngularJS."
subtitle: "[Part 2]."
cover_image: images/blog/django+angular.png
cover_image_caption: "django+angular"
tags: [angular, django, ui]
---

Let’s pick up where we left off. If you haven’t already, make sure you go through part1 to create your base Django app with the Django REST Framework API setup.

Now, that we have our backend up and runing, we can get our hands dirty using AngularJs. We will create a simple one page app:

```bash
>  git checkout part2
```

Let’s create this one page served by our Django project.

```html
{% raw %}
{% extends "base.html" %}
{% load i18n %}
{% block extra_head %}
    <meta name="keywords" content=""/>
    <meta name="description" content=""/>
    <script src="{{ STATIC_URL }}js/app/controllers/app-controller.js"></script>
    <script src="{{ STATIC_URL }}js/app/controllers/feed-controller.js"></script>
    <script src="{{ STATIC_URL }}js/app/controllers/posts-controller.js"></script>
    <script src="{{ STATIC_URL }}js/app/services/app-service.js"></script>
    <script src="{{ STATIC_URL }}js/app/services/posts-service.js"></script>
    <script src="{{ STATIC_URL }}js/app/directives/directives.js"></script>
    <script src="{{ STATIC_URL }}js/app/filters/filters.js"></script>
{% endblock %}
{% block content %}
    <div class="padded-content">
        <div id="main-content" class="container">
            <div class="row">
                <div class="span12">
                    <div class="pending-bg" pendingbar>
                        <div class="alert alert-warning">loading ...</div>
                    </div>
                </div>
             </div>
                <ng-view viewstate></ng-view>
        </div>
    </div>
{% endblock %}
{% block extra_content %}
<script type="text/javascript"></script>
{% endblock %}
{% endraw %}
```

As you can see, our “index” page is very simple, it inherits from a “base” page, and has two important features, or new html tags. This tags, directives in the angular language, extend our html abilities. We will go through this a bit later.

For now, let’s have a look at the structure of our angular app.

N.B: There are a couple of different ways we can add Angular into our application. In this project, I didn’t follow any particular one.

 * static
   * css
   * img
   * js
     * app
     * controllers
     * directives
     * filters
     * services
     * views
     * app.js
   * libs

My angular app will live under the folder app following this particular structure. At this point it is fairly simple, and should allow us to do basic interaction with the backend (create post, list posts, view a post by id, and edit a post).

**App.js**

First thing first, let’s start with the “app.js”. This file is important because it is there where we set some global settings, and set the routing of our app.

```js
'use strict';

//(1)
var Blog = angular.module("Blog", ["ui.bootstrap", "ngCookies"], function ($interpolateProvider) {
        $interpolateProvider.startSymbol("{[{");
        $interpolateProvider.endSymbol("}]}");
    }
);

//(2)
Blog.run(function ($http, $cookies) {
    $http.defaults.headers.common['X-CSRFToken'] = $cookies['csrftoken'];
})

//(3)
Blog.config(function ($routeProvider) {
    $routeProvider
        .when("/", {
            templateUrl: "static/js/app/views/feed.html",
            controller: "FeedController",
            resolve: {
                posts: function (PostService) {
                    return PostService.list();
                }
            }
        })
        .when("/post/:id", {
            templateUrl: "static/js/app/views/view.html",
            controller: "PostController",
            resolve: {
                post: function ($route, PostService) {
                    var postId = $route.current.params.id
                    return PostService.get(postId);
                }
            }
        })
        .otherwise({
            redirectTo: '/'
        })
})
```

**(1)**: Since both Angular and Django use double braces, we need to set a new rule for Angular to separate between them. It’s not obligatory, but it is recommended. Especially if you want to get most of both frameworks.

We also add the dependency of ui.bootstrap  and ngCookies.

**(2)**:  Sets the CSRF token, this needs angular-cookies.js

**(3)**: routing for our app. We have two rules:

 1. “/” : pull the list of all posts
 2. “/post/:id” : the details of a particular post

You should notice that each view is delivered with controller. Also you can notice that we are using the resolve, to load data before changing the view. This have a better user experience, than changing the view and having to wait for the data to load.

Another thing to notice is that we don’t use routeParams,butinsteadweuseroute to access the params. Because $routeParams doesn’t get resolved until after the route is changed.

**Directives**

To make the user experience even better, we will create some directives.

```js
Blog.directive('timeAgo', function ($timeout) {
    return {
        restrict: 'A',
        scope: {
            title: '@'
        },
        link: function (scope, elem, attrs) {
            var updateTime = function () {
                if (attrs.title) {
                    elem.text(moment(attrs.title).fromNow());
                    $timeout(updateTime, 15000);
                }
            };
            scope.$watch(attrs.title, updateTime);
        }
    };
});

Blog.directive('pendingbar', ['$rootScope',
    function ($rootScope) {
        return {
            link: function (scope, element, attrs) {
                element.addClass('hide');
                $rootScope.$on('$routeChangeStart', function () {
                    element.removeClass('hide');
                });
                $rootScope.$on('$routeChangeSuccess', function () {
                    element.addClass('hide');
                });
                $rootScope.$on('$routeChangeError', function () {
                    element.removeClass('hide');
                });
            }
        };
    }]);

Blog.directive('viewstate', ['$rootScope',
    function ($rootScope) {
        return {
            link: function (scope, element, attrs) {
                element.addClass('hide');
                $rootScope.$on('$routeChangeStart', function () {
                    element.addClass('hide');
                });
                $rootScope.$on('$routeChangeSuccess', function () {
                    element.removeClass('hide');
                });
                $rootScope.$on('$routeChangeError', function () {
                    element.addClass('hide');
                });
            }
        };
    }]);
```

 * timeAgo: Formats a date as the time since creation. What is particular without this directive is that it updats itself with refreshing the page.
 * pendingBar : This is a pending bar that shows when the user try to switch to another view.
 * viewState : This hides the view until the routing is completed.

**Services**

Our Angular part is going to access the data from our API using servieces. ngResource enables interation with RESTful server-side data sources, but in this post we will be using http and q.

Angular services are singletons that carry out specific tasks common to web apps. Services are commonly used to perform the XHR interaction with the server.

```js
Blog.factory('GlobalService', function () {
    var vars = {
        is_authenticated: false
    }
	return vars;
});
```

Our first service will allow to communicate between controllers and hold global variables. You can put some informations from the request [for example the context].

```js
Blog.factory('PostService', function ($http, $q) {
    var api_url = "/posts/";
    return {
        get: function (post_id) {
            var url = api_url + post_id + "/";
            var defer = $q.defer();
            $http({method: 'GET', url: url}).
                success(function (data, status, headers, config) {
                    defer.resolve(data);
                })
                .error(function (data, status, headers, config) {
                    defer.reject(status);
                });
            return defer.promise;
        },
        list: function () {
            var defer = $q.defer();
            $http({method: 'GET', url: api_url}).
                success(function (data, status, headers, config) {
                    defer.resolve(data);
                }).error(function (data, status, headers, config) {
                    defer.reject(status);
                });
            return defer.promise;
        },
        update: function (post) {
            var url = api_url + post.id + "/";
            var defer = $q.defer();
            $http({method: 'PUT',
                url: url,
                data: post}).
                success(function (data, status, headers, config) {
                    defer.resolve(data);
                }).error(function (data, status, headers, config) {
                    defer.reject(status);
                });
            return defer.promise;
        },
        save: function (post) {
            var url = api_url;
            var defer = $q.defer();
            $http({method: 'POST',
                url: url,
                data: post}).
                success(function (data, status, headers, config) {
                    defer.resolve(data);
                }).error(function (data, status, headers, config) {
                    defer.reject(status);
                });
            return defer.promise;
        },
    }
});
```

The second service, is an object for accessing a Post object of our REST API.

**Controllers**

For this app, we create 3 controllers.

A controller to initialize our app. This is a global controller that we put at the beginning of our app.

```js
var appController = Blog.controller('AppController', function ($scope, $rootScope, $location, GlobalService) {
    var failureCb = function (status) {
        console.log(status);
    };
    $scope.globals = GlobalService;

    $scope.initialize = function (is_authenticated) {
        $scope.globals.is_authenticated = is_authenticated;
    };
})
```

```html
<body data-spy="scroll" id="index" style="zoom: 1;" ng-controller="AppController" ng-init="initialize('{{ user.is_authenticated }}')">
```

A controller to manage the list of posts, and create new ones

```js
Blog.controller('FeedController', function ($scope, GlobalService, PostService, posts) {
    $scope.posts = posts;
    $scope.globals = GlobalService;
    //options for modals
    $scope.opts = {
        backdropFade: true,
        dialogFade: true
    };
    //open modals
    $scope.open = function (action) {
        if (action === 'create'){
            $scope.postModalCreate = true;
            $scope.post = new Object();
        };
    };
    //close modals
    $scope.close = function (action) {
        if (action === 'create'){
            $scope.postModalCreate = false;
        };
    };
    //calling board service
    $scope.create = function () {
        PostService.save($scope.post).then(function (data) {
            $scope.post = data;
            $scope.posts.push(data);
            $scope.postModalCreate = false;
        }, function(status){
            console.log(status);
        });
    };
});
```

A controller to view and edit a single post

```js
Blog.controller('PostController', function ($scope, $routeParams, $location, PostService, GlobalService, post) {
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
        $scope.postName = $scope.post.name;
        $scope.postDescription = $scope.post.description;
        if (action === 'edit'){
            $scope.postModalEdit = true;
        };
    };
    //close modals
    $scope.close = function (action) {
        $scope.postName = "";
        $scope.postDescription = "";
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
});
```

**filters**

For aesthetic purposes, we create a simple filter to truncate the posts’ description.

```js
Blog.filter('truncate', function () {
        return function (text, length, end) {
            if (isNaN(length))
                length = 10;

            if (end === undefined)
                end = "...";

            if (text.length <= length || text.length - end.length <= length) {
                return text;
            }
            else {
                return String(text).substring(0, length-end.length) + end;
            }

        };
    });
```

**Views**

We need 2 views:

feed.html : List of posts and creation of new posts

```html
<br>
<div class="row">
    <div class="span9 main">
        <div class="row">
            <div class="span9">
                <a class="btn" ng-click="open('create')"> Create</a>

                <div modal="postModalCreate" close="close('create')" options="opts">
                    <a ng-click="close('create')" class="close" data-dismiss="modal">×</a>

                    <div class="modal-header">
                        <h4>Edit tip</h4>
                    </div>
                    <div class="modal-body">
                        <form class="form-horizontal">
                            <div class="control-group">
                                <label class="control-label" for="id_title">Title</label>

                                <div class="controls">
                                    <input type="text" id="id_title" placeholder="Title" ng-model="post.title">
                                </div>
                            </div>
                            <div class="control-group">
                                <label class="control-label" for="id_description">Description</label>

                                <div class="controls">
                                    <textarea type="text" id="id_description"
                                              ng-model="post.description"></textarea>
                                </div>
                            </div>
                        </form>
                    </div>
                    <div class="modal-footer">
                        <button class="btn btn-success" ng-click="create()">create</button>
                        <button class="btn btn-danger" ng-click="close('create')">Cancel</button>
                    </div>
                </div>
            </div>
        </div>
        <div class="row" ng-repeat="post in posts">
            <div class="span9">
                <div class="bg"></div>
                <div class="overlay">
                    <div class="content">
                        <a class="clip-open" href="{[{post.api_url}]}">
                            <h3>{[{post.title}]}</h3>
                        </a>

                        <div class="meta clips-note">
                            <p>
                                {[{post.description|truncate:150}]}
                            </p>
                        </div>
                        <small><abbr time-ago title="{[{post.created_on}]}" class="date"></abbr></small>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

view.html : View and update a  single post.

```html
<div class="row">
    <div class="span9 main">
        <div class="row">
            <div class="span9">
                <h3>{[{ post.title }]}</h3>
                <p>
                    {[{ post.description }]}
                </p>
            </div>
        </div>
        <div class="row">
            <div class="span9">
                <a class="btn" ng-click="open('edit')"> Edit</a>
                <a class="btn" href="#/"> Go back</a>
                <div modal="postModalEdit" close="close('create')" options="opts">
                    <a ng-click="close('edit')" class="close" data-dismiss="modal">×</a>

                    <div class="modal-header">
                        <h4>Edit tip</h4>
                    </div>
                    <div class="modal-body">
                        <form class="form-horizontal">
                            <div class="control-group">
                                <label class="control-label" for="id_title">Title</label>

                                <div class="controls">
                                    <input type="text" id="id_title" placeholder="Title" ng-model="post.title">
                                </div>
                            </div>
                            <div class="control-group">
                                <label class="control-label" for="id_description">Description</label>

                                <div class="controls">
                                    <textarea type="text" id="id_description"
                                              ng-model="post.description"></textarea>
                                </div>
                            </div>
                        </form>
                    </div>
                    <div class="modal-footer">
                        <button class="btn btn-success" ng-click="update()">Update</button>
                        <button class="btn btn-danger" ng-click="close('edit')">Cancel</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

In the next parts, we’ll add permissions, tags, sorting, searching, and pagination.
