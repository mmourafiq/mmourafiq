---
layout: blog
title: "Unique Factorization"
subtitle: "A simlpe proof for the unique factorization theorem."
cover_image: images/blog/math.png
cover_image_caption: "math."
tags: [ math ]
---


Today’s post we are going to see an important theorem in arithmetic, “The unique factorization”.
This theorem is used in various application and it states that : every integer strictly positive is
either prime itself or is the product of a set of prime factors in an unique way, the order of its
prime factors.

Proof by induction :

Let n be an integer strictly positive.

* **Existence**:

For n = 1, 1 is the product of a set of prime numbers (the empty set)

We shall assume that the result hold for every integer <= n, and we will prove it for n+1.

Either n+1 is prime, and we are done.

Or n+1 = l * k, where l, k < n. From the induction assumption we can imply that both k and l could
be written as the product of prime factors. And therefor, their product too. Making n+1 the product
of a set of prime factors.

* **Uniqueness**:

Suppose that n is the products of prime factors in two ways:

- n = P1*P2* … *Pu
- n = Q1*Q2* … *Qv

We have to prove that u = v, and that ∀ i, ∃ j such Pi = Qj.

We shall use the Euclid’s lemma that states, if p prime number divides a product x*y, p must divide
x or/and y.

Let i, 1 <= i <= u. We know that Pi divides n, hence Pi divides one of Qjs.

Let j such index, since Qj is prime we conclude that Pi is either 1 or Qj. Pi is prime means that
Pi = Qj. We will rearrange Pis and Qjs in a way that makes P1 = Q1.

we have now :

- n/P1 = P2*P3* … *Pu
- n/P1 = Q2*Q3* … *Qv

Following the same reasoning, we will find that :

- n/(P1*P2* … *Pu) = 1
- n/(P1*P2* … *Pu) = Qu+1 * … * Qv

Which means Qu+1 * … * Qv = 1, which is impossible. Therefor, u = v and every Pi = Qi.
