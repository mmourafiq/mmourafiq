---
layout: blog
title: "Flipping coin."
subtitle: "How many coin flips does it take, on average, to get n consecutive heads?"
cover_image: images/blog/random.jpg
cover_image_caption: "Random"
tags: [ math, probability, python ]
---

Let's assume that we have a coin with a probability p (=0.5 if fair coin) of coming up head.

Let $$Avg_1$$ be the average number of flips before getting the first head. In this case, if the
first flip is head, then the answer is 1. If, on the other hand, the first flip is tail, then we
used 1 flip and we should do $$Avg_1$$ more flips before getting a head.

In other words, we have: $$Avg_1 = (1 - p) * (1 + Avg_1) + p * 1$$

Which leads to : $$Avg_1 = \frac{1}{p}$$

This seems right, since for a fair coin (p=0.5) the average number of flips to get a head is 2.

Now let $$Avg_2$$ be the average number of flips before getting two heads in a row. Which means, if
the first flip is a tail, we should expect $$(1 - p) * (1 + Avg_2)$$ flips before getting heads. In
case we have a head in the first flip, there are two probabilities for what's next. If next is tail,
then two flips are gone and we are back to the start point. But if next is head, then we get what we
want.

In other terms, we have:

$$Avg_2 = (1 - p) * (1 + Avg_2) + p * (1 - p) * (2 + Avg_2) + p^2 * 2$$

Which leads to : $$Avg_2 = \frac{1 + p}{p^2}$$

For the fair coin, the average number of flips to get two heads in a row is 6.

If we follow the same reasoning for the average number of flips to get 3 heads in a row.

$$Avg_3 = (1 - p) * (1 + Avg_3) + p * (1 - p) * (2 + Avg_3) + p^2 * (1 - p) * (3 + Avg_3) + p^3 *
3$$

which leads to : $$Avg_3 = \frac{1 + p + p^2}{p^3}$$

And for the fair coin, you should expect 14 flips on average to get 3 heads in a row.

In general:

$$Avg_n = \frac{1 + p + p^2 + ... + p^{n-1}}{p^n} = \frac{1 - p^n}{p^n * (1 - p)}$$

For the fair coin, we find that $$Avg_n = 2^{n+1} - 2$$ flips required to get n heads in a row.

Now having said that, doesn't mean that we will have our heads!

let's go back to $$n = 1$$ we said that we need on average 2 flips to get one head. The probability
to get this head is : 3/4 = 0.75.
Because the list of possible outcomes are : {(0,0), (0,1), (1,0), (1,1)}

Suppose now we want to calculate the probability for getting k heads in row with n flips. Sure we
can use the Markov chain to calculate this probability, but it will get difficult when our
experience grow.

Another solution is to simulate this process by doing a Monte Carlo.

A programme that simulates this operation:

```python
# generate the flips
flip = lambda n_times: ''.join([str(random.getrandbits(1)) for i in range(n_times)])
# flip(10) would give 1011100000 for ex
# calculate the payoff
payoff = lambda n_times, k: 1.0 if '1' * k in flip(n_times) else 0.0
# payoff returns 1.0 if k heads is found in n_times flips
# monte carlo
monte_carlo = lambda n_times, k, j: sum([payoff(n_times, k) for i in xrange(j)]) / j
```

monte_carlo(2, 1, 1000000) = 0.750024 approximately the same from a Markov chain method 3/4 = 0.75.

monte_carlo(10, 3, 1000000) = 0.508233 approximately the same from a Markov chain method 520/1024 =
0.5078125
