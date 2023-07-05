---
layout: blog
title: "Make your code pythonic"
subtitle: "Some python best practices to follow."
tags: [python]
cover_image: images/blog/write-code.png
cover_image_caption: "Write code."
---
**Enemurate**
```python
#  example of bad code
for indice in xrange(0, len(l)):
    print("l[%d] = %r" % (indice, l[indice]))

# ==>
#  the best way to perform such operation (get indices and corresponding values) is by using enumerate

for indice, value in enumerate(l):
    print("l[%d] = %r" % (indice, value))
#  since enumerate returns a list of tuples :
for i in enumerate(l):
    print("l[%d] = %r" % i)
```

**Generators**
```python
#  simple range
def my_range(x, y=0):
    if y < x: # inverse if y < x
        x, y = y, x

    yield x # return the first x
    while x < y:
        x += 1
        yield x

#  simple enumerate
def enumerate(i):
    for i in xrange(0, len(i)):
        yield i, i[i]

#  simple xrange
def xrange(start, stop=None, step=1):
    if stop is None:
        start, stop = 0, start
    while step > 0 and start < stop or step < 0 and start > stop:
        yield start
        start += step
```

**Coroutines**
```python
def cpt(x):
    n = 0
    while n <= x:
        v = (yield n)
        if v is not None:
            n = v
        else:
            n += 1

gen = cpt(20)
for i in gen:
    print(i)
    # we want to go from 15 to 18
    if i == 15:
        gen.send(18)
```

**Zip**
```python
#  suppose we want to iterate over 2 list l1 & l2
#  the easiest way to do is :
for i in xrange(min(len(l1), len(l2))):
    v1, v2 = l1[i], l2[i]
    # do something with v1, v2

#  the easiest way to do so:
for v1, v2 in zip(l1, l2):
    # do something with v1, v2

#  zip definition:
#  zip(...)
    # zip(seq1 [, seq2 [...]]) -> [(seq1[0], seq2[0] ...), (...)]
    # Return a list of tuples, where each tuple contains the i-th element
    # from each of the argument sequences. The returned list is truncated
    # in length to the length of the shortest argument sequence.

#  e.g
print(zip(['a', 'b', 'c'], ['d', 'e', 'f'])) # [('a', 'd'), ('b', 'e'), ('c', 'f')]

#  python doesn't provide an unzip function but we can easily code one:
unzip = lambda l: [list(li) for li in zip(*l)]
unzip([(1, 2), (3, 4), (5, 6)]) # [[1, 3, 5], [2, 4, 6]]
```

**Itertools**

```python
#  imap, ifilter, ifilterfalse
from itertools import imap, ifilter, ifilterfalse

pair = lambda n: n % 2 == 0
sqr = lambda n: n ** 2

imap(carre, xrange(10))  # Renvoie un générateur <itertools.imap object at 0x7f4ab174fc10>

for elem in imap(sqr, xrange(10)): print(elem)
# 0
# 1
# 4
# 9
# 16
# 25
# 36
# 49
# 64
# 81

list(ifilter(pair, xrange(10)))  # [0, 2, 4, 6, 8]

list(ifilterfalse(pair, xrange(10)))  # [1, 3, 5, 7, 9]

#  chain
from itertools import chain

list(chain(xrange(11), xrange(15, 21), xrange(30, 46)))
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 15, 16, 17, 18, 19, 20, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45]

#  takewhile & dropwhile
from itertools import takewhile, dropwhile

list(takewhile(lambda i: i < 10, xrange(20)))  # takes while i < 10 : [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
list(dropwhile(lambda i: i < 10,
               xrange(20)))  # drops while i < 10 : [10, 11, 12, 13, 14, 15, 16, 17, 18, 19]

#  groupby
from itertools import groupby

l = list('aaaaabbbcddddee')
for elt, itr in groupby(l):
    print("%s : %d times" % (elt, len(list(itr))))
# a : 5 times
# b : 3 time
# c : 1 time
# d : 4 times
# e : 2 times
```

**With**
```python
#  very important use of "with" is with file management
f = open('file', 'w')
try:
    # Operations f
except Exception:
    # some problems
finally:
    # close f
    f.close()

#  we can the exact same :
with open('file', 'w') as f:
    # Operations f
```
