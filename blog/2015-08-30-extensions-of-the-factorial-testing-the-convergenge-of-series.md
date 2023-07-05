---
layout: blog
title: "Stirling approximation for factorials"
subtitle: "[Part 1] Testing the convergence of series."
cover_image: images/blog/math.png
cover_image_caption: ""
tags: [math, python]
---
When evaluating distribution functions for statistics, it is often necessary to evaluate the factorials of sizable numbers. Stirling formula is an approximation for factorials that leads to a very accurate results of the factorials. In this series of posts, we will try to get a mathematical intuition of this formula.

In this first part we will try to have a closer look at the behavior of infinite series and some of their properties. The motivation behind is to have a better intuition about some very interesting concepts and tools used in everyday mathematics, physics and computer science.

## Definition

We call a series associated with sequence $$ {\left(U_n\right)}_{n \in {\mathbb{N}} } $$ and noted $$ \sum U_n $$ or $$ \sum_{n \in {\mathbb{N}}} U_n $$, the couple:

$$ \left( \left( U_n \right)_{n \in {\mathbb{N}}} , \left( S_n \right)_{n \in {\mathbb{N}} } \right) \in  {\mathbb{E}}^{\mathbb{N}} \times {\mathbb{E}}^{\mathbb{N}} $$

where $$ \forall n \in {\mathbb{N}} , S_n = \sum_{k=0}^{n} U_k $$

$$ S_n $$ is called the partial sum of the series $$ \sum U_n $$

## Convergence and divergence

Let $$ \left( {\mathbb{E}}, \|.\| \right) $$ be normed vector space, $$ \sum U_n $$ a series defined in $$ {\mathbb{E}} $$ and $$ {\left( S_n \right)}_n $$ the sequence of partial sums.

 * We say that the series $$ \sum U_n $$ converges (respectively diverges) in $$ \left( {\mathbb{E}}, \|.\| \right) $$ if the sequence $$ {\left( S_n \right)}_n $$ converges (respectively diverges) in $$ \left( {\mathbb{E}}, \|.\| \right) $$.
 * In this case, the limit $$ S $$ of the sequence $$ {\left( S_n \right)}_n $$ is called the sum of the series $$ \sum U_n $$, and is noted: $$ \sum_{n=0}^{\infty} U_n = S = \lim \limits_{n \to \infty} S_n $$

 **Examples**

  * the geometric series for complex numbers.

$$ \sum{q^n} , q \in {\mathbb{C}} $$

We have the following, $$ \forall n \in {\mathbb{N}} $$

$$
S_n = \sum_{k=0}^{n} q^n =
\begin{cases}
n + 1,  & \text{if $q = 1$} \\
\frac{1 - q^{n+1}}{1 - q}, & \text{if $q \neq 1$}
\end{cases}
$$

Meaning that the series converges iff $$ \|q\| \lt 1 $$

and in this case $$ \sum_{n = 0}^{+\infty} q^n = \frac{1}{1-q} $$

The result is simple yet so impressive, a lot of philosophers struggled to resolve [Zeno’s paradoxes](https://en.wikipedia.org/wiki/Zeno%27s_paradoxes) because they lacked the mathematical tools to correctly model the problem.

 * another example $$ \sum_{n \gt 0} \frac{1}{n(n+1)} , \forall n \in {\mathbb{ N^* }} $$

$$ S_n = \sum_{k = 1}^{n} (\frac{1}{k} - \frac{1}{k+1}) = 1 - \frac{1}{n+1} \require{AMScd} \begin{CD} @>>{n \to \infty}>\end{CD}  1 $$

Hence the series $$ \sum\frac{1}{n(n+1)} $$ converges, and the infinite sum is 1.

## Convergence tests

### 1- The n-th term test (convergence’s necessary condition)

Let $$ \left( {\mathbb{E}}, \|.\| \right) $$ be normed vector space, $$ \sum U_n $$ a series defined in $$ {\mathbb{E}} $$

$$ \sum U_n \text{converges} \Rightarrow \lim \limits_{n \to \infty} U_n = 0 $$

**Proof**

Let $$ S_n = \sum_{k = 0}^{n} $$ be the partial sums of the series, we know that the series converges if the $$ \lim \limits_{n \to \infty = S} $$

Hence, we have $$ U_n = S_n - S_{n-1} \require{AMScd} \begin{CD} @>>{n \to \infty}>\end{CD} 0 $$

**N.B. 1**

A direct conclusion from this test is that, if the general term $$ U_n $$ does not converge to 0, then the series diverges.

Again if we look at the geometric series which converges only if $$ \|q\| \lt 1 $$, we can see that if $$ \|q\| >= 1, \lim \limits_{n \to \infty} q^n > 0 $$

**N.B. 2**

having the general term $$ U_n $$ converging to 0, does not guarantee the convergence of the series.

Let’s study the series $$ \sum log \left( \frac{n+1}{n} \right) $$

We have the following $$ \lim \limits_{n \to \infty} log \left( \frac{n+1}{n} \right) = 0 $$

$$ \begin{aligned}
\sum_{k = 0}^{n} log \left( \frac{k+1}{k} \right) & = \sum_{k = 0}^{n} \left( log(k+1) - log(k) \right) \\
 & = log(n+1) \require{AMScd} \begin{CD} @>>{n \to \infty}>\end{CD} +\infty
\end{aligned} $$

Hence the series diverges.

### 2- Cauchy’s condition

Let $$ \left( {\mathbb{E}}, \|.\| \right) $$ be normed vector space, $$ \sum U_n $$ a series defined in $$ {\mathbb{E}} $$

$$ \begin{aligned}
\sum U_n   \text{converges} \Rightarrow & \forall \epsilon \gt 0, \, \exists N \in {\mathbb{N}}, \,\, \forall n,p \in {\mathbb{N}} \\
& n \ge N \Rightarrow \| \sum_{k = n + 1}^{n+p} U_k \| \le \epsilon
\end{aligned} $$

**Proof**

The proof is quite simple if we consider the sequence of partial sums $$ {(S_n)}_{n} $$.
The series converges implies that $$ (S_n)_{n} $$ converges, which means that it’s a Cauchy sequence.

$$ \begin{aligned}
& \forall \epsilon, \,\, \exists N \in {\mathbb{N}}, \,\, \forall n,p \in {\mathbb{N}} \\
& n \ge N \Rightarrow \| S_{n+p} - S_n \| \le \epsilon \\
& \text{And we have} \,\, \| S_{n+p} - S_n \| = \| \sum_{k = n + 1}^{n+p} U_k \|
\end{aligned} $$

**Example**

Let’s consider the harmonic series $$ \sum \frac{1}{n} $$

$$ \forall n \in {\mathbb{ N^* }} \, \sum_{k=n+1}^{2n} \ge n * \frac{1}{2n} \ge \frac{1}{2} $$

Hence the series does not verify the Cauchy’s condition, and so it does not converge.

**N.B.**

This condition is important, and most of the time is used to study series in Banach spaces.

Let $$ {\mathbb{E}} $$ be a Banach space:

 * A series converges $$ iff $$ it verifies the Cauchy’s condition.
 * A series converges absolutely $$ \Rightarrow $$ it converges.

In the second property the convergence does not imply the absolute convergence, as an example let’s consider the following series $$ \sum \frac{ (-1)^{n+1} }{n} $$

$$ \begin{aligned}
 S_n & = \sum_{k=1}^{n} \frac{-1^{k+1}}{k} \\
  & = \sum_{k=1}^{n} \left(-1\right)^{k+1} \int_{0}^{1} x^{k-1} dx \\
  & = \int_{0}^{1} \sum_{k=1}^{n} (-x)^{k-1} dx \\
  & = \int_{0}^{1} \frac{ 1 - (-x)^n }{ 1 + x } dx \\
  & = \int_{0}^{1} \frac{ dx }{ 1 + x } - \frac{ \left(-x\right)^n }{ 1 + x } dx \\
  & = ln(2) - \int_{0}^{1} \frac{ \left(-x\right)^n }{ 1 + x } dx
\end{aligned} $$

And since $$ \lvert \int_{0}^{1} \frac{ (-x)^n }{ 1 + x } dx \lvert \le \int_{0}^{1} (-x)^n = \frac{ 1 }{ n + 1 } \require{AMScd} \begin{CD} @>>{n \to \infty}>\end{CD} 0 $$

It follows that $$ \lim \limits_{ n \to \infty } S_n = ln(2) $$

Hence the series converges to $$ ln(2) $$ but it does not converge absolutely.

Let's check this result in python, we will use the symbolic library `sympy`

```python
from sympy import symbols, Sum, oo

k, n = symbols('k n', integer=True)
ps = Sum((-1)**(k+1)/k, (k, 1, n))

#  lets calculate the partial sum
ps.doit()
> (-1)**(n + 1)*lerchphi(exp_polar(I*pi), 1, n + 1) + log(2)

#  the infinite sum
s = ps.subs(n, oo)
s.doit()
> log(2)
s.evalf()
> 0.693147180559945
```

**N.B.**

A series that converges but does not converge absolutely, is called semi-convergent or conditionally convergent.

### 3- Leibnitz rule for testing the convergence of alternating series

Let $$ \sum U_n $$ be a series over reals, if $$ \sum U_{n} $$ is an alternating series, i.e. $$ U_n = (-1)^{n} \lvert U_n \lvert $$ or $$ U_n = (-1)^{ n + 1 } \lvert U_n \lvert $$, and $$ (\lvert U_n \lvert)_n $$ is decreasing and converges to 0, then the series $$ \sum U_n $$ converges and $$ R_n = S - S_n \lt \lvert U_{ n + 1} \lvert $$

**Proof**

Let’s suppose that $$ \forall n \in {\mathbb{ N }} U_n = (-1)^n \lvert U_n \lvert $$ and $$ \forall n \in {\mathbb{ N }} \,\, S_n = \sum_{k=0}^{n} U_k $$

$$ \begin{aligned}
\text{ We have } \,\, S_{ 2(n+1) } - S_{2n} & = U_{ 2n + 1 } + U_{ 2n + 2 } \\
  & = -\lvert U_{ 2n + 1 } \lvert + \lvert U_{ 2n + 2 } \lvert
\end{aligned} $$

Hence $$ {(S_{2n})}_n $$ is decreasing. The same way we can show that $$ {(S_{2n + 1})}_n $$ is increasing. And since $$ {(S_{2n + 1})}_n - {(S_{2n})}_n = U_{2n + 1} \require{AMScd} \begin{CD} @>>{n \to \infty}>\end{CD} 0 $$, both $$ {(S_{2n + 1})}_n $$ and $$ {(S_{2n})}_n $$ converge to the same limit $$ S $$, then we find that $$ \lim \limits_{n \to \infty} S_n = S $$.

Also, since

$$ \begin{aligned}
R_n & = S - S_n \\
\lvert R_{2n} \lvert & = \lvert S - S_{2n} \lvert \le \lvert S_{2n} - S_{2n + 1} \lvert = \lvert U_{2n + 1} \lvert \\
\lvert R_{2n + 1} \lvert & = \lvert S - S_{2n + 1} \lvert \le \lvert S_{2n + 2} - S_{2n + 1} \lvert = \lvert U_{2n + 2} \lvert \\
\forall n \in {\mathbb{ N }} \,\, \lvert R_n \lvert & \le \lvert U_{n+1} \lvert
\end{aligned} $$

**Example**

$$ \begin{aligned}
& \sum \frac{(-1)^n}{n}  \, \text{converges} \\
\forall \alpha > 0 , \,\, & \sum \frac{(-1)^n}{n^\alpha} \, \text{converges}
\end{aligned} $$

### 4- Comparison test

$$ \sum U_n \,\, \text{converges} \, \iff \, \exists c \, \text{constant} \forall n \, S_n \le c $$

**Proof**

We know that $$ S_{n + 1} - S_n = U_{n + 1} >= 0 $$ and that the sequence $$ {(S_n)}_n $$ is increasing.

$$ {(S_n)}_n \, \text{converges} \, \iff \, \exists c \, \text{constant} \forall n \, S_n \le c $$

Based on this property we can deduce many other properties related to comparing series in different ways, i.e. using $$ \sim , \, o , \, O , \, ... $$

**Example: Riemann series**

We cal a Riemann series, a series of the form $$ \sum \frac{1}{ n^\alpha } , \, \alpha \in {\mathbb{R}} $$

 * Case when $$ \alpha \le 0 $$

The series $$ \sum \frac{1}{ n^\alpha } $$ obviously diverges.

 * Case when $$ \alpha = 1 $$

We saw that $$ \sum \frac{1}{n} $$ diverges.

 * Case when $$ \alpha \in [ 0, 1 ] $$

The series $$ \sum \frac{1}{ n^\alpha } $$ diverges, because $$ \frac{1}{n} =o \frac{1}{ n^\alpha } $$

 * Case when $$ \alpha > 1 $$

Let’s consider the application $$ f $$

$$ \begin{aligned}
f : [ 0, +\infty [ & \rightarrow {\mathbb{R}} \\
x & \mapsto \frac{ x^{1 -\alpha } }{ 1 - \alpha }
\end{aligned} $$

The application $$ f $$ is of class $$ C^1 $$, and by applying the Mean value theorem over $$ [n, n+1] $$, we have the following result $$ \exists \theta_n \in [n, n+1] $$ such that $$ f\left( n+1 \right) - f\left( n \right) = f^{''}\left( \theta_n \right) = \frac{ 1 }{ { \theta_n }^\alpha } $$

And since $$ \theta_n \sim n $$

It follows that $$ \frac{1}{ { \theta_n }^\alpha } \sim \frac{1}{ n^\alpha } $$

Which means that $$ \frac{1}{ n^\alpha } \sim \left( f\left( n+1 \right) - f\left( n \right) \right) $$

We know that the series $$ \sum \left( f\left( n+1 \right) - f\left( n \right) \right) $$ converges, because $$ \sum_{ k = 1 }^{ n } \left( f\left( n+1 \right) - f\left( n \right) \right) = f\left( n+1 \right) - f\left( 1 \right) \require{AMScd} \begin{CD} @>>{n \to \infty}>\end{CD} -f \left( 1 \right) $$

Finally $$ \sum \frac{1}{n^\alpha} $$ converges if $$ \alpha > 1 $$.


```python
from sympy import symbols, Sum, oo

a = symbols('a')
n, k  = symbols('n k', integer=True)
ps = Sum(1/(k**a), (k, 1, n))
s = ps.subs(n, oo)

#  the infinite sum for a < 0
s.subs(a, -2).doit()
> oo

#  the infinite sum for a == 1
s.subs(a, 1).doit()
> oo

#  the infinite sum for a > 1
s.subs(a, 2).doit()
> pi**2/6

s.subs(a, 3).doit()
> zeta(3)

s.subs(a, 4).doit()
> pi**4/90

s.subs(a, 5).doit()
> zeta(5)

```
