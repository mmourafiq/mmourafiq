---
layout: blog
title: "Quora Answer Classifier (Part 2, with KNN)."
subtitle: "Designing an algorithm to develop classifier that is able to tell good answers from bad answers, as well as humans can."
cover_image: images/blog/quora.png
cover_image_caption: "Quora"
tags: [code, python, machine-learning]
---

K-nearest neighbors (kNN) is an algorithm used to show how you could build models for predicting observations given a set of examples. kNN works by taking a new item for which you want a numerical prediction and comparing it to a set of items for which it already has values. It finds the ones most similar to the item in question and averages their values to get a predicted value.

We will be using the same item model built during part 1.

The first thing we’ll need is a way to measure how similar two observations are. There are multiple choices for calculating similarity; based on Euclidean distance, Pearson correlation and Tanimoto score. We will be using the Euclidean distance for our solution.

It’s always wise to take more than one neighbor, because you’d never know if its is just an anomaly. The k in k-nearest neighbors refers to how many of the top matches you use for averaging.

An extension to the basic averaging is to use a weighted average based on how far away the neighbors are. A very close neighbor would count for more than an item that was further away. The weights would be in proportion to the total distance. We will be using 3 kind of weighted average functions: inverse weight, subtract weight and gaussian weight.

```python
class KNN(object):

    @staticmethod
    def euclidean(v1, v2):
        d = 0.0
        for i in range(len(v1)):
            d += (v1[i] - v2[i]) ** 2
        return math.sqrt(d)

    @staticmethod
    def all_distances(data, item, mask=None):
        len_coords = len(data[0].coords)
        if mask is None:
            mask = [i for i in xrange(len_coords)]
        distancelist = []
        for i in range(len(data)):
            item2 = data[i]
            distancelist.append((KNN.euclidean([item.scaled_coords[x] for x in xrange(len_coords) if x in mask],
                                               [item2.scaled_coords[x] for x in xrange(len_coords) if x in mask]), i))
        distancelist.sort()
        return distancelist

    @staticmethod
    def inverse_weight(dist, num=1.0, const=0.1):
        return num / (dist + const)

    @staticmethod
    def subtract_weight(dist, const=1.0):
        if dist > const:
            return 0
        else:
            return const - dist

    @staticmethod
    def gaussian(dist, sigma=10.0):
        return math.e ** (-dist ** 2 / (2 * sigma ** 2))

    @staticmethod
    def knn_estimate(data, item, k=8, mask=None):
        # Get sorted distances
        all_dist = KNN.all_distances(data, item, mask)
        avg = 0.0
        # Take the average of the top k results
        for i in range(k):
            idx = all_dist[i][1]
            avg += int(data[idx].value)
        #avg=avg/k
        result = "-1"
        if avg > 0:
            result = "+1"
        return result

    @staticmethod
    def weighted_knn(data, item, k=8, weight_f="gaussian", mask=None):
        if weight_f == "subtract_weight":
            weightf = KNN.subtract_weight
        elif weight_f == "inverse_weight":
            weightf = KNN.inverse_weight
        elif weight_f == "gaussian":
            weightf = KNN.gaussian
        # Get distances
        all_dist = KNN.all_distances(data, item, mask)
        avg = 0.0
        totalweight = 0.0
        # Get weighted average
        for i in range(k):
            dist = all_dist[i][0]
            idx = all_dist[i][1]
            weight = weightf(dist)
            avg += weight * int(data[idx].value)
            totalweight += weight
        #avg=avg/totalweight
        result = "-1"
        if avg > 0:
            result = "+1"
        return result

    @staticmethod
    def divide_data(data, test=0.05):
        trainset = []
        testset = []
        for row in data:
            if random.random() < test:
                testset.append(row)
            else:
                trainset.append(row)
        return trainset, testset

    @staticmethod
    def test_algorithm(algf, trainset, testset):
        error = 0.0
        for item in testset:
            guess = algf(trainset, item)
            if item.value != guess:
                error += 1
        return error / len(testset)

    @staticmethod
    def cross_validate(algf, data, trials=100, test=0.05):
        error = 0.0
        for i in range(trials):
            trainset, testset = KNN.divide_data(data, test)
            error += KNN.test_algorithm(algf, trainset, testset)
        return error / trials
```

The object also introduce some utilities to cross validate the test data, by dividing and testing.

It’s also wise to point out that scaling data is important, the same way it is important to get rid of the dimensions that only introduces anomalies to our model. for this purpose we need a utility function that creates a mask for the useful dimensions and their scores.

```python
def create_score_factors_linear_knn(training_data, classify_data, results, k=8):
    len_coords = len(training_data[0].coords)
    corrects = [0] * len_coords
    t_corrects = [0] * len_coords
    f_corrects = [0] * len_coords
    factors = [0] * len_coords
    scores = [0] * len_coords
    for i in xrange(len_coords):
        means = linear_mean(training_set, "+1", "-1", [i])
        for obs in classify_data:
            if dot_product_classification(obs, means, "+1", "-1", [i]) == results[obs.id]:
                corrects[i] += 1
                if results[obs.id] == "-1":
                    f_corrects[i] += 1
                else:
                    t_corrects[i] += 1
    print(corrects)
    print(t_corrects)
    print(f_corrects)
    scores = [(corrects[i], i) for i in xrange(len_coords)]
    scores.sort()
    mask = [scores[-1][1]]
    current_correct = scores[-1][0]
    for i in xrange(len_coords - 2, -1, -1):
        new_mask = mask[:]
        new_mask.append(scores[i][1])
        correct = 0
        for obs in classify_data:
            cr = KNN.knn_estimate(training_data, obs, k, mask)
            if cr == results[obs.id]:
                correct += 1
        if correct >= current_correct - 5:
            current_correct = correct
            mask = new_mask[:]
    min_corrects = min(corrects)
    max_corrects = max(corrects)
    for i in xrange(len_coords):
        factors[i] = ((corrects[i] - min_corrects) / ((max_corrects - min_corrects) + 0.000001))
    return mask, factors
```

Finally to run everything together, we need to read and check values:

```python
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
    #coords=[coords[i] for i in xrange(M) if i in [1, 2, 4, 6, 7, 11, 14, 18]]
    training_set.append(Item(id=id, coords=coords, value=value))
Q = int(r_file.readline().strip('\n'))
classify_set = []
for i in xrange(Q):
    current_line = r_file.readline().strip('\n').split()
    id = current_line[0]
    coords = [float(x.split(':')[1]) for x in current_line[1:]]
    assert len(coords) == M
    #coords=[coords[i] for i in xrange(M) if i in [1, 2, 4, 6, 7, 11, 14, 18]]
    classify_set.append(Item(id=id, coords=coords))
#reading the results
results = {}
for i in xrange(Q):
    current_line = w_file.readline().strip('\n').split()
    results[current_line[0]] = current_line[1]

mask = [1, 2, 4, 6, 7, 11, 14, 18]
for item in training_set:
    item.scale()
correct = 0
for obs in classify_set:
    obs.scale()
    cr = KNN.knn_estimate(training_set, obs, k=8, mask=mask)
    if cr == results[obs.id]:
        correct += 1
print(correct)
correct = 0
for obs in classify_set:
    cr = KNN.weighted_knn(training_set, obs, k=8, mask=mask)
    if cr == results[obs.id]:
        correct += 1
print(correct)
correct = 0
for obs in classify_set:
    cr = KNN.weighted_knn(training_set, obs, k=8, weight_f="subtract_weight", mask=mask)
    if cr == results[obs.id]:
        correct += 1
print(correct)
correct = 0
for obs in classify_set:
    cr = KNN.weighted_knn(training_set, obs, k=8, weight_f="inverse_weight", mask=mask)
    if cr == results[obs.id]:
        correct += 1
print(correct)
```

As you can see there’s a mask, it’s the best mask that I found, and you should note that we scale all observations before training and predicting.
