---
layout: blog
title: "Quora Typeahead search."
subtitle: "Designing an algorithm to return the most relevant results that match the search query entered into the input text field."
cover_image: images/blog/quora.png
cover_image_caption: "Quora"
tags: [ code, python, machine-learning ]
---

This is an attempt to solve the quora typeahead challenge.

![quora-typeahead-search](/images/blog/quora-type-ahead.png)

The search bar at the top of every page on Quora allows you to search the most up-to-date people,
topics and questions on our site.

We want to quickly return the most relevant results that match the search query entered into the
input text field. Every time a new user, question or topic is added (or old ones deleted),
subsequent queries must reflect those changes immediately. We currently use a fast in-memory service
to handle this.

Input comes into the service as the following commands:

* **ADD <type> <id> <score> <data string that can contain spaces>**

Adds the following <type> of item (user | topic | question | board) with the unique <id> string
and <score> float, corresponding to the <data string> that would be used to match queries.

* **DEL <id>**

Deletes the item specified by unique identifier <id>.

* **QUERY <number of results> <query string that can contain spaces>**

Queries for the specified integer number of results (up to 20) that match a given query string. For
a data item to be considered a matching result to a query, each token in the query must be found in
the data string as a case-insensitive prefix to any token in the data string. The results are ranked
in descending order of score, and we only take the top few results as specified. When there is a tie
in the score, newer items (more recently added) should be ranked higher. If there are no results,
just output the empty string on that line.

* *
  *WQUERY <number of results> <number of boosts> (<type>:<boost>)* (<id>:<boost>)* <query string that can contain spaces>**

Same as QUERY, except that instead of using the raw input score specified by ADD for the particular
item, the scores are weighted by the optional number of boosting factors. The boosts are floats that
should be multiplied to the raw score, and affect either whole types, or specific items with the
given <id>s. If there are multiple boosts applicable, they each are multiplied commutatively to the
raw score.

Your task will be to write an equivalent service as a standalone program, with input files that
correspond to the queries and updates to the data, and expected output files that correspond to the
results obtained for each query.

**Input format (read from STDIN)**:

Your program will be given an integer N on the first line of stdin, followed by N lines of the form:

```bash
<command> <command data>
```

The input commands are: ADD | DEL | QUERY | WQUERY

Types are: user | topic | question | board

Ids are alphanumeric strings without spaces or punctuation and will not include the same strings
used for types.

Data strings can be any ASCII character, but are delimited by spaces or tabs. We will not be using
anything special unprintable characters or \r and \n in the data string.

Command data for each command is as specified above. For example:

```bash
15
ADD user u1 1.0 Adam D’Angelo
ADD user u2 1.0 Adam Black
ADD topic t1 0.8 Adam D’Angelo
ADD question q1 0.5 What does Adam D’Angelo do at Quora?
ADD question q2 0.5 How did Adam D’Angelo learn programming?
QUERY 10 Adam
QUERY 10 Adam D’A
QUERY 10 Adam Cheever
QUERY 10 LEARN how
QUERY 1 lear H
QUERY 0 lea
WQUERY 10 0 Adam D’A
WQUERY 2 1 topic:9.99 Adam D’A
DEL u2
QUERY 2 Adam
```

**Output format (write to STDOUT)**:

For each QUERY and WQUERY command, you should output the following line:

<sorted result ids>

Where each line contains the <id>s in descending score order, up to the required number of results,
as specified above according to the QUERY and WQUERY commands. If there are no matches, output the
empty line. For example:

```bash
u2 u1 t1 q2 q1
u1 t1 q2 q1

q2
q2

u1 t1 q2 q1
t1 u1
u1 t1
```

**Constraints**:

* 0 < N < 100000
* 0 < number of ADDs < 40000
* 0 < number of DELs < 10000
* 0 < number of QUERYs < 20000
* 0 < number of WQUERYs < 1000
* 0 < number of boosts < 25
* 0.0 < each score < 100.0
* 0 < data string length < 100 chars

You should aim to have your algorithm be fast enough to solve our largest test inputs in under 5
seconds, or be as close to that as possible.

**Notes**:

In this problem, we are presenting a representative simplified version of our service for the
purposes of the contest.

##Solution:

Before we dive into the solution we need to process the entries. We can use the regular expression
lib from python, to match, validate and extract informations from the inputs:

```python
COMMANDS = "(ADD)|(DEL)|(W?QUERY)"
ANY_STRING = "(\\S*.*)"
SEPARATORS = "(?: |\\t)"
IDS = "\\w+"
TYPES = "user|topic|question|board"
FLOATS = "[0-9]+(?:.[0-9]*)?"
INTS = "[0-9]+"
BOOSTS = "((?:" + TYPES + "|(?:" + IDS + ")):" + FLOATS + SEPARATORS + ")*"
ANY_COMMAND = "(?P<command>" + COMMANDS + ")" + SEPARATORS + "(?P<parameters>" + ANY_STRING + ")"
ADD_COMMAND = "(?P<type>" + TYPES + ")" + SEPARATORS +
              "(?P<id>" + IDS + ")" + SEPARATORS +
              "(?P<score>" + FLOATS + ")" + SEPARATORS +
              "(?P<content>" + ANY_STRING + ")"
DEL_COMMAND = "(?P<id>" + IDS + ")"
QUERY_COMMAND = "(?P<nbr_results>" + INTS + ")" + SEPARATORS +
                "(?P<query>" + ANY_STRING + ")"
WQUERY_COMMAND = "(?P<nbr_results>" + INTS + ")" + SEPARATORS +
                 "(?P<nbr_boosts>" + INTS + ")" + SEPARATORS +
                 "(?P<boosts>" + BOOSTS + ")" +
                 "(?P<query>" + ANY_STRING + ")"
COMMAND_MATCHER = re.compile(ANY_COMMAND)
ADD_COMMAND_MATCHER = re.compile(ADD_COMMAND)
DEL_COMMAND_MATCHER = re.compile(DEL_COMMAND)
QUERY_COMMAND_MATCHER = re.compile(QUERY_COMMAND)
WQUERY_COMMAND_MATCHER = re.compile(WQUERY_COMMAND)
```

So, now that we are able to process inputs, we can move to the next phase, modelling the items:

```python
class Item(object):
    """
    either a topic, a user, a board or a question

    @type type: str
    @param type: The item's type.

    @type id: str
    @param id: The item's id.

    @type score: float
    @param score: The item's score.

    @type content: str
    @param contetn: The item's content.

    @type time: time
    @param time: The item's time of creation.
    """

    def __init__(self, type, id, score, content):
        self.type = type
        self.id = id
        self.score = float(score)
        self.content = content
        self.time = datetime.datetime.now()

    def __repr__(self):
        return self.id

    def better_than(self, item):
        """
        compare the current item to the input item.
        follows this method:
            . highest score goes first.
            . same score; time FIFO.
        return true if the current item is better than the input, otherwise returns false
        """
        if (self.score > item.score):
            return True
        if (self.score < item.score):
            return False
        return True if (self.time > item.time) else False
```

Our solution should provide 4 services adding, deleting, querying and w-querying. And since we are
dealing with strings as entries, we need a data structure that is practical for inserting and
retrieving. The radix trees seems like a good choice and are practical in this kind of situations.

```python
class Prefixer():

    def __init__(self):
        self.__data = {}

    def __repr__(self):
        return 'Prefixer(%s)' % (self.__data,)

    def __eq__(self, other):
        return self.__data == other.__data

    def get_data(self):
        return self.__data

    def insert(self, word, item_id):
        node = self.__data
        while word:
            prefix, key = self.longest_prefix(word, node.keys())
            if not prefix:
                break
            len_prefix = len(prefix)
            if prefix != key:
                # split key into prefix:suffix, move data
                suffix = key[len_prefix:]
                current_node = node[key]
                node[prefix] = {suffix: current_node}
                del node[key]
            word = word[len_prefix:]
            node = node[prefix]
        if word:
            node[word] = eval(NOK)
            node[word][''].append(item_id)
        else:
            try:
                node[word].append(item_id)
            except:
                node[word] = []
                node[word].append(item_id)
        return True

    def remove(self, word, item_id):
        node = self.__data
        while word:
            prefix, key = self.longest_prefix(word, node.keys())
            if not prefix:
                return False
            node = node.get(prefix, None)
            if not node:
                return False
            word = word[len(prefix):]
        try:
            node[''].remove(item_id)
            return True
        except:
            return False

    def _search_dico(self, word):
        node = self.__data
        while word:
            prefix, key = self.longest_prefix(word, node.keys())
            if not prefix:
                return False
            if not key:
                return False
            if prefix != key:
                if prefix == word:
                    return node[key]
                else:
                    return False
            node = node[prefix]
            word = word[len(prefix):]
        return node

    def search(self, word):
        dico = self._search_dico(word)
        if dico != False:
            return self.traverse_dico(dico)
        return []

    @staticmethod
    def traverse_dico(dico):
        results = []
        for key, value in dico.iteritems():
            if key == '':
                results += value
            else:
                results += Prefixer.traverse_dico(value)
        return results

    @staticmethod
    def longest_prefix(word, candidates):
        """
        return the longest prefix match between word and any of the
        candidates, if any. Only one candidate will much.
        """
        if word:
            wc = word[0]
            for c in candidates:
                if c.startswith(wc):
                    for i in reversed(xrange(1, min(len(word), len(c)) + 1)):
                        if c.startswith(word[:i]):
                            return (word[:i], c)
        return ('', None)
```

Finally, we put all this together:

```python
# -*- coding: utf-8 -*-
'''
Created on Jan 04, 2013

@author: Mourad Mourafiq

About: This is an attempt to solve the Quora challenge Typehead.
'''
import re
import copy
import datetime

COMMANDS = "(ADD)|(DEL)|(W?QUERY)"
ANY_STRING = "(\\S*.*)"
SEPARATORS = "(?: |\\t)"
IDS = "\\w+"
TYPES = "user|topic|question|board"
FLOATS = "[0-9]+(?:.[0-9]*)?"
INTS = "[0-9]+"
BOOSTS = "((?:" + TYPES + "|(?:" + IDS + ")):" + FLOATS + SEPARATORS + ")*"
ANY_COMMAND = "(?P<command>" + COMMANDS + ")" + SEPARATORS + "(?P<parameters>" + ANY_STRING + ")"
ADD_COMMAND = "(?P<type>" + TYPES + ")" + SEPARATORS +
"(?P<id>" + IDS + ")" + SEPARATORS +
"(?P<score>" + FLOATS + ")" + SEPARATORS +
"(?P<content>" + ANY_STRING + ")"
DEL_COMMAND = "(?P<id>" + IDS + ")"
QUERY_COMMAND = "(?P<nbr_results>" + INTS + ")" + SEPARATORS +
"(?P<query>" + ANY_STRING + ")"
WQUERY_COMMAND = "(?P<nbr_results>" + INTS + ")" + SEPARATORS +
"(?P<nbr_boosts>" + INTS + ")" + SEPARATORS +
"(?P<boosts>" + BOOSTS + ")" +
"(?P<query>" + ANY_STRING + ")"
COMMAND_MATCHER = re.compile(ANY_COMMAND)
ADD_COMMAND_MATCHER = re.compile(ADD_COMMAND)
DEL_COMMAND_MATCHER = re.compile(DEL_COMMAND)
QUERY_COMMAND_MATCHER = re.compile(QUERY_COMMAND)
WQUERY_COMMAND_MATCHER = re.compile(WQUERY_COMMAND)

NOK = "{'':[]}"


class Prefixer():

    def __init__(self):
        self.__data = {}

    def __repr__(self):
        return 'Prefixer(%s)' % (self.__data,)

    def __eq__(self, other):
        return self.__data == other.__data

    def get_data(self):
        return self.__data

    def insert(self, word, item_id):
        node = self.__data
        while word:
            prefix, key = self.longest_prefix(word, node.keys())
            if not prefix:
                break
            len_prefix = len(prefix)
            if prefix != key:
                # split key into prefix:suffix, move data
                suffix = key[len_prefix:]
                current_node = node[key]
                node[prefix] = {suffix: current_node}
                del node[key]
            word = word[len_prefix:]
            node = node[prefix]
        if word:
            node[word] = eval(NOK)
            node[word][''].append(item_id)
        else:
            try:
                node[word].append(item_id)
            except:
                node[word] = []
                node[word].append(item_id)
        return True

    def remove(self, word, item_id):
        node = self.__data
        while word:
            prefix, key = self.longest_prefix(word, node.keys())
            if not prefix:
                return False
            node = node.get(prefix, None)
            if not node:
                return False
            word = word[len(prefix):]
        try:
            node[''].remove(item_id)
            return True
        except:
            return False

    def _search_dico(self, word):
        node = self.__data
        while word:
            prefix, key = self.longest_prefix(word, node.keys())
            if not prefix:
                return False
            if not key:
                return False
            if prefix != key:
                if prefix == word:
                    return node[key]
                else:
                    return False
            node = node[prefix]
            word = word[len(prefix):]
        return node

    def search(self, word):
        dico = self._search_dico(word)
        if dico != False:
            return self.traverse_dico(dico)
        return []

    @staticmethod
    def traverse_dico(dico):
        results = []
        for key, value in dico.iteritems():
            if key == '':
                results += value
            else:
                results += Prefixer.traverse_dico(value)
        return results

    @staticmethod
    def longest_prefix(word, candidates):
        """
        return the longest prefix match between word and any of the
        candidates, if any. Only one candidate will much.
        """
        if word:
            wc = word[0]
            for c in candidates:
                if c.startswith(wc):
                    for i in reversed(xrange(1, min(len(word), len(c)) + 1)):
                        if c.startswith(word[:i]):
                            return (word[:i], c)
        return ('', None)


class TypeHead(object):
    """
    typehead object that manages all items

    @type items: dict
    @param items: dict of {id : item}
    """

    def __init__(self):
        self.items = {}
        self.prefixer = Prefixer()

    def _add(self, item):
        item_id = item.id
        item_content = item.content
        # add item to the dict
        self.items[item_id] = item
        tokens = re.split(SEPARATORS, item_content.lower())
        # add tokens to the prefixer
        for token in tokens:
            self.prefixer.insert(token, item_id)

    def _delete(self, item_id):
        item_content = self.items[item_id].content
        # delete the item from the dict
        del self.items[item_id]
        tokens = re.split(SEPARATORS, item_content.lower())
        # remove items from the prefixer for each token
        for token in tokens:
            self.prefixer.remove(token, item_id)

    def _set_items_query(self, query):
        items_ids = set()
        tokens = re.split(SEPARATORS, query.lower())
        cpt = True
        for token in tokens:
            if cpt:
                items_ids = set(self.prefixer.search(token))
            else:
                items_ids = items_ids.intersection(set(self.prefixer.search(token)))
            if items_ids == set():
                return items_ids
            cpt = False
        return items_ids

    def _query(self, nbr_results, query):
        # collect potential items' ids
        items_ids = self._set_items_query(query)
        # check if items_ids is not empty
        if items_ids == set():
            return ""
        # rank them according to the scoring method
        sorted_results = SortedItems(nbr_results)
        for item_id in items_ids:
            sorted_results.add(self.items[item_id])
        return sorted_results

    def _wquery(self, nbr_results, nbr_boosts, boosts, query):
        nbr_boosts = int(nbr_boosts)
        # collect potential items' ids
        items_ids = self._set_items_query(query)
        # check if items_ids is not empty
        if items_ids == set():
            return ""
        # check the boosts and create boosts_dict
        boosts_dict = {}
        if nbr_boosts > 0:
            boosts = boosts.split()
            for boost in boosts:
                type, score = boost.split(':')
                boosts_dict[type] = float(score)
        # rank them according to the scoring method
        sorted_results = SortedItems(nbr_results)
        for item_id in items_ids:
            item = copy.deepcopy(self.items[item_id])
            # chech the boost
            if nbr_boosts > 0:
                if item.id in boosts_dict.keys():
                    item.score *= boosts_dict[item.id]
                if item.type in boosts_dict.keys():
                    item.score *= boosts_dict[item.type]
            sorted_results.add(item)
        return sorted_results

    def process_command(self, in_command):
        """
        validate the current command and map it to the right function
        """
        any_command = COMMAND_MATCHER.match(in_command)
        #
        if any_command:
            command = any_command.group("command")
            parameters = any_command.group("parameters")
            if (command == "ADD"):
                add_command = ADD_COMMAND_MATCHER.match(parameters)
                self._add(Item(add_command.group("type"), add_command.group("id"),
                               add_command.group("score"), add_command.group("content")))
            elif (command == "DEL"):
                del_command = DEL_COMMAND_MATCHER.match(parameters)
                self._delete(del_command.group("id"))
            elif (command == "QUERY"):
                query_command = QUERY_COMMAND_MATCHER.match(parameters)
                results = self._query(query_command.group("nbr_results"),
                                      query_command.group("query"))
                print(results)
            elif (command == "WQUERY"):
                wquery_command = WQUERY_COMMAND_MATCHER.match(parameters)
                results = self._wquery(wquery_command.group("nbr_results"),
                                       wquery_command.group("nbr_boosts"),
                                       wquery_command.group("boosts"),
                                       wquery_command.group("query"))
                print(results)


class Item(object):
    """
    either a topic, a user, a board or a question

    @type type: str
    @param type: The item's type.

    @type id: str
    @param id: The item's id.

    @type score: float
    @param score: The item's score.

    @type content: str
    @param contetn: The item's content.

    @type time: time
    @param time: The item's time of creation.
    """

    def __init__(self, type, id, score, content):
        self.type = type
        self.id = id
        self.score = float(score)
        self.content = content
        self.time = datetime.datetime.now()

    def __repr__(self):
        return self.id

    def better_than(self, item):
        """
        compare the current item to the input item.
        follows this method:
            . highest score goes first.
            . same score; time FIFO.
        return true if the current item is better than the input, otherwise returns false
        """
        if (self.score > item.score):
            return True
        if (self.score < item.score):
            return False
        return True if (self.time > item.time) else False


class SortedItems(object):
    """
    Keeps a list of sorted elements depending on the scoring method.

    @type items: list
    @param items: the list sorted items  

    @type max_size: int
    @param max_sier: the max size of the list (-1 means unlimited number of items)
    """

    def __init__(self, max_size=-1):
        self.items = []
        self.max_size = int(max_size)

    def __repr__(self):
        return " ".join([item.id for item in self.items])

    def set_max_size(self, max_size):
        self.max_size = int(max_size)

    def add(self, item):
        """
        add new item to the list of items.
        if the list is full, add the item only if it has better score than at least one item
        in the list, and pop the item with the worst score.
        """
        items_l = len(self.items)
        pos = items_l
        for i in xrange(items_l):
            if (item.better_than(self.items[i])):
                pos = i
                break
        if (self.max_size < 0 or pos < self.max_size):
            temp = self.items[:pos]
            temp.append(item)
            temp += self.items[pos:]
            self.items = temp
            # now in the case of exceeding max_size
            if (self.max_size > 0 and (items_l + 1) > self.max_size):
                self.items.pop()


t = TypeHead()
N = int(raw_input())
while (N):
    t.process_command(raw_input())
    N -= 1
```
