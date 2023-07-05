---
layout: blog
title: "Quora Answer Classifier (Part 3, linear and non linear classifiers)."
subtitle: "Designing an algorithm to develop classifier that is able to tell good answers from bad answers, as well as humans can."
cover_image: images/blog/quora.png
cover_image_caption: "Quora"
tags: [code, python, machine-learning]
---

We saw in previous posts how we could use decision trees and K-nearest neighbors to classify answers.

In this post, the last of this series of posts, we will be using two classifiers; linear and non linear classifiers.

Grab the code of the model item from part 1. Since we are going to use it for this two classifiers.

## Linear Classifiers

It is one of the simplest classifiers to construct. It works by finding the average of all the data in each class and constructing a point that represents the center of the class. It can then classify new points by determining to which center point they are closest.

First we need a function that calculates the average of items. Then we need a way to calculate the closeness of new item. While we can use Euclidean distance, which will work for this classifier, we can take a different approach using vectors and dot-products. Also we will be using the scaling method, since all the variables aren’t really comparable.

```python
def linear_mean(data, t_value, f_value, mask=None):
    #two categories
    t_coords = []
    f_coords = []
    for item in data:
        if item.value == t_value:
            if mask is None:
                t_coords.append(item.scaled_coords)
            else:
                t_coords.append([item.scaled_coords[i] for i in xrange(len(item.scaled_coords)) if i in mask])
        elif item.value == f_value:
            if mask is None:
                f_coords.append(item.scaled_coords)
            else:
                f_coords.append([item.scaled_coords[i] for i in xrange(len(item.scaled_coords)) if i in mask])
    t_mean = numpy.average(numpy.array(t_coords), 0)
    f_mean = numpy.average(numpy.array(f_coords), 0)
    return t_mean, f_mean

def dot_product_classification(item, means, t_value, f_value, mask=None):
    t_mean, f_mean = means
    if mask is None:
        item_coords = numpy.array(item.scaled_coords)
    else:
        item_coords = numpy.array([item.scaled_coords[i] for i in xrange(len(item.scaled_coords)) if i in mask])
    y = (numpy.dot(f_mean, f_mean) - numpy.dot(t_mean, t_mean)) / 2
    x = (numpy.dot(item_coords, t_mean) - numpy.dot(item_coords, f_mean)) + y
    return t_value if x > 0 else f_value
```

## Non Linear Classifiers

We will be using a very basic non linear classifier. also we will be using the radial-basis function as a kernel trick for our classifier. which replaces the dot-product function, returning what the dot-product would have been if the data had first been transformed to a higher dimensional space using some mapping function.

```python
def radial_basis(item1, item2, gamma=10):
    diff_item = numpy.array(item1.coords) - numpy.array(item2.coords)
    dist_item = numpy.sqrt(numpy.sum(diff_item))
    return math.e ** (-gamma * dist_item)

def non_linear_classification(item, data, t_value, f_value, offset, gamma=10):
    t_sum = 0
    t_count = 0
    f_sum = 0
    f_count = 0
    for item2 in data:
        if item2.value == t_value:
            t_sum += radial_basis(item2, item, gamma)
            t_count += 1
        elif item2.value == f_value:
            f_sum += radial_basis(item2, item, gamma)
            f_count += 1
    res = (float(t_sum) / t_count - float(f_sum) / f_count) + offset
    return t_value if res > 0 else f_value

def calculate_offset(data, t_value, f_value, gamma=10):
    t_data = []
    f_data = []
    for item in data:
        if item.value == t_value:
            t_data.append(item)
        elif item.value == f_value:
            f_data.append(item)
    t_sum = sum(sum([radial_basis(item1, item2, gamma) for item1 in t_data]) for item2 in t_data)
    f_sum = sum(sum([radial_basis(item1, item2, gamma) for item1 in f_data]) for item2 in f_data)
    return (float(f_sum) / (len(f_data) ** 2)) - (float(t_sum) / (len(t_data) ** 2))
```

Finally some utilities to run everything together:

```python
def create_score_factors_linear(training_data, classify_data, results):
    len_coords = len(training_data[0].coords)
    corrects = [0] * len_coords
    factors = [0] * len_coords
    scores = [0] * len_coords
    for i in xrange(len_coords):
        means = linear_mean(training_set, "+1", "-1", [i])
        for obs in classify_data:
            if dot_product_classification(obs, means, "+1", "-1", [i]) == results[obs.id]:
                corrects[i] += 1
    scores = [(corrects[i], i) for i in xrange(len_coords)]
    scores.sort()
    mask = [scores[-1][1]]
    current_correct = scores[-1][0]
    for i in xrange(0, len_coords - 1):
        new_mask = mask[:]
        new_mask.append(scores[i][1])
        means = linear_mean(training_set, "+1", "-1", new_mask)
        correct = 0
        for obs in classify_data:
            if dot_product_classification(obs, means, "+1", "-1", new_mask) == results[obs.id]:
                correct += 1
        if correct >= current_correct:
            current_correct = correct
            mask = new_mask[:]
    min_corrects = min(corrects)
    max_corrects = max(corrects)
    for i in xrange(len_coords):
        factors[i] = ((corrects[i] - min_corrects) / ((max_corrects - min_corrects) + 0.000001))
    return mask, factors


r_file = open('test.txt', 'r')
w_file = open('results.txt', 'r')
N, M = [int(x) for x in r_file.readline().strip('\n').split()]
training_set = []
for i in xrange(N):
    current_line = r_file.readline().strip('\n').split()
    id = current_line[0]
    value = current_line[1]
    coords = [float(x.split(':')[1]) for x in current_line[2:]]
    assert len(coords) == M
    # coords=[coords[i] for i in xrange(M) if i in [1, 2, 4, 6, 7, 11, 14, 18]]
    training_set.append(Item(id=id, coords=coords, value=value))
Q = int(r_file.readline().strip('\n'))
classify_set = []
for i in xrange(Q):
    current_line = r_file.readline().strip('\n').split()
    id = current_line[0]
    coords = [float(x.split(':')[1]) for x in current_line[1:]]
    assert len(coords) == M
    # coords=[coords[i] for i in xrange(M) if i in [1, 2, 4, 6, 7, 11, 14, 18]]
    classify_set.append(Item(id=id, coords=coords))
# reading the results
results = {}
for i in xrange(Q):
    current_line = w_file.readline().strip('\n').split()
    results[current_line[0]] = current_line[1]

# linear
correct = 0
for item in training_set:
    item.scale()
means = linear_mean(training_set, "+1", "-1")
for obs in classify_set:
    obs.scale()
    if dot_product_classification(obs, means, "+1", "-1") == results[obs.id]:
        correct += 1
print(correct)

# non linear
correct = 0
offset = calculate_offset(training_set, "+1", "-1", gamma=20)
for obs in classify_set:
    if non_linear_classification(obs, training_set, "+1", "-1", offset, gamma=20) == results[
        obs.id]:
        correct += 1
print(correct)
```
