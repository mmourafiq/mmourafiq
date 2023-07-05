---
layout: blog
title: "Stirling approximation for factorials"
subtitle: "[Part 2] Deriving the Stirling formula."
cover_image: images/blog/math.png
cover_image_caption: ""
tags: [math, python]
---
In the previous post, we introduced some properties of infinite series and how to test their convergence. In this post we will try to derive the Stirling formula. But before we do that, we will first introduce 2 more properties for testing series convergence, and the Wallis formula, which will be used for the proof.

## Limit Comparison test

Let $$ \sum U_n $$ and $$ \sum V_n $$ two series, where $$ \forall n \in {\mathbb{N}}, \, \, U_n > 0 \, \text{and} \, V_n > 0 $$.

If $$ \exists N \gt 0 , \, \frac{U_{ n+1 }}{U_n} \le \frac{V_{ n+1 }}{V_n} $$

Then $$ \sum V_n \, \text{converges} \Rightarrow \sum U_n \, \text{converges} $$

**Proof**

Let $$ N \in {\mathbb{N}} $$ such that $$ \forall n \gt N , \,\,  \frac{U_{ n+1 }}{U_n} \le \frac{V_{ n+1 }}{V_n} $$

It follows then $$ \forall n \gt N $$

$$ \frac{U_{ n+1 }}{V_{ n+1 }} \le \frac{U_n}{V_n} \le ... \le \frac{U_N}{V_N} = M $$

Whihch means that $$ U_{n+1} \le M V_{n+1} $$

Hence $$ \sum V_n \, \text{converges} \Rightarrow \sum U_n \, \text{converges} $$

**N.B**

From this property we can derive the following, if $$ \sum U_n $$ is a series and $$ \exists k \in ]0, 1[ $$ such that

If $$ \exists N \in {\mathbb{N}} $$ and $$ \forall n \ge N , \,\, \frac{U_{ n+1 }}{n} \le k $$

Then $$ \sum U_n $$ converges.

The proof is quite simple if we consider the series $$ \sum k^n $$, we have $$ \frac{U_{ n+1 }}{U_n} \le \frac{k_{ n+1 }}{k_n} $$

## The ratio rule, D'Alembert's rule

Let $$ \sum U_n $$ a series with the general term strictly positive, and $$ \lim \limits_{n \to +\infty} \frac{U_{n+1}}{U_n} = l $$, with $$ l \in {\mathbb{R_+}} \cup \{ \infty \}$$

 * i) if $$ l \lt 1 $$ the series $$ \sum U_n $$ converges
 * ii) if $$ l \gt 1 $$ the series $$ \sum U_n $$ diverges

**Proof**

 * i) Let $$ \epsilon = \frac{1 - l}{2} $$

 We know that $$ \lim \limits_{n \to +\infty} \frac{U_{n+1}}{U_n} = l $$

 Then $$ \exists N \in {\mathbb{N}}, \,\, \text{such that} \,\, n \ge N \Rightarrow \vert U_{n+1} - l \vert \le \epsilon $$

 Which means that $$ \forall n \ge N, \,\, \frac{U_{n+1}}{U_n} \le l + \epsilon $$

 And since $$ l + \epsilon \lt 1 $$, we can conclude the convergence using the previous property.

  * ii) $$ \lim \limits_{n \to +\infty} \frac{U_{n+1}}{U_n} = l $$ and $$ l \lt 1 $$

This means that there's a certain integer after which $$ \frac{U_{n+1}}{U_n} \lt 1 $$, which means $$ U_n $$ diverges.

**N.B**

If $$ l = 1 $$, we can't say anything about the convergence of the series.

As an example, let's consider $$ \sum \frac{1}{n^{\alpha}} $$

We have $$ \forall \alpha \in {\mathbb{R}}, \,\, \frac{U_{n+1}}{U_n} = \left( \frac{n}{n+1} \right)^{\alpha} \require{AMScd} \begin{CD} @>>{n \to \infty}>\end{CD} 1 $$

And we know that the series converges for $$ \alpha \gt 1 $$ and diverges for $$ \alpha \le 1 $$.

## Wallis formula

Let's consider, Wallis integral, $$ \forall n \in {\mathbb{ N }} , \,\, I_n = \int_{0}^{ \pi / 2 } {\left( \sin t \right)}^n dt $$

This integral verifies the following properties:

 * strictly positive and decreasing
 * for $$ n \le 2, \,\, I_n = \frac{n}{n-1} I_{n-1} $$

We will try these properties by induction

 * case $$ n = 0 $$ , $$ I_0 = \pi / 2 \gt 0 $$
 * case $$ n = 1 $$ , $$ I_1 = 1 \gt 0, \,\, I_1 \lt I_0 $$
 * case $$ n = 2 $$ , $$ I_2 = \frac{\pi}{4} \gt 0, \,\, I_2 \lt I_1 $$

$$ \begin{aligned}
I_2 & = \int_{0}^{ \pi / 2 } \sin^2 t \, dt \\
 & = \require{cancel} \cancelto{𝟶}{-[\cos t \, \sin t]_{0}^{ \pi / 2 }} + \int_{0}^{ \pi / 2 } \cos^2 t \, dt \\
 & = \int_{0}^{ \pi / 2 } 1 - \sin^2 t \, dt \\
 & = \pi / 2 - I_2 \\
 & = \frac{\pi}{4}
\end{aligned} $$

 * We will assume that $$ \forall k \in {\mathbb{N}}, \,\, k \le n, \,\, I_k $$ is strictly positive and decreasing, and we will prove that the property holds for $$ n+1 $$

 $$ \begin{aligned}
 I_{n+1} & = \int_{0}^{ \pi / 2 } \sin^{n+1} t \, dt \\
  & =  \int_{0}^{ \pi / 2 } \sin t \, \sin^n t \, dt \\
  & = \require{cancel} \cancelto{𝟶}{-[\cos t \, \sin^n t]_{0}^{ \pi / 2 }} + \int_{0}^{ \pi / 2 } \cos t \, n \, \cos t \, \sin^{n-1} t \, dt \\
  & = \int_{0}^{ \pi / 2 } \cos t \, n \, \cos t \, \sin^{n-1} t \, dt \\
  & = \int_{0}^{ \pi / 2 } n \, \left( 1 - \sin^2 t \right) \, \sin^{n-1} t \, dt \\
  & = \int_{0}^{ \pi / 2 } n \, \sin^{n-1} t \, - int_{0}^{ \pi / 2 } n \, \sin^{n+1} t \, dt \\
  & = n I_{n -1} - n I_{n + 1} \\
  & = \frac{n}{n+1} I_n
 \end{aligned} $$

 Hence $$ I_{n+1} > 0 $$ and $$ I_{n+1} < I_n $$, and $$ I_{n+1} = \frac{n}{n+1} I_n $$.

 By applying the result for $$ 2n $$ and $$ 2n + 1 $$, we get

 $$ \begin{aligned}
 & I_{2n} = \frac{\left( 2n - 1 \right) \left( 2n - 3 \right) ... 1}{2n \left( 2n - 2 \right) ... 2} \frac{\pi}{2} \\
 & I_{2n+1} = \frac{2n \left( 2n - 2 \right) ... 2}{\left( 2n + 1 \right) \left( 2n - 1 \right) ... 1}
 \end{aligned} $$

 Which gives us the Wallis formula

 $$ \frac{2n \left( 2n - 2 \right) ... 2}{\left( 2n - 1 \right) \left( 2n - 3 \right) ... 1} \sim \sqrt{ \pi n } $$


## Stirling formula

Let's consider the following series $$ \sum \frac{a^n n^n}{n!}, \,\, a \in {\mathbb{R}} $$

If we apply the ratio test to this series

$$ \begin{aligned}
\frac{U_{ n+1 }}{n} & = \frac{ a^{n+1} \left( n+1 \right)^{n+1} }{ \left( n+1 \right)! } * \frac{ n! }{ a^n n^n } \\
 & = a \left( 1 + \frac{1}{n} \right)^n \\
 & \require{AMScd} \begin{CD} @>>{n \to +\infty}>\end{CD} ae
\end{aligned} $$

 * if $$ a \lt e^{-1} $$, the series converges
 * if $$ a \gt e^{-1} $$, the series dieverges
 * if $$ a = e^{-1} $$, we have

 $$ \begin{aligned}
 ln \left( \frac{U_{n+1}}{U_n} \right) & = ln \left( e^{-1} \left( 1 + \frac{1}{n} \right)^n \right) \\
  & = -1 + nln \left( 1 + \frac{1}{n} \right) \\
  & = -1 + n \left( \frac{1}{n} - \frac{1}{2n^2} + o\left( \frac{1}{n^3} \right) \right) \\
  & = -\frac{1}{2n} + o\left( \frac{1}{n^2} \right)
 \end{aligned} $$

 Ans since

 $$ ln\left( \sqrt{ \frac{n+1}{n} } \right) = \frac{1}{2n} + o\left( \frac{1}{n^2} \right) $$

 It follows that

 $$ ln\left( \frac{ U_{n+1} }{U_n} \right) + ln\left( \sqrt{ \frac{n+1}{n} } \right) = o\left( \frac{1}{n^2} \right)$$

 Which means that

$$ \sum ln\left( \sqrt(n+1) + U_{n+1} \right) - ln\left( \sqrt{n} \ln\left( n \right) \right) \,\,\, \text{converges}$$

 Hence $$ {\left( ln\left( \sqrt{n} U_n \right) \right)}_n $$ converges.

 $$ \begin{aligned}
  \exists k \lt 0, \,\, & \text{such that}, \,\, lim \sqrt{n}U_n = k \\
  & \Rightarrow U_n \sim \frac{k}{\sqrt(n)} \\
  & \Rightarrow \frac{ e^{-n}n^n }{n!} \sim \frac{k}{\sqrt{n}} \\
  & \Rightarrow n! \sim \frac{1}{k} \sqrt{n} \left( \frac{n}{e} \right)^n \\
 \end{aligned} $$

 Now we only need to determine the value of $$ k $$, and for this we will use the Wallis formula,

 $$ \frac{ 2n \left( 2n - 2 \right) ... 2 }{ \left( 2n - 1 \right) \left( 2n - 3 \right) ... 1 } \sim \sqrt{ \pi n } $$

 Also,

 $$ \begin{aligned}
  \frac{ 2n \left( 2n - 2 \right) ... 2 }{ \left( 2n - 1 \right) \left( 2n - 3 \right) ... 1 } & = \frac{ \left( 2n \left( 2n - 2 \right) ... 2 \right)^2 }{ \left( 2n \right)! } \\
  & = \frac{ 2^{ 2n } \left( n! \right)^2 }{ \left( 2n \right)! }  \\
  & = \frac{ \require{cancel} \cancel{ 2^{ 2n } } \left( \frac{1}{k} \sqrt{n} \require{cancel} \cancel{\left( \frac{n}{e} \right)^n } \right)^2 }{ \frac{ \sqrt{2n} }{k} \require{cancel} \cancel{\left( \frac{2n}{ e } \right)^{2n}} }  \\
 \end{aligned} $$

 $$ \begin{aligned}
 & \Rightarrow \sqrt{\pi n} \sim \frac{1}{k} \sqrt(n/2) \\
 & \Rightarrow \frac{1}{k} \sim \sqrt{2 \pi} \\
 & \Rightarrow n! \sim \sqrt{2 \pi n} \left( \frac{n}{e} \right)^n
 \end{aligned} $$

The result is amazing, because it replaces the factorial function which is complicated with a simple expression. Also for very large numbers, the factorial is an expensive function, on the other hand the approximation is less computationally intensive.

```python
from sympy.mpmath import e, sqrt, pi

def stirling(n):
    return sqrt(2 * pi * n) * (n / e) ** n


for n in xrange(1, 10):
    print('n:{} \t factorial: {} \t stirling: {}'.format(n, math.factorial(n), stirling(n)))


# n:1 	 factorial: 1 	 stirling: 0.922137008895789
# n:2 	 factorial: 2 	 stirling: 1.91900435148898
# n:3 	 factorial: 6 	 stirling: 5.83620959134586
# n:4 	 factorial: 24 	 stirling: 23.5061751328933
# n:5 	 factorial: 120 	 stirling: 118.01916795759
# n:6 	 factorial: 720 	 stirling: 710.078184642185
# n:7 	 factorial: 5040 	 stirling: 4980.39583161246
# n:8 	 factorial: 40320 	 stirling: 39902.3954526567
# n:9 	 factorial: 362880 	 stirling: 359536.872841948


# approximation for a large value

stirling(10**10)
# 2.3257959759770513e+95657055186

```

Also based on this formula we can find easily the following result:

$$ \log(n!) \sim n\log(n) $$

In the next post we will try to approximate the factorials using the Euler gamma function.

