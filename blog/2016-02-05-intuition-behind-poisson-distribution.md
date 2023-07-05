---
layout: blog
title: "Intuition behind Poisson distribution"
subtitle: "Intuitive deduction of the Poisson distribution."
cover_image: images/blog/random.jpg
cover_image_caption: ""
tags: [math, probability, python]
---
Let's consider again our experiment where we place an ad on a webpage. In the previous post we wanted to calculate the number of users clicking the ad after visiting our web page. In this post we would like to estimate the number of users visiting our web page per hour.

Let consider the random variable: $$ X = \text{number of views per hour} $$.

Let $$ \lambda $$ be the expected value of $$ X $$, $$ E(X) = \lambda $$. If we had a binomial distribution we would have $$ E(X) = n * p $$

in other words, we can write $$ E(X) = \lambda = 60 * \frac{ \lambda }{ 60 } = 60 * \lambda_m $$, with $$ \lambda_m = \frac{ \lambda }{60} $$ is the probability for views per minute if it was a binomial distribution.

So the probability that we get $$ k $$ views in a hour would be (if we consider the binomial distribution):

$$ P(X = k) = \binom{60}{k} * \left( \frac{ \lambda }{ 60 } \right) ^ k * \left( 1 - \frac{\lambda }{ 60 } \right) ^ {(60 - k)} $$

The issue here is that we assume we can only get one click per minute, meaning that if we get more views per minute we would still count them as 1.

An easy way to remediate to this problem is to make it more granular, let’s say per second

$$ P(X = k) = \binom{3600}{k} * \left( \frac{ \lambda }{ 3600 } \right) ^ k *  \left( 1 - \frac{\lambda }{ 3600 } \right) ^ {(3600 - k)} $$

Again we would get the same problem if we get more than one click per second. And so we need to make our estimation more granular.

We know that

$$ \lim \limits_{x \to +\infty} (1 + \frac{a}{x})^x = \mathrm{e}^a $$

If we take the granularity to the infinite numbers we find the Poisson distribution:

$$ \begin{aligned}
P(X = k) & = \lim \limits_{n \to +\infty} \binom{n}{k} * \left( \frac{ \lambda }{ n } \right) ^ k * \left( 1 - \frac{\lambda }{ n } \right) ^ {(n - k)} \\
 & = \lim \limits_{n \to +\infty} \frac{n!}{(n - k)! * k!} * \left( \frac{ \lambda }{ n } \right) ^ k *  \left( 1 - \frac{\lambda }{ n } \right) ^ n * \left( 1 - \frac{\lambda }{ n } \right) ^ {-k} \\
 & = \lim \limits_{n \to +\infty} \frac{n!}{(n - k)!} * \frac{1}{n ^ k} * \frac{\lambda ^ k}{k!} *  \frac{1 - \lambda}{n} ^ n * \frac{1 - \lambda}{n} ^ {-k} \\
 & = \frac{\lambda ^ k}{k!} * e^{-\lambda}
 \end{aligned} $$
