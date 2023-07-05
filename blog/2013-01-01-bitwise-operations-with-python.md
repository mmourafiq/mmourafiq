---
layout: blog
title: "Bitwise operations with python."
subtitle: "Handling string of bits."
cover_image: images/blog/binary.jpg
cover_image_caption: "Binary"
tags: [ python ]
---

Bitwise operators allows us to write more efficient code due to the fact that CPUs are good at
handling bits. They treat numbers naturally but in different way, indeed, instead of considering a
number as single value they treat it as a string of bits.

* 0 is written as “0”
* 1 is written as “1”
* 2 is written as “10”
* 3 is “11”
* …

Important bitwise operators to know:

* x << y : Returns x with the bits shifted to the left by y places (and new bits on the
  right-hand-side are zeros). This is the same as multiplying x by 2**y.
* x >> y : Returns x with the bits shifted to the right by y places. This is the same as dividing x
  by 2**y.
* x & y : Does a “bitwise and”. Each bit of the output is 1 if the corresponding bit of x AND of y
  is 1, otherwise it’s 0.
* x \| y : Does a “bitwise or”. Each bit of the output is 0 if the corresponding bit of x AND of y
  is 0, otherwise it’s 1.
* ~ x : Returns the complement of x - the number you get by switching each 1 for a 0 and each 0 for
  a 1. This is the same as -x - 1.
* x ^ y : Does a “bitwise exclusive or”. Each bit of the output is the same as the corresponding bit
  in x if that bit in y is 0, and it’s the complement of the bit in x if that bit in y is 1.

Now let’s do some experiments with these bitwise operators.

**Finding even and odd numbers**:

```python
# odd or even the way most people would do
odd_or_even = lambda n: "Even" if n % 2 == 0 else "Odd"
# odd or even using bitwise operations
odd_or_even = lambda n: "Even" if n & 1 == 0 else "Odd"
# the last odd_or_even function has the virtue to only check the last digit.
# Conversely, the former deals with division, and it would be annoying to do long division.
# The "&" operator performs a bitwise AND.
# If our number was 17 then it would yield 1 because only the first binary column for both numbers contains a 1.
# Likewise if the number was 60 then we'd get back 0.
bin(17)
'0b10001'
odd_or_even(17)
'Odd'

bin(60)
'0b111100'
odd_or_even(60)
'Even'
```

**Double and half**:

```python
# double and half the intuitive way
double = lambda n: n * 2
half = lambda n: n / 2
# double and half using bitwise operations
double = lambda n: n << 1
half = lambda n: n >> 1
```

**URL shortener**:

Now we are going to see a more elaborated example of using bitwise operations; a URL shortener. I
think you are more used to this concept since you probably use it everyday, either to access a post
blog using a short URL, or as mean to shorten your URL and share it more conveniently.

In order for us to make this URL shortener, we will be in need of an object that, for instance, take
the URL id from your database and generate from it a code. The same code will be used to retrieve
the URL afterward.

```python
DEFAULT_ALPHABET = 'az7er5tyu1io0pq4sd9fg6hjk8lmw3xcv2bn'
LEGTH_GENERATION = 32
MIN_LENGTH = 6


class URLEncoder(object):
    """
It generates 36**6 = 2176782336 values (6 lowercase letters) which falls between 2**31 = 2147483648 and 2**32 = 4294967296.
It pads the codes that are shorter than 6 characters with leading 'a' characters,
"""

    def __init__(self, alphabet=DEFAULT_ALPHABET, length_generation=LEGTH_GENERATION):
        self.alphabet = alphabet
        self.length_generation = length_generation
        self.mask = (1 << length_generation) - 1
        self.mapping = range(length_generation)
        self.mapping.reverse()

    def encode_url(self, n, min_length=MIN_LENGTH):
        return self.__enbase(self.__encode(n), min_length)

    def decode_url(self, n):
        return self.__decode(self.__debase(n))

    def __encode(self, n):
        result = 0
        for i, b in enumerate(self.mapping):
            if (n & self.mask) & (1 << i):
                result |= (1 << b)
        return (n & ~self.mask) | result

    def __decode(self, n):
        result = 0
        for i, b in enumerate(self.mapping):
            if (n & self.mask) & (1 << b):
                result |= (1 << i)
        return (n & ~self.mask) | result

    def __enbase(self, x, min_length=MIN_LENGTH):
        result = self.__enbase_iter(x)
        padding = self.alphabet[0] * (min_length - len(result))
        return '%s%s' % (padding, result)

    def __enbase_iter(self, x):
        n = len(self.alphabet)
        if x < n:
            return self.alphabet[x]
        return self.__enbase_iter(x / n) + self.alphabet[x % n]

    def __debase(self, x):
        n = len(self.alphabet)
        result = 0
        for i, c in enumerate(reversed(x)):
            result += self.alphabet.index(c) * (n ** i)
        return result
```

For a full example, I created this django
app : [django-short-urls](https://github.com/mmourafiq/django-short-urls).

