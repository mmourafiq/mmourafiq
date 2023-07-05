---
layout: blog
title: "_, __ and __fct__ Python"
subtitle: "Why does python use so much underscores?"
cover_image: images/blog/python-power.jpg
cover_image_caption: "Python."
tags: [ python ]
---

I was asked this question about 2 days ago, why does python use so much underscores?

So here is a small documentation about different uses of underscores.

* One underscore at the beginning:

Variables with a leading underscore are read only.

Functions with a leading underscore are not meant to be accessed out of the class. This function can
be changed without notice.

from the python docs :

name prefixed with an underscore (e.g. _spam) should be treated as a non-public part of the API (
whether it is a function, a method or a data member). It should be considered an implementation
detail and subject to change without notice.

* Two underscores at the beginning:

Variables with at least two leading underscores, at most one trailing underscore, used to define
class-private instance and class variables, methods, variables stored in globals, and even variables
stored in instances. private to this class on instances of other classes.

* Two underscores at the beginning and Two underscores in the end:

Two leading and Two trailing underscore is a convention, a way for the Python system to use names
that won’t conflict with user names.

There is always an operator or native function that calls these magic methods. The idea here is to
give you the ability to override operators in your own classes. Sometimes it’s just a hook python
calls in specific situations. `__init__()`, for example, is called when the object is created so you
can initialize it. `__new__()` is called to build the instance, and so on…
