---
layout: blog
title: "Intuition behind the Binomial distribution"
subtitle: "Intuitive deduction of the Binomial distribution."
cover_image: images/blog/random.jpg
cover_image_caption: ""
tags: [math, probability, python]
---
Let’s consider a small experiment where we place an ad on a webpage and we want to calculate how many users would click this ad after visiting the page given that the probability of user clicking the ad is « p ».

For each visit we have two outcomes, user clicks the ad and user does't click the ad, in other terms we have two outcomes that we will denote success and failure. So basically what we want to do is to compute the probability of observing a specific number of « successes »  when the process is repeated a specific number of times (i.e. users visiting a webpage) and the outcome for a given user is either a success or a failure.

First, let $$ n $$ be the number of observations or the number of times the process is repeated (users coming to the webpage), and $$ k $$ the number of "successes" or events of interest occurring during the $$ n $$ observations. The probability of "success" or occurrence of the outcome of interest is indicated by $$ p $$.

To make an example, if we had 10 users visiting our webpage, we would want to know how many of these 10 users would click our ad, knowing that the probability of clicking an ad is p.

Let’s calculate different probabilities getting different numbers of outcome:

 * the probability of getting no clicks

This is the same as getting 10 failures $$ P(X=FFFFFFFFFF) = (1-p)^{10} $$

 * The probability of getting 1 click

This could be the first user clicking (SFFFFFFFFF), or the second one (FSFFFFFFFF), or the third (FFSFFFFFFF), or any other position.

The probability of each one of these experiences is

$$ \begin{aligned}
P(X = SFFFFFFFFF) & = p * (1 - p)^9 \\
P(X = FSFFFFFFFF) & = (1 - p) * p * (1 - p)^8 = p * (1 - p)^9 \\
P(X = FFSFFFFFFF) & = (1 - p)^2 * p * (1 - p)^7 = p * (1 - p)^9 \\
\end{aligned} $$

And hence the probability of getting one success is : $$ 10 * p * (1 - p)^9 $$

 * the probability of getting 2 clicks

Again this could be the first 2 users (SSFFFFFFFF), or the first and the fourth (SFFSFFFFFF), and you can see that it’s getting a bit complicated to go through all combinations. But what we can realize is that each scenario has a probability $$ P = p^2 * (1 - p)^8 $$, and what we should do is choose 2 users out of 10 users to make a click without caring about the order of the users.

You might start getting an intuition why we would need the binomial coefficient, for the first user we can choose from 10 users and for the second we can choose from 9 users and since this could happen in two different ways (first user could be the second one) we would get $$ \frac{10 * 9}{2} = \binom{10}{2} $$

 * the probability of getting 3 clicks

again this is similar to the case of 2 users, we can define the probability of one scenario $$ P(SSSFFFFFFF)  = p^3 * (1 - p)^7 $$

and again since the order does not matter we get $$ \frac{10 * 9 * 8}{3} * p^3 * (1 - p)^7 = \frac{10!}{3! * 7!} * p^3 * (1 - p)^7 = \binom{10}{3} * p^3 * (1 - p)^7 $$

Finally we can generalize for any $$ n $$ and $$ k $$

$$ P(\text{k successes}) = \binom{n}{k} * p^k * (1 - p)^{n - k} $$
