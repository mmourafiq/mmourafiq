---
layout: blog
title: "Prime Factors of n!"
subtitle: "Algorithm for computing the prime factorization of the integer n!"
cover_image: images/blog/math.png
cover_image_caption: "math."
tags: [ python, math ]
---


The purpose of this post is to explain an algorithm for computing the prime factorization of the
integer $$n!$$. We covered in a previous post
the [Unique Factorization Theorem](/2012/04/01/unique-factorization.html) stating that every integer
strictly positive could be written as a product of prime numbers. Every prime factor of n is also a
prime factor of n!, but usually with a higher multiplicity. Except for the trivial case $$n = 2$$,
$$n!$$ always has additional prime factors that $$n$$ doesn’t have.

Finding prime factors of $$n!$$ has a great impact on how to calculate the exact values for
permutations and combinations, specially when it comes to big numbers.

We shall prove that, for every positive integer n, we have:

$$
\begin{align*}
n! = p_1^{r_1} * p_2^{r_2}* ... * p_i^{r_i}
\end{align*}
$$

* **Theorem1**:

For every positive integer n. Let p be any prime where p <= n. If p is a prime factor of n! then its
multiplicity is given by:

$$
\begin{align*}
\sum_k IntPart(n/p^k)\ ,\ where\ IntPart\ is\ integer\ the\ part.
\end{align*}
$$

* **Proof**:

For each $$k$$ we need to compute how many times $$p^k$$ appears in the list of numbers between 1
and n. While $$p^k <= n$$, each such appearance contributes

$$
\begin{align*}
IntPart(n/p^k)
\end{align*}
$$

to the final power of $$p$$ that is part of the factorization of $$n!$$

* **Theorem2**:

For every positive integer $$n>1$$ 
let $$p$$ 
be such that $$1 < p <= n$$
let $$k > 1$$ denote any
positive integer where $$p^{k+1} <= 1$$ we have then:

$$IntPart(n/p^{k+1}) = IntPart(\frac{IntPart(n/p^k)}{p})$$

* **Proof**:

First note that every multiple of $$p^{k+1}$$ is also a multiple of $$p^k$$ . The number of
multiples of $$p^{k+1}$$ is less than or equal to n, if anything, is smaller than the number of
multiples of $$p^k$$ that are less than or equal to $$n$$. The last multiple of $$p^k$$ that is less
than or equal to $$n$$, if anything, is larger than the last multiple of $$p^{k+1}$$ that is less
than or equal to $n$.

If $$p^{k+1} > n$$, this theorem is still true, but then it says 0 = 0. So in practice we assume
$$p^{k+1} <= n$$.

Let $$r = IntPart(n/p^k) * p^k$$ and $$s = IntPart()n/p^{k+1}$$ Then $$r>=s$$.
There exits an integer $$j$$ such that $$s+j=r$$, where $$0 <= j <= p^{k+1}$$. That $$j < p^{k+1}$$
is because s is the last multiple of $$p^{k+1}$$ that is less or equal to $$n$$. Which implies that
$$\frac{j}{p^{k+1}}<1$$

Finally, $$r = s + j$$, $$IntPart(n/p^k) * p^k = IntPart(n/p^{k+1}) * p^{k+1} + j$$

Now we divide by $$p^{k+1}$$ in both sides of the equation.

$$
\begin{align*}
IntPart(\frac{IntPart(n/p^k)}{p}) = IntPart(n/p^{k+1})
\end{align*}
$$
