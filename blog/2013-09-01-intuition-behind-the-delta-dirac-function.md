---
layout: blog
title: "Intuition behind the Delta Dirac function."
subtitle: "Particular function that is zero everywhere except at zero where it approaches infinity."
cover_image: images/blog/math.png
cover_image_caption: "Math"
tags: [math, probability, python]
---

Today we are going to look closely at a very interesting function, called the [delta Dirac function](http://en.wikipedia.org/wiki/Dirac_delta_function). This function has a particularity that is zero everywhere except at zero where it approaches infinity. This function was introduced by the theoretical physicist [Paul Dirac](http://en.wikipedia.org/wiki/Paul_Dirac). In the context of [signal processing](http://en.wikipedia.org/wiki/Signal_processing) it is often referred to as the unit impulse symbol. Its discrete analog is the [Kronecker delta](http://en.wikipedia.org/wiki/Kronecker_delta) function which is usually defined on a finite domain and takes values 0 and 1.

The Dirac delta can be loosely thought of as a function on the real line which is zero everywhere except at the origin, where it is infinite,

$$\delta (x) = \left\{
  \begin{array}{l l}
 \infty & \text{ if } x = 0  &,\\
 0 & \text{ if } x \neq 0
\end{array} \right.$$

and which is also constrained to satisfy the identity

$$\int_{-\infty}^{\infty} \delta(x) dx = 1$$

In this post we will try to think about the intuition behind this definition. Because in mathematics, we can't define a function with theses properties on real numbers.

First, why to have such a function? Let's think about a system where nothing happens during a long period of time, and then there is an impulse that hits hard but vanish quickly, and again nothing happens during a long period of time. So we need a function that can model this kind of behavior, a function that can mimic this behavior where nothing happens except for t = 0.

So how can a function defined this way have an integral of 1 over the entire real numbers?

For this purpose, we are going to define another function that makes a better intuition of how this delta Dirac function can be constructed.

Let's consider $$\delta_{\tau}$$ define as the following:

$$\delta_{\tau} = \left\{
  \begin{array}{l l}
 \frac{1}{2 \tau} & \text{ if } -\tau <x< \tau  &,\\
 0 & \text{ otherwise }
\end{array} \right.$$

This function has the particularity to look more like a real function since we can calculate the integral for this function. In the same time, this can turn into the delta Dirac function when when we narrow the interval $$[-\tau, \tau]$$, i.e. when $$\tau \to 0$$

Let's calculate the integral of this new function:

$$\int_{-\infty}^{\infty} \delta_{\tau}(x) dx = \int_{-\tau}^{\tau} \frac{1}{2\tau} dx = 1$$

And since, we said that :

$$\lim_{\tau \to 0} \delta_{\tau}(x) = \delta(x)$$

We can do something like :

$$\lim_{\tau \to 0} \int_{-\infty}^{\infty} \delta_{\tau}(x) dx = 1$$

And intuitively, we can say that :

$$ \int_{-\infty}^{\infty} \delta(x) dx = 1$$
