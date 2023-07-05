---
layout: blog
title: "Quora Nearby."
subtitle: "Designing an algorithm to find topics or questions that are near to a given input location, up to a specified limit."
cover_image: images/blog/quora.png
cover_image_caption: "Quora"
tags: [code, python, machine-learning]
---

Topics on Quora have location data optionally associated with them, allowing the Quora Nearby feature for our iPhone app. Each question on Quora can have one or more topics associated with it. This feature allows us to display topics and questions near different locations, be it the user’s current location, or a specified location on the map.

![quora-feed-optimizer](/images/blog/quora-nearby.png)

Your task will be to write a program that will be able to find topics or questions that are near to a given input location, up to a specified limit.

Input format (read from STDIN):
The first line of input will be 3 integers: number of topics T, number of questions Q, and number of queries N.

There will be T lines following that, each with a topic id integer, and two doubles representing that topic’s location (you can consider the points to be located on a XY plane, location of a entity is in form of its x and y coordinate in the plane).

There will be Q lines following that, each with a question id integer and number of topics the question is associated with, Qn, followed by Qn number of topic ids associated with that question. These integers are guaranteed to be well-formed existing topics.  There may be 0 topics associated with a question, and in such cases, the question should never appear in the query results.

The next N lines will consist of a char, an integer and two doubles each, representing the type of results required (“t” for topic or “q” for question), the number of results required, and the location to be used as the query.

**Sample Input**:

```bash
3 6 2
0 0.0 0.0
1 1.0 1.0
2 2.0 2.0
0 1 0
1 2 0 1
2 3 0 1 2
3 0
4 0
5 2 1 2
t 2 0.0 0.0
q 5 100.0 100.0
```

**Output format (write to STDOUT)**:

For each query line given in the input, you are to output the ids of the nearest entities in ascending order of distance from the query location, up to the specified number of results.  When there is a tie between different entities in the ordering due to equal distances (threshold of 0.001 for equality comparisons), the entity with the larger id should be ranked first.

Distance of a question from a point is the minimum of the distance of all topics associated with that question.

Sample Output:

```bash
1
2
0 1
5 2 1 0
```


Explanation:

There are 3 topics with ids 0, 1 and 2. There are also 6 questions, with ids 0 to 5. We first ask a nearest topic query.

The closest 2 topics to (0.0, 0.0) are topics 0 and 1.

The next query asks for upto 5 closest questions to (100.0, 100.0). Since questions 5 and 2 are tagged with topic 2

located at (2.0, 2.0), they are closest to the query location.

Because of the tie in distance, we put question 5 before question 2.

The next closest question is question 1, followed by question 0.

We do not output questions 3 or 4 because there are no topics associated with them.

Constraints:

 * 1 <= T <= 10000
 * 1 <= Q <= 1000
 * 1 <= N <= 10000


Integer ids are between 0 and 100000 inclusive.

Number of topics associated with a question is not more than 10.

The number of results required for a query is not more than 100.

0.0 <= x,y <= 1000000.0 (10^6)

For the large testcases, all topic x,y co-ordinates will be approximately uniformly distributed over the bounds.

You should aim to have your algorithm be fast enough to solve our largest test inputs in under 5 seconds, or be as close to that as possible.

## My Solutions:

The first solution that I propose is a naive approach that sort all entities each time we make a query.

First we need to create the topic model, we should also be able to calculate the distance between this topic and the current origin, which is the query coordinates.

```python
class Topic(object):
    """
    Topic

    @type _id: int
    @param _id: the id of the topic

    @type _x: float
    @param _x: the x coordinate in the plane

    @type _y: float
    @param _y: the y coordinate in the plane

    @type _current_distance: float
    @param _current_distance: the current distance from the origin(origin being the query coordiantaes)

    @type _qn: int
    @param _qn: the number of questions associated with this topics

    @type _questions: list
    @param _questions: the list of the questions associated with this topic
    """

    def __init__(self, id, x, y):
        self._id = id
        self._x = x
        self._y = y
        self._current_distance = 0
        self._qn = 0
        self._questions = []

    def __gt__(self, topic):
        delta = self._current_distance - topic._current_distance
        if delta < -THRESHOLD:
            return True
        if delta > THRESHOLD:
            return False
        return True if self._id > topic._id else False

    def add(self, question):
        self._qn += 1
        self._questions.append(question)

    def get_questions(self, qn, questions):
        go_on = True
        for question in self._questions:
            if question not in questions:
                questions.append(question)
                qn -= 1
                if qn == 0:
                    go_on = False
                    break
        return sorted(questions, reverse=True), qn, go_on

    def set_current_distance(self, origin_x, origin_y):
        self._current_distance = self.euclidean_dis(self._x, self._y, origin_x, origin_y)

    @staticmethod
    def euclidean_dis(x1, y1, x2, y2):
        return math.sqrt(pow(x1 - x2, 2) + pow(y1 - y2, 2))
```

Next we need to create the utility that allows to add topics and questions, and process queries.

```python
class Nearby(object):
    """
    Nearby solver

    @type _tn: int
    @param _tn: the number of topics created

    @type _topics: dict
    @param _topics: the dictionary of topics created  
    """


    def __init__(self, tn):
        self._tn = tn
        self._topics = {}

    def add_topic(self, topic_id, x, y):
        self._topics[topic_id] = Topic(topic_id, x, y)

    def add_question(self, question, nbr_topics, topics):
        if nbr_topics <= 0:
            return
        for i in xrange(nbr_topics):
            topic_id = int(topics[i])
            self._topics[topic_id].add(question)

    def _process_query_topic(self, nbr_results, list_topics):
        if nbr_results > self._tn:
            nbr_results = self._tn
        return ' '.join([str(list_topics[i]._id) for i in xrange(nbr_results-1, -1, -1)])

    def _process_query_question(self, nbr_results, list_topics):
        results = []
        go_on = True
        for i in xrange(self._tn-1, -1, -1):
            results, nbr_results, go_on = list_topics[i].get_questions(nbr_results, results)
            if not go_on:
                break
        return ' '.join([str(x) for x in results])

    def process_query(self, q_type, q_nbr_results, q_x, q_y):
        list_topics = []
        for topic in self._topics.itervalues():
            topic.set_current_distance(q_x, q_y)
            heapq.heappush(list_topics, topic)
        if q_type == "t":
            return self._process_query_topic(q_nbr_results, list_topics)
        if q_type == "q":
            return self._process_query_question(q_nbr_results, list_topics)
```

Note that for performance purpose we are using the heap data-structure.

To go a little bit further we can easily think of a data structure to divide the space of topics, I created an easy to use data structure Square that represents a part of the plane

```python
class Square(object):
    """
    Square is data structure that represents a part of the plane.
    A square is divided to 4 parts.

    @type _origine_x: float
    @param _origine_x: the x coordinate of the origine of this square

    @type _origine_y: float
    @param _origine_y: the y coordinate of the origine of this square

    @type _curent_distance: int
    @param _curent_distance: current distance from the query coordiantes

    @type _tn: int
    @param _tn: numbre of points in this square  

    @type _topics: list
    @param _topics: list of topics
    """

    def __init__(self, origine_x, origine_y):
        self._origine_x = origine_x
        self._origine_y = origine_y
        self._current_distance = 0
        self._tn = 0
        self._topics = []

    def __gt__(self, square):
        delta = self._current_distance - square._current_distance
        if delta < 0:
            return True
        if delta > 0:
            return False

    def add(self, topic):
        self._tn += 1
        self._topics.append(topic)

    def set_current_distance(self, origin_x, origin_y):
        self._current_distance = Topic.euclidean_dis(self._origine_x, self._origine_y, origin_x, origin_y)
        for topic in self._topics:
            topic.set_current_distance(origin_x, origin_y)

    def get_topics(self, tn):
        if self._tn >= tn:
            return self._topics[:tn], 0, False
        else:
            return self._topics, tn-self._tn, True
```

This data structure will enable us to perform the sort part only on squares and not on all topics.

```python
class NearbySquare(Nearby):
    """
    Nearby solver using the square data structure

    @type _tn: int
    @param _tn: the number of topics created

    @type _topics: dict
    @param _topics: the dictionary of topics created

    @type _squares: dict
    @param _squares: the dictionary of squares created  
    """


    def __init__(self, tn):
        self._tn = tn
        self._ts = 0
        self._topics = {}
        self._squares = {}

    def add_topic(self, topic_id, x, y):
        topic = Topic(topic_id, x, y)
        self._topics[topic_id] = topic
        #locate which square this topic should go
        left_x = x % 10
        left_y = y % 10
        square_x = (x - left_x) + 5
        square_y = (y - left_y) + 5
        #check if this square exists
        try:
            square = self._squares[(square_x, square_y)]
        except:
            square = Square(square_x, square_y)
            self._squares[(square_x, square_y)] = square
            self._ts += 1
        square.add(topic)

    def _process_query_topic(self, nbr_results, list_squares):
        results = []
        go_on = True
        if nbr_results > self._tn:
            nbr_results = self._tn
        for i in xrange(self._ts-1, -1, -1):
            temp_results, nbr_results, go_on = list_squares[i].get_topics(nbr_results)
            results += temp_results
            if not go_on:
                break
        results = sorted(results, reverse=True)
        return ' '.join([str(result._id) for result in results])

    def _process_query_question(self, nbr_results, list_squares):
        results = []
        go_on = True
        for i in xrange(self._ts-1, -1, -1):
            for topic in sorted(list_squares[i]._topics, reverse=True):
                results, nbr_results, go_on = topic.get_questions(nbr_results, results)
                if not go_on:
                    break
            if not go_on:
                break
        return ' '.join([str(x) for x in results])

    def process_query(self, q_type, q_nbr_results, q_x, q_y):
        list_squares = []
        for square in self._squares.itervalues():
            square.set_current_distance(q_x, q_y)
            heapq.heappush(list_squares, square)
        if q_type == "t":
            return self._process_query_topic(q_nbr_results, list_squares)
        if q_type == "q":
        	return self._process_query_question(q_nbr_results, list_squares)
```

The complete solution:

```python
# -*- coding: utf-8 -*-
'''
Created on Jan 09, 2013

@author: Mourad Mourafiq

About: This is an attempt to solve the Quora challenge Nearby.
'''
import math
import heapq

THRESHOLD = 0.001
SQUARE_SIDE = 10


class Square(object):
    """
    Square is data structure that represents a part of the plane.
    A square is divided to 4 parts.

    @type _origine_x: float
    @param _origine_x: the x coordinate of the origine of this square

    @type _origine_y: float
    @param _origine_y: the y coordinate of the origine of this square

    @type _curent_distance: int
    @param _curent_distance: current distance from the query coordiantes

    @type _tn: int
    @param _tn: numbre of points in this square  

    @type _topics: list
    @param _topics: list of topics
    """

    def __init__(self, origine_x, origine_y):
        self._origine_x = origine_x
        self._origine_y = origine_y
        self._current_distance = 0
        self._tn = 0
        self._topics = []

    def __gt__(self, square):
        delta = self._current_distance - square._current_distance
        if delta < 0:
            return True
        if delta > 0:
            return False

    def add(self, topic):
        self._tn += 1
        self._topics.append(topic)

    def set_current_distance(self, origin_x, origin_y):
        self._current_distance = Topic.euclidean_dis(self._origine_x, self._origine_y, origin_x,
                                                     origin_y)
        for topic in self._topics:
            topic.set_current_distance(origin_x, origin_y)

    def get_topics(self, tn):
        if self._tn >= tn:
            return self._topics[:tn], 0, False
        else:
            return self._topics, tn - self._tn, True


class Topic(object):
    """
    Topic

    @type _id: int
    @param _id: the id of the topic

    @type _x: float
    @param _x: the x coordinate in the plane

    @type _y: float
    @param _y: the y coordinate in the plane

    @type _current_distance: float
    @param _current_distance: the current distance from the origin(origin being the query coordiantaes)

    @type _qn: int
    @param _qn: the number of questions associated with this topics

    @type _questions: list
    @param _questions: the list of the questions associated with this topic
    """

    def __init__(self, id, x, y):
        self._id = id
        self._x = x
        self._y = y
        self._current_distance = 0
        self._qn = 0
        self._questions = []

    def __gt__(self, topic):
        delta = self._current_distance - topic._current_distance
        if delta < -THRESHOLD:
            return True
        if delta > THRESHOLD:
            return False
        return True if self._id > topic._id else False

    def add(self, question):
        self._qn += 1
        self._questions.append(question)

    def get_questions(self, qn, questions):
        go_on = True
        for question in self._questions:
            if question not in questions:
                questions.append(question)
                qn -= 1
                if qn == 0:
                    go_on = False
                    break
        return sorted(questions, reverse=True), qn, go_on

    def set_current_distance(self, origin_x, origin_y):
        self._current_distance = self.euclidean_dis(self._x, self._y, origin_x, origin_y)

    @staticmethod
    def euclidean_dis(x1, y1, x2, y2):
        return math.sqrt(pow(x1 - x2, 2) + pow(y1 - y2, 2))


class Nearby(object):
    """
    Nearby solver

    @type _tn: int
    @param _tn: the number of topics created

    @type _topics: dict
    @param _topics: the dictionary of topics created  
    """

    def __init__(self, tn):
        self._tn = tn
        self._topics = {}

    def add_topic(self, topic_id, x, y):
        self._topics[topic_id] = Topic(topic_id, x, y)

    def add_question(self, question, nbr_topics, topics):
        if nbr_topics <= 0:
            return
        for i in xrange(nbr_topics):
            topic_id = int(topics[i])
            self._topics[topic_id].add(question)

    def _process_query_topic(self, nbr_results, list_topics):
        if nbr_results > self._tn:
            nbr_results = self._tn
        return ' '.join([str(list_topics[i]._id) for i in xrange(nbr_results - 1, -1, -1)])

    def _process_query_question(self, nbr_results, list_topics):
        results = []
        go_on = True
        for i in xrange(self._tn - 1, -1, -1):
            results, nbr_results, go_on = list_topics[i].get_questions(nbr_results, results)
            if not go_on:
                break
        return ' '.join([str(x) for x in results])

    def process_query(self, q_type, q_nbr_results, q_x, q_y):
        list_topics = []
        for topic in self._topics.itervalues():
            topic.set_current_distance(q_x, q_y)
            heapq.heappush(list_topics, topic)
        if q_type == "t":
            return self._process_query_topic(q_nbr_results, list_topics)
        if q_type == "q":
            return self._process_query_question(q_nbr_results, list_topics)


class NearbySquare(Nearby):
    """
    Nearby solver using the square data structure

    @type _tn: int
    @param _tn: the number of topics created

    @type _topics: dict
    @param _topics: the dictionary of topics created

    @type _squares: dict
    @param _squares: the dictionary of squares created  
    """

    def __init__(self, tn):
        self._tn = tn
        self._ts = 0
        self._topics = {}
        self._squares = {}

    def add_topic(self, topic_id, x, y):
        topic = Topic(topic_id, x, y)
        self._topics[topic_id] = topic
        # locate which square this topic should go
        left_x = x % 10
        left_y = y % 10
        square_x = (x - left_x) + 5
        square_y = (y - left_y) + 5
        # check if this square exists
        try:
            square = self._squares[(square_x, square_y)]
        except:
            square = Square(square_x, square_y)
            self._squares[(square_x, square_y)] = square
            self._ts += 1
        square.add(topic)

    def _process_query_topic(self, nbr_results, list_squares):
        results = []
        go_on = True
        if nbr_results > self._tn:
            nbr_results = self._tn
        for i in xrange(self._ts - 1, -1, -1):
            temp_results, nbr_results, go_on = list_squares[i].get_topics(nbr_results)
            results += temp_results
            if not go_on:
                break
        results = sorted(results, reverse=True)
        return ' '.join([str(result._id) for result in results])

    def _process_query_question(self, nbr_results, list_squares):
        results = []
        go_on = True
        for i in xrange(self._ts - 1, -1, -1):
            for topic in sorted(list_squares[i]._topics, reverse=True):
                results, nbr_results, go_on = topic.get_questions(nbr_results, results)
                if not go_on:
                    break
            if not go_on:
                break
        return ' '.join([str(x) for x in results])

    def process_query(self, q_type, q_nbr_results, q_x, q_y):
        list_squares = []
        for square in self._squares.itervalues():
            square.set_current_distance(q_x, q_y)
            heapq.heappush(list_squares, square)
        if q_type == "t":
            return self._process_query_topic(q_nbr_results, list_squares)
        if q_type == "q":
            return self._process_query_question(q_nbr_results, list_squares)


T, Q, N = [int(x) for x in raw_input().split()]
nearby = NearbySquare(T)
while (T):  # list of topics
    command = raw_input().split()
    nearby.add_topic(int(command[0]), float(command[1]), float(command[2]))
    T -= 1
while (Q):  # list of questions
    command = raw_input().split()
    nearby.add_question(int(command[0]), int(command[1]), command[2:])
    Q -= 1
while (N):  # process queries
    command = raw_input().split()
    print(nearby.process_query(command[0], int(command[1]), float(command[2]), float(command[3])))
    N -= 1
```
