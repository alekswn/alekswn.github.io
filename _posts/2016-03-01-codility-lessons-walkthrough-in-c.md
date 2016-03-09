---
layout: post
title: Codility Lessons Walkthrough in C++
---

Here are my C++ solutions for [Codility's training tasks](https://codility.com/programmers/lessons/) here. All these solutions are scored 100% score.

* TOC
{:toc}

# _Lesson 99_: Future training

## PolygonConcavityIndex

_Check whether a given polygon in a 2D plane is convex; if not, return the index of a vertex that doesn't belong to the
convex hull._

Expected worst-case time and space complexity is _linear_ in the number of points.


[FULL TASK DESCRIPTION](https://codility.com/programmers/task/polygon_concavity_index/)

### Solution approach

**The property of a convex polygon**

Polygon is convex if and only if every internal angle is less than or equal to 180 degrees. 

**Key Lemma**

_Polygon is convex if and only if every edge from any sequence of adjusted edges in that polygon makes rotation in the same direction (clockwise or counter-clockwise)._

The formal proof follows from the property above. 

Using the lemma above it's not hard to construct a linear-time algorithm to determine if polygon is convex.

**High-Level algorithm**

1) Select some edge.

2) Determine the direction of rotation of successive edge.

3) Ensure every successive edge makes rotation to the same direction.

This algorithm is correct but it does not gives us a point which doesn't belong to convex hull. _How do we find such point ?_ Obviously such points must belong to edges which make wrong rotation.
If we'd select an edge from convex hull at step 1 the trouble-making point is the starting point of the first edge which
makes wrong rotation.

_How do we select an edge from a convex hull before any computation ?_
We'll take an extreme, for example, the
very bottom point. This point and two adjusted points form two edges of convex hull for sure. If there are a number of
points with the same extreme small **y** coordinate they all lie on the same line which belongs to convex hull and the
corner points of that line are starting point of two edges which belong to the convex hull as well.

_And how do we determine the direction of rotation ?_
Given edges **AB** and **BC** we can define the direction of rotation as the sign of vector product.

### Implementation

<script src="https://gist.github.com/alekswn/41daba265510643d429f.js"></script>

See my screencast for details on implementation.

<iframe width="420" height="315" src="https://www.youtube.com/embed/PU9r1-MDTGI" frameborder="0"
allowfullscreen></iframe>

### [Codility's test results](https://codility.com/demo/results/trainingVXSGSW-9NM/)
