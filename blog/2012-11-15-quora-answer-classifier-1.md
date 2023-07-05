---
layout: blog
title: "Quora Answer Classifier (Part 1, with decision tree learning)."
subtitle: "Designing an algorithm to develop classifier that is able to tell good answers from bad answers, as well as humans can."
cover_image: images/blog/quora.png
cover_image_caption: "Quora"
tags: [ code, python, machine-learning ]
---

Quora uses a combination of machine learning (ML) algorithms and moderation to ensure high-quality
content on the site. High answer quality has helped Quora distinguish itself from other Q&A sites on
the web.

Your task will be to devise a classifier that is able to tell good answers from bad answers, as well
as humans can. A good answer is denoted by a +1 in our system, and a bad answer is denoted by a -1.

![quora-answer-classifier](/images/blog/quora-answer-classifier.png)

**Input format (read from STDIN)**:

The first line contains N, M. N = Number of training data records, M = number of parameters.
Followed by N lines containing records of training data. Then one integer q, q = number of records
to be classified, followed by q lines of query data

Training data corresponds to the following format:

```bash
<answer-identifier> <+1 or -1> (<feature-index>:<feature-value>)*
```

Query data corresponds to the following format:

```bash
<answer-identifier> (<feature-index>:<feature-value>)*
```

The answer identifier is an alphanumeric string of no more than 10 chars. Each identifier is
guaranteed unique. All feature values are doubles.

0 < M < 100

0 < N < 50,000

0 < q < 5,000

For example:

```bash
5 23
2LuzC +1 1:2101216030446 2:1.807711 3:1 4:4.262680 5:4.488636 6:87.000000 7:0.000000 8:0.000000 9:0 10:0 11:3.891820 12:0 13:1 14:0 15:0 16:0 17:1 18:1 19:0 20:2 21:2.197225 22:0.000000 23:0.000000
LmnUc +1 1:99548723068 2:3.032810 3:1 4:2.772589 5:2.708050 6:0.000000 7:0.000000 8:0.000000 9:0 10:0 11:4.727388 12:5 13:1 14:0 15:0 16:1 17:1 18:0 19:0 20:9 21:2.833213 22:0.000000 23:0.000000
ZINTz -1 1:3030695193589 2:1.741764 3:1 4:2.708050 5:4.248495 6:0.000000 7:0.000000 8:0.000000 9:0 10:0 11:3.091042 12:1 13:1 14:0 15:0 16:0 17:1 18:1 19:0 20:5 21:2.564949 22:0.000000 23:0.000000
gX60q +1 1:2086220371355 2:1.774193 3:1 4:3.258097 5:3.784190 6:0.000000 7:0.000000 8:0.000000 9:0 10:0 11:3.258097 12:0 13:1 14:0 15:0 16:0 17:1 18:0 19:0 20:5 21:2.995732 22:0.000000 23:0.000000
5HG4U -1 1:352013287143 2:1.689824 3:1 4:0.000000 5:0.693147 6:0.000000 7:0.000000 8:0.000000 9:0 10:1 11:1.791759 12:0 13:1 14:1 15:0 16:1 17:0 18:0 19:0 20:4 21:2.197225 22:0.000000 23:0.000000
2
PdxMK 1:340674897225 2:1.744152 3:1 4:5.023881 5:7.042286 6:0.000000 7:0.000000 8:0.000000 9:0 10:0 11:3.367296 12:0 13:1 14:0 15:0 16:0 17:0 18:0 19:0 20:12 21:4.499810 22:0.000000 23:0.000000
ehZ0a 1:2090062840058 2:1.939101 3:1 4:3.258097 5:2.995732 6:75.000000 7:0.000000 8:0.000000 9:0 10:0 11:3.433987 12:0 13:1 14:0 15:0 16:1 17:0 18:0 19:0 20:3 21:2.639057 22:0.000000 23:0.000000
```

This data is completely anonymized and extracted from real production data, and thus will not
include the raw form of the
answers. We, however, have extracted as many features as we think are useful, and you can decide
which features make sense to be included in your final algorithm. The actual labeling of a good
answer and bad answer is done organically on our site, through human moderators.

**Output format (write to STDOUT)**:

For each query, you should output q lines to stdout, representing the decision made by your
classifier, whether each answer is good or not:

```bash
<answer-identifier> <+1 or -1>
```

For example:

```bash
PdxMK +1
ehZ0a -1
```

You are given a relative large sample input dataset offline with its corresponding output to
finetune your program with your ML libraries. It can be downloaded
here: http://qsf.cf.quoracdn.net/Quora…

**Scoring**:

Only one very large test dataset will be given for this problem online as input to your program for
scoring. This input data set will not be revealed to you.

Output for every classification is awarded points separately. The score for this problem will be the
sum of points for each correct classification. To prevent naive solution credit (outputting all +1s,
for example), points are awarded only after X correct classifications, where X is number of +1
answers or -1 answers (whichever is greater).

**Timing**:

Your program should complete in minutes. Try to achieve as high an accuracy as possible with this
constraint in mind.

## Solution with decision tree learning:

Decision tree is a simple machine-learning method, completely transparent method of classifying
observations, which, after training, look like a series of if-then statements arranged into a tree.

But before we dive in our decision tree, we should first model items, an item represent an
observation, and should contain an id and value. Also each observations has certain parameters or
coordinates that our model should also contain. we can also go a bit further and add some utilities
that stores the highest and lowest values for this coordinates:

```python
class Item(object):
    """
    Describe an item

    @type id: string
    @param id: the id of the elemet

    @type value: string
    @param value: the value of the item

    @type coords: list
    @param coords: a list representing the parameters that locates the item
    """
    __highs = []
    __lows = []

    def __init__(self, id, coords, value=None):
        self.id = id
        self.coords = coords
        self.scaled_coords = coords[:]
        self.value = value
        self.__update_highs()
        self.__update_lows()

    def __update_highs(self):
        if Item.__highs == []:
            Item.__highs = [-9999999.0] * len(self.coords)
        else:
            for i in xrange(len(self.coords)):
                if self.coords[i] > Item.__highs[i]:
                    Item.__highs[i] = self.coords[i]

    def __update_lows(self):
        if Item.__lows == []:
            Item.__lows = [9999999.0] * len(self.coords)
        else:
            for i in xrange(len(self.coords)):
                if self.coords[i] < Item.__lows[i]:
                    Item.__lows[i] = self.coords[i]

    def scale(self, factors=None):
        if factors is None:
            self.scaled_coords = [
                (self.coords[i] - Item.__lows[i]) / ((Item.__highs[i] - Item.__lows[i]) + 0.0001)
                for i in xrange(len(self.coords))]
        else:
            self.scaled_coords = [((self.coords[i] - Item.__lows[i]) / (
                    (Item.__highs[i] - Item.__lows[i]) + 0.0001) * factors[i]) for i in
                                  xrange(len(self.coords))]
```

Now, that we have our observations modeled, we can start modeling our decision tree. The first step
is to create a representation of a the nodes. Each node has five variables:  column or coordinate
index, value of this coordinate, results stores a dictionary of results for this branch, t_node and
f_node which are the next nodes in the tree if the result is true or false, respectively.

```python
class Node(object):
    """
    A node object in a decision tree

    @type column: int
    @param column: the column index of the criteria to be tested

    @type value: string
    @param value: the value that the column must match to get a true result

    @type results: dict
    @param results: stores a dictionary of results for this branch. This is None for everything
                    except endpoints

    @type t_node: Node
    @param t_node: the next nodes in the tree if the result is true

    @type f_node: Node
    @param f_node: the next nodes in the tree if the result is false
    """

    def __init__(self, col=-1, value=None, results=None, t_node=None, f_node=None):
        self.col = col
        self.value = value
        self.results = results
        self.t_node = t_node
        self.f_node = f_node

    def draw(self, indent=''):
        # Is this a leaf node?
        if self.results != None:
            print(str(self.results))
        else:
            # Print the criteria
            print(str(self.col) + ':' + str(self.value) + '? ')
            # Print the branches
            print(indent + 'T->')
            self.t_node.draw(indent + '  ')
            print(indent + 'F->')
            self.f_node.draw(indent + '  ')
```

Now we need the decision tree methods, that will allow us to train and build the tree, classify and
prune in case of an over fitting tree.

```python
class DecisionTree(object):
    """
    A decision tree object
    """

    @staticmethod
    def count_results(data, item=True):
        """
        count the occurrences of each result in the data set
        """
        results_count = defaultdict(int)
        if item:
            for i in data:
                results_count[i.value] += 1
        else:
            results_count = Counter(data)
        return results_count

    @staticmethod
    def divide_data(data, column, value):
        """
        Divides a set of rows on a specific column.
        """
        # a function that decides if the row goes to the first or the second group (true or false)
        spliter = None
        if isinstance(value, int) or isinstance(value, float):
            spliter = lambda item: item.scaled_coords[column] >= value
        else:
            spliter = lambda item: item.scaled_coords[column] == value
        # divide the rows into two sets and return them
        set_true = []
        set_false = []
        for item in data:
            if spliter(item):
                set_true.append(item)
            else:
                set_false.append(item)
        return (set_true, set_false)

    @staticmethod
    def gini_impurity(data, item=True):
        """
        Probability that a randomly placed item will be in the wrong category
        """
        results_count = DecisionTree.count_results(data, item)
        len_data = len(data)
        imp = 0.0
        for k1, v1 in results_count.iteritems():
            p1 = float(v1) / len_data
            for k2, v2 in results_count.iteritems():
                if k1 == k2: continue
                p2 = float(v2) / len_data
                imp += p1 * p2
        return imp

    @staticmethod
    def entropy(data, item=True):
        """
        estimate the disorder in the data set : sum of p(x)log(p(x))
        """
        results_count = DecisionTree.count_results(data, item)
        len_data = len(data)
        ent = 0.0
        for v in results_count.itervalues():
            p = float(v) / len_data
            ent -= p * log2(p)
        return ent

    @staticmethod
    def variance(data):
        """
        calculates the statistical variance for a set of rows
        more preferably to be used with numerical outcomes
        """
        len_data = len(data)
        if len_data == 0: return 0
        score = [float(item.value) for item in data]
        mean = sum(score) / len(score)
        variance = sum([(s - mean) ** 2 for s in score]) / len(score)
        return variance

    @staticmethod
    def build_tree(data, disorder_function="entropy"):
        """
        a recursive function that builds the tree by choosing the best dividing criteria
        disorder_function :
            for data that contains words and booleans; it is recommended to use entropy or gini_impurity
            for data that contains number; it is recommended to use variance
        """
        if disorder_function == "entropy":
            disorder_estimator = DecisionTree.entropy
        elif disorder_function == "gini_impurity":
            disorder_estimator = DecisionTree.gini_impurity
        elif disorder_function == "variance":
            disorder_estimator = DecisionTree.variance
        len_data = len(data)
        if len_data == 0: return Node()
        current_disorder_level = disorder_estimator(data)
        # track enhancement of disorer's level
        best_enhancement = 0.0
        best_split = None
        best_split_sets = None
        # number columns
        nbr_coords = len(data[0].scaled_coords)
        for coord_ind in xrange(nbr_coords):
            # get unique values of the current column
            coord_values = {}
            for item in data:
                coord_values[item.scaled_coords[coord_ind]] = 1
            for coord_value in coord_values.iterkeys():
                set1, set2 = DecisionTree.divide_data(data, coord_ind, coord_value)
                p1 = float(len(set1)) / len_data
                p2 = (1 - p1)
                enhancement = current_disorder_level - (p1 * disorder_estimator(set1)) - (
                    p2 * disorder_estimator(set2))
                if (enhancement > best_enhancement) and (len(set1) > 0 and len(set2) > 0):
                    best_enhancement = enhancement
                    best_split = (coord_ind, coord_value)
                    best_split_sets = (set1, set2)
        if best_enhancement > 0:
            t_node = DecisionTree.build_tree(best_split_sets[0])
            f_node = DecisionTree.build_tree(best_split_sets[1])
            return Node(col=best_split[0], value=best_split[1],
                        t_node=t_node, f_node=f_node)
        else:
            return Node(results=DecisionTree.count_results(data))

    @staticmethod
    def prune(tree, min_enhancement, disorder_function="entropy"):
        """
        checking pairs of nodes that have a common parent to see if merging
        them would increase the entropy by less than a specified threshold
        """
        if disorder_function == "entropy":
            disorder_estimator = DecisionTree.entropy
        elif disorder_function == "gini_impurity":
            disorder_estimator = DecisionTree.gini_impurity
        elif disorder_function == "variance":
            disorder_estimator = DecisionTree.variance
        if tree.t_node.results == None:
            DecisionTree.prune(tree.t_node, min_enhancement)
        if tree.f_node.results == None:
            DecisionTree.prune(tree.f_node, min_enhancement)
        # If both the subbranches are now leaves, see if they should merged
        if (tree.t_node.results != None and tree.f_node.results != None):
            # Build a combined dataset
            t_node, f_node = [], []
            for key, value in tree.t_node.results.items():
                t_node += [[key]] * value
            for key, value in tree.f_node.results.items():
                f_node += [[key]] * value
            # Test the enhancement
            delta = disorder_estimator(t_node + f_node, item=False) - (
                disorder_estimator(t_node, item=False) + disorder_estimator(f_node,
                                                                            item=False) / 2)
            if delta < min_enhancement:
                # Merge the branches
                tree.t_node, tree.f_node = None, None
                tree.results = DecisionTree.count_results(t_node + f_node, item=False)

    @staticmethod
    def classify(observation, tree):
        """
        Classify a new observation given a decision tree
        """
        if tree.results != None:
            return tree.results
        # the observation value for the current criteria column
        observation_value = observation.scaled_coords[tree.col]
        if observation_value == None:
            t_results, f_results = DecisionTree.classify(observation,
                                                         tree.t_node), DecisionTree.classify(
                observation, tree.f_node)
            t_count = sum(t_results.values())
            f_count = sum(f_results.values())
            t_prob = float(t_count) / (t_count + f_count)
            f_prob = float(f_count) / (t_count + f_count)
            result = {}
            for key, value in t_results.items(): result[key] = value * t_prob
            for key, value in f_results.items(): result[key] = value * f_prob
            return result
        else:
            # with branch to follow
            branch = None
            if (isinstance(observation_value, int) or isinstance(observation_value, float)):
                branch = tree.t_node if (observation_value >= tree.value) else tree.f_node
            else:
                branch = tree.t_node if (observation_value == tree.value) else tree.f_node
            return DecisionTree.classify(observation, branch)
```

Finally the utilities to read the test and results :

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

d = DecisionTree()
node = d.build_tree(training_set)
correct = 0
for obs in classify_set:
    cr = d.classify(obs, node)
    if cr.keys()[0] == results[obs.id]:
        correct += 1
print(correct)
```
