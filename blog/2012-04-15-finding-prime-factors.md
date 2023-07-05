---
layout: blog
title: "Finding prime factors of x"
cover_image: images/blog/matrix-code.jpg
cover_image_caption: "Write code."
tags: [python, math]
---

In a previous post, we showed how any number can be written, in a unique way, as the product of prime numbers.

In this post we will make a simple algorithm to compute this prime numbers.

```python
def get_primes(x):
    non_primes = set()
    primes = set()
    for i in range(2, x+1):
        if x%i == 0 and i not in non_primes:
            primes.add(i)
            j = i
            while j*i <= x:
                non_primes.add(i*j)
                j += 1
    return primes
```
