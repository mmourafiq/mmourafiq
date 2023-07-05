---
layout: blog
title: "The magic formula from facebook!"
subtitle: "The mathematical formula by Elo, which allows to rate the relative skill levels in two-player games."
cover_image: images/blog/facebook.png
cover_image_caption: "Facebook."
tags: [ math ]
---

In the Facebook movie [The Social Network](http://www.imdb.com/title/tt1285016/), Mark Zuckerberg
builds a website to compare the attractiveness of girls. Behind the scenes, the website uses an
algorithm, the one that Mark asked his friend Eduardo to give him.

In fact, it was actually a mathematical formula by Elo, which allows to rate the relative skill
levels in two-player games.

The equations driving the algorithm are shown briefly, written on young Zuck’s dorm room window:

`Ea = 1/(1 + 10 * (Ra-Rb) * 400) Eb = 1/(1 + 10 * (Rb-Ra) * 400)`

As explained in the movie, Facemash was quite simple. students went to a page and 2 random images of
girls were picked and presented to them. If a girl A wins over a girl B would gain more points than
winning over an ugly girl C.

Here’s the formula plotted
with [wolframalpha](http://www.wolframalpha.com/input/?i=+1%2F%281%2B10%5E%28%28x-y%29%2F400%29%29+)
