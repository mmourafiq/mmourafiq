---
layout: blog
title: "Quora Feed Optimizer."
subtitle: "Designing an algorithm that picks the stories to display in the feed."
cover_image: images/blog/quora.png
cover_image_caption: "Quora"
tags: [ code, python, machine-learning ]
---

For those not familiar with the Feed Optimizer challenge here is a rundown of the problem:

![quora-feed-optimizer](/images/blog/quora-feed-optimizer.png)

Quora shows a customized feed of recent stories on a user’s home page. Stories in Quora refer to activities that happen on the site, for example, when a user posts a note, adds a question, or upvotes an answer. We score each story based on its type and other characteristics, and this score represents the value we think a story brings to the user. We want to be able to generate quickly the feed of best and most recent stories for the user every time they reload their home page.

Quora shows a customized feed of recent stories on a user’s home page. Stories in Quora refer to activities that happen on the site, for example, when a user posts a note, adds a question, or upvotes an answer. We score each story based on its type and other characteristics, and this score represents the value we think a story brings to the user. We want to be able to generate quickly the feed of best and most recent stories for the user every time they reload their home page.

Your task will be to design the algorithm that picks the stories to display in the feed.

You are given a list of stories, each having a time of publication, a score and a certain height in pixels that it takes to display the story. Given the total number of pixels in the browser available for displaying the feed, you want to maximize the sum of scores for the stories that you can display in the feed at each time the user reloads their home page. We only want to consider recent stories, so only stories that were published in a recent time window from the time of reload should be considered. You do not have to use up all the pixels in the browser.

**Input format (read from STDIN)**:

The first line of input will be 3 positive integers: N the number of events, W the time window representing the window of recent stories, and H the height of the browser in pixels.

There will be N lines following that, each beginning with ‘S’ if it is a story event, or ‘R’ if it is a reload event.  A story event will have 3 positive integers: the time of publication, the score and the height in pixels of that story. A reload event will have 1 positive integer: the time of reload.

The events will always be in chronological order, and no two events will happen at the same time.

For example:

```bash
9 10 100
S 11 50 30
R 12
S 13 40 20
S 14 45 40
R 15
R 16
S 18 45 20
R 21
R 22
```

**Output format (write to STDOUT)**:

For each reload event given in the input, you are to output a line of integers. First, the maximum score of stories you can show in the feed. This should be followed by the number of stories picked and the id number for each story picked, in increasing order. Stories are given an id starting from 1 in the order of their time of publication.
If two sets of stories give the same score, choose the set with fewer stories. If there is still a tie, choose the set which has the lexicographically smaller set of ids, e.g. choose [1, 2, 5] over [1, 3, 4].

For example:

```bash
50 1 1
135 3 1 2 3
135 3 1 2 3
140 3 1 3 4
130 3 2 3 4
```

**Explanation**:

There are 4 stories (with ids 1 to 4) and 5 reload events. At the first reload, there is only one story with score of 50 available for display. The next two reloads, we have 3 stories that take up 90 of the 100 pixels available, for a total score of 135. When we reload at time 21, there are 4 stories available for choosing, but only 3 will fit into the browser height. The best set is [1, 3, 4] for a total score of 140. At the last reload event, we no longer consider story 1 when choosing stories because it is more than 10 time units old. The best set is thus [2, 3, 4].

**Constraints**:

All times are positive integers up to 1,000,000,000.

All scores are positive integers up to 1,000,000.

All heights are positive integers.

0 < N <= 10,000

0 < H <= 2,000

0 < W <= 2,000

You should aim to have your algorithm be fast enough to solve our largest test inputs in under 5 seconds, or be as close to that as possible.

**Notes**:

For this contest, we are considering a smaller-scale version of our feed optimization problem with simplified constraints. (The actual feed on our site uses a different algorithm.)

## Solution 1 (backtracking method):

It Checks all possibilities:

  * 1. best solution for combination of 2 stories (if it exists).
  * 2. best solution for combination of 3 stories (if it exists).
  * .
  * .
  * l-1. best solution for combination of l-1 stories (if it exists), l : being the length of the current stories.

```python
def brute_force(self):
    """
    check all possibilities:
    1) best solution for combination of 2 stories (if it exists).
    2) best solution for combination of 3 stories (if it exists).
    .
    .
    l-1) best solution for combination of l-1 stories (if it exists).
    l : being the length of the current stories.
    """
    best_solution = Solution()
    best_solution.add(self._best_story)
    for i in xrange(2, self._len_stories+1):
        for tuple_stories in itertools.combinations(self._stories, i):
            if self.addable(tuple_stories):
                current_solution = Solution()
                for story in tuple_stories:
                    current_solution.add(story)
                if current_solution > best_solution:
                    best_solution = current_solution
    return best_solution
```

## Solution 2 (using the [annealing simulated algorithm](http://en.wikipedia.org/wiki/Simulated_annealing)):

Performs the annealing simulated algorithm (Simulated annealing (SA) is a generic probabilistic metaheuristic for the global optimization problem of locating a good approximation to the global optimum of a given function in a large search space):

 * 1. start with a random solution.
 * 2. move to a neighbour solution. (favors better solutions, and accepts worst solutions with a certain probabilitie
to avoid local minimum until the temperature is totally down)

```python
def annealing_simulated(self, T=1000.0,cool=0.35):
    """
    perform the annealing simulated algorithm:
    1) start with a random solution.
    2) move to a neighbour solution.
    (favors better solutions, and accepts worst solutions with a certain probabilities
    to avoid local minimum until the temperature is totally down)
    """
    #order stories based on their proportioned score
    ordered_stories = sorted(self._stories, reverse=True)
    #produce a random solution
    current_solution, stories_in_current = self.random_solution(ordered_stories, self._len_stories)
    best_solution = Solution.clone(current_solution)
    while(T>0.1):
        temp_solution = Solution.clone(current_solution)
        stories_in_temp = copy.copy(stories_in_current)
        stories_at_true = [i for i in xrange(self._len_stories) if stories_in_temp[i]]
        #check if there is still stories
        if len(stories_at_true) == self._len_stories:
            break
        #choose a story and remove it
        if stories_at_true:
            indice = choice(stories_at_true)
            stories_in_temp[indice] = False
            temp_solution.remove(ordered_stories[indice])
        else:
            indice = -1
        #add any number of other stories available
        for i in xrange(indice+1, self._len_stories):
            if stories_in_temp[i]:
                continue
            story = ordered_stories[i]
            if self.addable((story,), temp_solution):
                stories_in_temp[i] = True
                temp_solution.add(story)
            elif temp_solution._height == self.__height:
                break
        #compare temp and current solutions
        if temp_solution > current_solution:
            current_solution = temp_solution
            stories_in_current = stories_in_temp
            #also since temp is better than current, compare it to best
            if current_solution > best_solution:
                best_solution = Solution.clone(current_solution)
        #current solution is better than temp
        #the algorithm states that we can still give it a try depending on a probability
        else:
            #since temp solution score is < current solution score
            #this probability will be near one at the beginning where T is high
            #but will get lower and lower as T cool down
            #hence will accept less and less bad solution
            p = pow(math.e,float(temp_solution._score - current_solution._score)/T)
            if p > random():
                current_solution = temp_solution
                stories_in_current = stories_in_temp
        #decrease the temperature
        T=T*cool
    return best_solution
```

So before we apply this algorithms, we need first to model a story:

```python
class Story(object):
    """
    Story object

    @type _cpt: int
    @param _cpt: counts the number of instance created.

    @type _height: int
    @param _height: The stroy's height.

    @type _time: int
    @param _time: The time of publication.

    @type _id: int
    @param _id: The story's id.

    @type _score: int
    @param _score: The story's score.

    @type _height: int
    @param _height: The stroy's height.

    @type _proportioned_score: float
    @param _proportioned_score: The stroy's _score proportioned to height.
    """
    __cpt = 0

    def __init__(self, time=-1, score=-1, height=-1):
        self._id = Story.__cpt
        self._time = time
        self._score = score
        self._height = height
        self._proportioned_score = float(score)/height
        Story.__cpt += 1

    def __repr__(self):
        return "id: %s, time: %s" % (self._id, self._time)

    def __gt__(self, story):
        if(self._proportioned_score > story._proportioned_score):
            return True
        if(self._proportioned_score < story._proportioned_score):
            return False
        if(self._id < story._id):
            return True
        return False

    def _better_score(self, story):
        if(self._score > story._score):
            return True
        if(self._score < story._score):
            return False
        if(self._id < story._id):
            return True
        return False
```

Then to model a solution:

```python
class Solution(object):
    """
    Potential solution for the upcoming reload

    @type _stories: list
    @param _stories: The list of potential items.

    @type _len_stories : int
    @param _len_stories: The length of the list of stories.

    @type _score: int
    @param _score: The current solution's score.

    @type _height: int
    @param _height: The current solution's height.
    """

    def __init__(self):
        self._stories = []
        self._len_stories = 0
        self._score = 0
        self._height = 0

    def __repr__(self):
        return "%s %s %s" % (self._score, self._len_stories, ' '.join(sorted([str(story._id) for story in self._stories])))

    def __gt__(self, solution):
        #check who's score is better
        if self._score > solution._score:
            return True
        if self._score < solution._score:
            return False
        #same score; check who has less stories
        if self._len_stories < solution._len_stories:
            return True
        if self._len_stories > solution._len_stories:
            return False
        #same score, same number of stories; check who has smaller lexicographically
        if sorted([story._id for story in self._stories]) <= sorted([story._id for story in solution._stories]):
            return True
        else:
            return False

    @classmethod
    def clone(cls, solution):
        clone_solution = cls()
        clone_solution._stories = copy.copy(solution._stories)
        clone_solution._len_stories = solution._len_stories
        clone_solution._score = solution._score
        clone_solution._height = solution._height
        return clone_solution

    def add(self, story):
        """
        add story to the solution
        """
        self._stories.append(story)
        self._score += story._score
        self._height += story._height
        self._len_stories += 1

    def remove(self, story):
        """
        remove story from the solution
        """
        self._stories.remove(story)
        self._score -= story._score
        self._height -= story._height
        self._len_stories -= 1  
```

Finally, an optimizer that should keep track of all stories, purge old ones and perform an algorithm to produce a solution:

```python
# -*- coding: utf-8 -*-
'''
Created on Jan 11, 2013

@author: Mourad Mourafiq

About: This is an attempt to solve the Quora challenge Feed Optimizer.
'''
import itertools
import copy
import math
from random import choice, random

BRUTE_FORCE = 1
ANNEALING_SIMULATED = 2


class Story(object):
    """
    Story object

    @type _cpt: int
    @param _cpt: counts the number of instance created.

    @type _height: int
    @param _height: The stroy's height.

    @type _time: int
    @param _time: The time of publication.

    @type _id: int
    @param _id: The story's id.

    @type _score: int
    @param _score: The story's score.

    @type _height: int
    @param _height: The stroy's height.

    @type _proportioned_score: float
    @param _proportioned_score: The stroy's _score proportioned to height.
    """
    __cpt = 0

    def __init__(self, time=-1, score=-1, height=-1):
        self._id = Story.__cpt
        self._time = time
        self._score = score
        self._height = height
        self._proportioned_score = float(score) / height
        Story.__cpt += 1

    def __repr__(self):
        return "id: %s, time: %s" % (self._id, self._time)

    def __gt__(self, story):
        if (self._proportioned_score > story._proportioned_score):
            return True
        if (self._proportioned_score < story._proportioned_score):
            return False
        if (self._id < story._id):
            return True
        return False

    def _better_score(self, story):
        if (self._score > story._score):
            return True
        if (self._score < story._score):
            return False
        if (self._id < story._id):
            return True
        return False


class Solution(object):
    """
    Potential solution for the upcoming reload

    @type _stories: list
    @param _stories: The list of potential items.

    @type _len_stories : int
    @param _len_stories: The length of the list of stories.

    @type _score: int
    @param _score: The current solution's score.

    @type _height: int
    @param _height: The current solution's height.
    """

    def __init__(self):
        self._stories = []
        self._len_stories = 0
        self._score = 0
        self._height = 0

    def __repr__(self):
        return "%s %s %s" % (self._score, self._len_stories,
                             ' '.join(sorted([str(story._id) for story in self._stories])))

    def __gt__(self, solution):
        # check who's score is better
        if self._score > solution._score:
            return True
        if self._score < solution._score:
            return False
        # same score; check who has less stories
        if self._len_stories < solution._len_stories:
            return True
        if self._len_stories > solution._len_stories:
            return False
        # same score, same number of stories; check who has smaller lexicographically
        if sorted([story._id for story in self._stories]) <= sorted(
            [story._id for story in solution._stories]):
            return True
        else:
            return False

    @classmethod
    def clone(cls, solution):
        clone_solution = cls()
        clone_solution._stories = copy.copy(solution._stories)
        clone_solution._len_stories = solution._len_stories
        clone_solution._score = solution._score
        clone_solution._height = solution._height
        return clone_solution

    def add(self, story):
        """
        add story to the solution
        """
        self._stories.append(story)
        self._score += story._score
        self._height += story._height
        self._len_stories += 1

    def remove(self, story):
        """
        remove story from the solution
        """
        self._stories.remove(story)
        self._score -= story._score
        self._height -= story._height
        self._len_stories -= 1


class Optimizer(object):
    """
    Keep track of stories that can potentially make a solution.
    The stories should be sorted by time of publication.

    @type _stories: list
    @param stories: The list of stories that can potentially make a solution.

    @type _len_stories : int
    @param _len_stories: The length of the list of stories.

    @type __height: int
    @param window: The height of the browser.  

    @type __window: int
    @param window: The window of recent stories.

    @type _best_story: Stroy
    @param _best_story: The  best story so far.
    """
    __height = 0
    __window = 0

    def __init__(self, window, height):
        self._stories = []
        self._len_stories = 0
        Optimizer.__window = window
        Optimizer.__height = height
        self._best_story = Story()

    def _purge_old_stories(self, current_time):
        """
        remove old stories form the current list of stories
        """
        # check if the oldest stories can still be part of the solution
        to_be_removed = []
        for old_story in self._stories:
            if (current_time - old_story._time) <= Optimizer.__window:
                break
            else:
                to_be_removed.append(old_story)
        for old_story in to_be_removed:
            self._stories.remove(old_story)
            self._len_stories -= 1

    def _brute_force(self):
        """
        check all possibilities:
            1) best solution for combination of 2 stories (if it exists).
            2) best solution for combination of 3 stories (if it exists).
            .
            .
            l-1) best solution for combination of l-1 stories (if it exists).

            l : being the length of the current stories.
        """
        best_solution = Solution()
        best_solution.add(self._best_story)
        for i in xrange(2, self._len_stories + 1):
            for tuple_stories in itertools.combinations(self._stories, i):
                if self.addable(tuple_stories):
                    current_solution = Solution()
                    for story in tuple_stories:
                        current_solution.add(story)
                    if current_solution > best_solution:
                        best_solution = current_solution
        return best_solution

    def _annealing_simulated(self, T=1000.0, cool=0.35):
        """
        perform the annealing simulated algorithm:
            1) start with a random solution.
            2) move to a neighbour solution.
                (favors better solutions, and accepts worst solutions with a certain probabilities
                 to avoid local minimum until the temperature is totally down)
        """
        # order stories based on their proportioned score
        ordered_stories = sorted(self._stories, reverse=True)
        # produce a random solution
        current_solution, stories_in_current = self.random_solution(ordered_stories,
                                                                    self._len_stories)
        best_solution = Solution.clone(current_solution)
        while (T > 0.1):
            temp_solution = Solution.clone(current_solution)
            stories_in_temp = copy.copy(stories_in_current)
            stories_at_true = [i for i in xrange(self._len_stories) if stories_in_temp[i]]
            # check if there is still stories
            if len(stories_at_true) == self._len_stories:
                break
            # choose a story and remove it
            if stories_at_true:
                indice = choice(stories_at_true)
                stories_in_temp[indice] = False
                temp_solution.remove(ordered_stories[indice])
            else:
                indice = -1
            # add any number of other stories available
            for i in xrange(indice + 1, self._len_stories):
                if stories_in_temp[i]:
                    continue
                story = ordered_stories[i]
                if self.addable((story,), temp_solution):
                    stories_in_temp[i] = True
                    temp_solution.add(story)
                elif temp_solution._height == self.__height:
                    break
            # compare temp and current solutions
            if temp_solution > current_solution:
                current_solution = temp_solution
                stories_in_current = stories_in_temp
                # also since temp is better than current, compare it to best
                if current_solution > best_solution:
                    best_solution = Solution.clone(current_solution)
            # current solution is better than temp
            # the algorithm states that we can still give it a try depending on a probability
            else:
                # since temp solution score is < current solution score
                # this probability will be near one at the beginning where T is high
                # but will get lower and lower as T cool down
                # hence will accept less and less bad solution
                p = pow(math.e, float(temp_solution._score - current_solution._score) / T)
                if p > random():
                    current_solution = temp_solution
                    stories_in_current = stories_in_temp
            # decrease the temperature
            T = T * cool
        return best_solution

    def add(self, story):
        # check if the story's height is within the browser's height
        if story._height <= Optimizer.__height:
            self._stories.append(story)
            self._len_stories += 1
            if (story > self._best_story):
                self._best_story = story

    def produce_solution(self, current_time, solution=BRUTE_FORCE):
        self._purge_old_stories(current_time)
        if solution == BRUTE_FORCE:
            return self._brute_force()
        elif solution == ANNEALING_SIMULATED:
            return self._annealing_simulated()

    @classmethod
    def addable(cls, tuple_stories, solution=Solution()):
        total_height = solution._height
        for story in tuple_stories:
            total_height += story._height
        if total_height <= cls.__height:
            return True
        return False

    @classmethod
    def random_solution(cls, list_stories, length_stories):
        """
        produce a random solution
        """
        stories_in = [False] * length_stories
        solution = Solution()
        for i in xrange(length_stories):
            story = list_stories[i]
            if cls.addable((story,), solution):
                solution.add(story)
                stories_in[i] = True
            elif solution._height == cls.__height:
                break
        return solution, stories_in


N, W, H = [int(x) for x in raw_input().split()]
p = Optimizer(W, H)
while (N):
    command = raw_input().split()
    if command[0] == "S":  # story
        t, s, h = [int(x) for x in command[1:]]
        p.add(Story(t, s, h))
    elif command[0] == "R":  # Reload
        tr = int(command[1])
        print(p.produce_solution(tr, solution=ANNEALING_SIMULATED))
    N -= 1  
```
