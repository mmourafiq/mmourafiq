---
layout: blog
title: "Infinity of prime numbers."
subtitle: "Proving the infinity of prime numbers, noted as P."
cover_image: images/blog/blueprint-math.jpg
cover_image_caption: "Math"
tags: [math, probability, python]
---

In this post we will prove the infinity of prime numbers, noted as $$P$$.

First, what is a prime number? a prime number is a natural number greater than 1 that has no positive divisors other than 1 and itself, e.g. 2, 3, 5, 71, 97 ...

Throughout history, multiple mathematicians studied this branch of mathematics, particularly Euclid, Gauss, and Riemann. Euclid proved this mathematical property by multiplying all primes from 1 to n and adding 1.

$$\prod_{i=1}^{n} P_{i} + 1$$

Hence the property seems obvious. In this post we will try Gauss's method. In order to do that, we need these notions:  

 * harmonic series
 * fundamental theorem of arithmetic or the [unique factorization theorem](/2012/04/01/unique-factorization.html).

First let’s suppose that, the set of prime numbers is finite, which means that we can enumerate the elements of this set:

$$P = {P_1, P_2, ..., P_r}$$ with $$r = card(P)$$

And we consider this series: $$S_n = \sum_{i=1}^{k} 1/k$$

If we apply the fundamental theorem of arithmetic, we find

$$S_n = \sum_{i=1}^{k} \frac{1}{\prod_{i=1}^{r}P_i ^{\alpha_i k}}$$

Now, we will try to find a max for this series $$S_n$$. For this purpose let's simplify our formula

$$S_n = \frac{1}{P_1} + \frac{1}{P_1*P_2} ... \frac{1}{\prod_{i=1}^{r}P_i ^{\alpha_i k}}$$

And let's consider also this value:

$$t = (1 + \frac{1}{P_1} + ... + \frac{1}{P_1^{\alpha_1}}) * ... * (1 + \frac{1}{P_r} + ... + \frac{1}{P_r^{\alpha_r}})$$

with $$\alpha_i$ the max of exponent of $P_i$$

$$\alpha_{i} = max(\alpha_{i,k})_{k=[1,n]}$$

If we take one of this  $$\alpha_i$$ and call it $$\beta$$, we then have

$$\frac{1}{P_1^{\beta_{1}}} * \frac{1}{P_2^{\beta_{2}}} * ... * \frac{1}{P_r^{\beta_{r}}}$$

We can then conclude that $$S_n < t$$ And since terms in $$t$$ are geometric sequences. We find then:

$$S_{n} < \prod_{i=1}^{r} \frac{1 - \frac{1}{P_i ^{\alpha_i + 1}}}{1 - \frac{1}{P_{i}}} < \prod_{i=1}^{r} \frac{1}{1-P_{i}}$$

and this quantity is finite, However since harmonic series diverge, we find that: $$+\infty < X$$

Which is absurd, hence we can say that prime numbers are infinite.
