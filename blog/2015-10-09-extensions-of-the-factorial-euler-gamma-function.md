---
layout: blog
title: "Euler Gamma function approximation for factorials"
subtitle: "Euler Gamma function."
cover_image: images/blog/math.png
cover_image_caption: ""
tags: [math, python]
---
In the previous post we introduced the Stirling formula, which is an accurate approximation for factorials even for small numbers. In this post we will look closely at an other extension of the concept of factorials, The gamma function. But unlike the factorial, the gamma function is more broadly defined for all complex numbers other than non-positive integers. In this post we will only study the behavior of this faction over $$ ]0, +\infty[ $$.

The gamma function is defined as follows

$$\Gamma (x) = \int_{0}^{+\infty} t^{x-1} e^{-t} \, dt $$

### 1- Domain of definition

The Gamma function is defined over $$ ]0, +\infty[ $$

Indeed, we have $$ \forall x \in {\mathbb{R}} $$ the function is continuous and positive over $$ ]0, +\infty[ $$.

$$\Gamma (x) \,\, \text{exists} \iff t \mapsto t^{x-1} e^{-t} \,\, \text{is integrable over} \,\, ]0, +\infty[ $$

 * near 0

$$ t^{x-1} e^{-t} \sim t^{x-1} $$

Therefore $$ t \mapsto t^{x-1} e^{-t} \,\, \text{is integrable over} \,\, ]0, 1[ \iff x-1 \gt 1 \,\, (x \gt 0) $$

 * near $$ + \infty $$

$$ t^{x-1} e^{-t} =o \frac{1}{ t^2 } $$

Therefore $$ t \mapsto t^{x-1} e^{-t} \,\, \text{is integrable over} \,\, [1, +\infty[ \,\, \forall x \in {\mathbb{R}} $$

We can deduce then that $$ t \mapsto t^{x-1} e^{-t} \,\, \text{is integrable over} \,\, ]0, +\infty[ \iff x \gt 0 $$

In other terms $$ D_{\Gamma} = ]0, + \infty[ $$

### 2- $$ \forall x \in ]0, + \infty[ $$ we have $$ \Gamma(x+1) = x \Gamma(x) $$

$$ \begin{aligned}
\Gamma(x+1) & = \int_{0}^{+\infty} t^{x} e^{-t} \, dt \\
 & = {[-t^x e^{-t}]}_{0}^{ + \infty } + x \int_{0}^{+\infty} t^{x-1} e^{-t} \, dt \\
 & = x \int_{0}^{+\infty} t^{x-1} e^{-t} \, dt \\
 & = x \Gamma (x)
\end{aligned} $$

By recursion it can be shown that

$$ \forall n \,\, , \,\, \Gamma (n+1) = n! $$

### 3- $$ \Gamma $$ is of $$ C^{+ \infty} $$ over $$ ]0, +\infty[ $$

Let's consider the function $$ f $$

$$ \begin{aligned}
f : ]0, +\infty[ \times ]0, +\infty[ & \rightarrow {\mathbb{R}} \\
(x, t) & \mapsto t^{x-1} e^{-t}
\end{aligned} $$


For all $$ n \in {\mathbb{N}} $$

 * We have $$ \forall (x, t) \in ]0, +\infty[ \times ]0, +\infty[ \, , \, \frac{\partial^{n} f}{\partial x^n} (x, t) = \left( ln (t) \right)^n t^{x-1} e^{-t} $$
 * $$ \frac{\partial^{n} f}{\partial x^n} $$ is continuous
 * Let $$ [a, b] \subset ]0, +\infty[ $$

 $$ \begin{aligned}
 \forall x \in [a,b] \, & , \, \forall t \in ]0, +\infty[ \\
 \lvert \frac{\partial^{n} f}{\partial x^n} (x, t) \rvert & = \lvert \left( ln (t) \right)^n t^{x-1} e^{-t} \rvert \\
  & \le \lvert \left( ln (t) \right)^n \rvert \left( t^{a - 1} + t^{b - 1} \right) e^{-t}
 \end{aligned} $$

And since the function $$ t \mapsto \lvert \left( ln (t) \right)^n \rvert \left( t^{a - 1} + t^{b - 1} \right) e^{-t} $$ is integrable over $$ ]0, +\infty[ $$.

Therefore $$ \forall n \in {\mathbb{N}} \, , \, \Gamma \,\, \text{is of class} C^{\infty} \,\, \text{over} ]0, +\infty[ $$

And $$ \forall n \in {\mathbb{N}} \, , \, \forall x \in ]0, +\infty[ $$

$$ \Gamma^{n} (x) = \int_{0}^{+\infty} \frac{\partial^{n} f}{\partial x^n} (x,t) \, dt = \int_{0}^{+\infty} \left( ln (t) \right)^n t^{x-1} e^{-t} \, dt $$

### 4- $$ \Gamma $$ is convex over $$ ]0, +\infty[ $$

Indeed $$ \forall x \in ]0, +\infty[ \,\, , \, \, \Gamma^{''} (x) = \int_{0}^{+\infty} \left( ln(t) \right)^2 t^{x-2} e^{-t} \,dt \ge 0 $$

Therefore $$ \Gamma $$ is convex.

### 5- $$ \exists ! x_0 \in ]1,2] $$ such that $$ \Gamma^{'} (x_0) = 0 $$

 * existence of $$ x_0 $$

We have $$ \Gamma (1) = \Gamma(2) = 1 $$

By applying Rolle's theorem, we find that $$ \exists x_0 \in ]1,2] \, , \, \text{such that} \,\, \Gamma^{'} (x_0) = 0 $$

 *  uniqueness of $$ x_0 $$

Since $$ \Gamma^{''} > 0 \,\, \text{over} \,\, ]0, +\infty[ $$ then $$ \Gamma{'} $$ is strictly increasing. Therefore $$ x_0 $$ is unique.

It follows also that:

$$
\begin{cases}
\text{Over} \,\, ]0, x_{0}] \,\, \Gamma^{'} \le 0 \,\, \left( \Gamma \, \text{is decreasing} \right) \\
\text{Over} \,\, [x_{0}, +\infty[ \,\, \Gamma^{'} \,\, \ge 0 \left( \Gamma \, \text{is increasing} \right)
\end{cases}
$$

### 6- $$ \Gamma $$ near $$ 0 $$ and $$ + \infty $$

 * near $$ 0 $$

We have $$ \forall x \gt 0 \, , \, \Gamma (x+1) = x \Gamma(x) $$

Therefore $$ \lim \limits_{x \to 0} x \Gamma(x) = \lim \limits_{x \to 0} \Gamma(x+1) = \Gamma(1) = 1 $$

Finally $$ \Gamma(x) \sim \frac{1}{x} $$

 * near $$ \infty $$

We have

$$ \begin{aligned}
\frac{ \Gamma(x) }{x} & \ge \frac{ \Gamma ([x]) }{x} \,\, \left( \Gamma \,\, \text{is increasing} \right) \\
 & \ge \frac{ ([x] - 1)! }{x} = \require{cancel} \cancelto{ + \infty }{ \frac{ ([x] - 1)! }{ ([x] - 1) } } . \require{cancel} \cancelto{1}{ \frac{ ([x] - 1) }{x} }
\end{aligned} $$

Finally $$ \lim \limits_{x \to +\infty} \frac{ \Gamma(x) }{x} = +\infty $$

We can verify our results with some python code:

```python
from scipy import gamma

gamma(0)
# inf

gamma(inf)
# inf
```

Gamma function is considered as the "correct" substitute for the factorial in various integrals, which seems to come more or less from its integral definition.

![euler-gamma-function](/images/blog/gamma.png)
