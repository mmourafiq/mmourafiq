---
layout: blog
title: "Estimating Pi with Monte Carlo method."
subtitle: "Using the Monte Carlo method to estimate the value of pi."
cover_image: images/blog/random.jpg
cover_image_caption: "Random"
tags: [ math, probability, python ]
---

[Monte Carlo](http://en.wikipedia.org/wiki/Monte_Carlo_method) is statistical method based on a
series of random numbers. Monte Carlos are used in a wide-range of physical systems, finance and
other research field.

In this post, we are going to use the Monte Carlo method to estimate the value of pi.

First, consider a circle inscribed in a square (as in the figure).

![circle-square](/images/blog/circle-square.gif)

f assume that the radius of the circle is $$R$$, then the Area of the circle $$= Pi * R^2$$ and the
Area of the squar $$= (2 * R)^2 = 4 * R^2$$.

If we throw a dart blindly inside of the square, what is the probability (P) the dart will actually
land inside the circle?

P = Area of the circle / Area of the square = Pi / 4

So the chances of hitting the circle are Pi / 4.

In other words, $$pi = 4 * P$$

Let's try to do this in python:

```python
from __future__ import division
import numpy

NBR_POINTS = 1000000
RADIUS = numpy.sqrt(NBR_POINTS)

print(
    (len(filter(lambda x: numpy.sqrt(numpy.random.randint(0, RADIUS) ** 2 + numpy.random.randint(0, RADIUS) ** 2) < RADIUS, xrange(NBR_POINTS))) / NBR_POINTS) * 4
)
```

If you have problem with the
filter, [read this](http://localhost:4000/2012/03/01/pythonic-lists.html)!

To go a little bit further with this we can imagine a map-reduce system to make the estimation
faster.

```python
from __future__ import division
import collections
import itertools
import multiprocessing
import numpy


class MapReduce(object):
    """
    The map reduce object, should be initialized with:
        map_fn
        reduce_fn
        nbr_workers
    """

    def __init__(self, map_fn, reduce_fn, num_workers=None):
        """
        initiaize the mapreduce object
            map_fn : Function to map inputs to intermediate data, takes as
            input one arg and returns a tuple (key, value)
            reduce_fn : Function to reduce intermediate data to final result
            takes as arg keys as produced from the map, and the values associated with it
        """
        self.map_fn = map_fn
        self.reduce_fn = reduce_fn
        self.pool = multiprocessing.Pool(num_workers)

    def partition(self, mapped_values):
        """
        returns the mapped_values organised by their keys. (keys, associated values)
        """
        organised_data = collections.defaultdict(list)
        for key, value in mapped_values:
            organised_data[key].append(value)
        return organised_data.items()

    def __call__(self, inputs=None, chunk_size=1):
        """
        process the data through the map reduce functions.
        inputs : iterable
        chank_size : amount of data to hand to each worker
        """
        mapped_data = self.pool.map(self.map_fn, inputs, chunksize=chunk_size)
        partioned_data = self.partition(itertools.chain(*mapped_data))
        reduced_data = self.pool.map(self.reduce_fn, partioned_data)
        return reduced_data


NBR_POINTS = 1000000
RADIUS = numpy.sqrt(NBR_POINTS)
NBR_WORKERS = 4
NBR_PER_WORKER = int(NBR_POINTS / NBR_WORKERS)


def probability_calculation(item):
    print(multiprocessing.current_process().name, 'calculating', item)
    output = []
    output.append(('pi', len(filter(lambda x: numpy.sqrt(
        numpy.random.randint(0, RADIUS) ** 2 + numpy.random.randint(0, RADIUS) ** 2) < RADIUS,
                                    xrange(NBR_PER_WORKER)))))
    return output


def estimate_pi(items):
    keys, occurrences = items
    return (sum(occurrences) / NBR_POINTS) * 4


mapper = MapReduce(probability_calculation, estimate_pi)
pi = mapper([i for i in xrange(NBR_WORKERS)])
print(pi)
```
