---
layout: blog
title: "On infinity"
subtitle: "Proving that some infinities are bigger than other infinities."
cover_image: images/blog/math.png
cover_image_caption: ""
tags: [math]
---
In this post we will look closely at the notion of infinity. Particularly we will try proof that infinity comes in different sizes.

First of all we need to define a couple of notions, and we start we the notion of countable. Any set that is equipotent to $$ {\mathbb{N}} $$, i.e. there's a one-to-one correspondence with $$ {\mathbb{N}} $$ is called countable, personally I find this term a bit extreme, they probably should have called it listable. In other terms, given such set, one can count the elements of the set and eventually can reach any element.

Some example of countable sets are: $$ {\mathbb{Z}}, {\mathbb{N^2}}, {\mathbb{Z^2}}, {\mathbb{Q}} $$

$$ \begin{aligned}
{\mathbb{Z}} & = \bigcup\limits_{n \in \mathbb{N}} [\lvert  -n , n \rvert] \\
{\mathbb{N^2}} & = \bigcup\limits_{n \in \mathbb{N}} [\lvert  -n , n \rvert]^2  \\
  & = \bigcup\limits_{n \in \mathbb{N}} T_n \,\,\,\, \text{with} \,\,\, T_n =  \{ (p,q) \, | \, p+q=n \}
\end{aligned} $$

For $$ {\mathbb{Q}} $$ we can consider the application

$$ \begin{aligned}
f : {\mathbb{Z}} \times {\mathbb{N}}^* & \rightarrow {\mathbb{Q}} \\
(p, q) & \mapsto \frac{p}{q}
\end{aligned} $$

Which makes $$ {\mathbb{Q}} $$ equipotent to a subset of $$ {\mathbb{Z}} \times {\mathbb{N}}^* $$

Now we shall prove that $$ {\mathbb{R}} $$ is not countable, for that reason we will show that the set of reals in interval $$ [0, 1] $$  is not countable.

Let's consider a function $$ f : {\mathbb{N}} \rightarrow [0, 1] $$, and let $$ \left( I_n \right) $$ be a sequence of closed real intervals such that $$ I_{n+1} \subset I_n $$.

We know that one of these intervals $$ [0, \frac{1}{3}], [\frac{1}{3}, \frac{2}{3}], [\frac{2}{3}, 1] $$ does not contain $$ f(0) $$.

Let $$ I_0 = [a_0, b_0] $$ be such interval.

Again one of these intervals $$ [a_0, a_0 + \frac{b_0 - a_0}{3}], [a_0 + \frac{b_0 - a_0}{3}, a_0 + \frac{2 * \left( b_0 - a_0 \right)}{3}], [a_0 + \frac{2 * \left( b_0 - a_0 \right)}{3}, b_0] $$ does not contain $$ f(1) $$.

Let $$ I_1 = [a_1, b_1] $$ be such interval.

We suppose that we construct $$ I_0, I_1, ..., I_n $$ for a certain $$ n >= 1 $$ such that $$ \forall k \in [\lvert 1, n-1 \rvert], \,\, I_{k+1} \subset I_k $$

We will denote $$ I_n = [a_n, b_n] $$ and again one of these intervals $$ [a_n, a_n + \frac{b_n - a_n}{3}], [a_n + \frac{b_n - a_n}{3}, a_n + \frac{2 * \left( b_n - a_n \right)}{3}], [a_n + \frac{2 * \left( b_n - a_n \right)}{3}, b_n] $$ does not contain $$ f(n+1) $$

Let  $$ I_{n+1} = [a_{n+1}, b_{n+1}] $$ be such interval.

Hence by induction, we just construct a sequence of intervals $$ {\left( I_n \right)}_n $$.

Let $$ \bigcap\limits_{n \in \mathbb{N}} I_n = \{ a \} $$

We have the following $$ \forall n \in {\mathbb{N}}, a \in I_n, \,\, f(n) \notin I_n $$

It follows then that $$ \forall n \in {\mathbb{N}}, a \neq f(n) $$

Hence $$ f $$ is not surjective, and so it's not bijective.

In other terms we proved that there's no bijection from $$ {\mathbb{N}} $$ to $$ [0, 1] $$.

Therefor $$ [0, 1] $$ is not countable and so is $$ {\mathbb{R}} $$.
