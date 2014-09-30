---
layout: post
title: "Best-Case Intuition of Alpha-Beta Pruning"
modified:
categories:
excerpt: Intuitive explanation of the square root branching factor
comments: true
tags:
  - Artificial Intelligence
  - Algorithms
image:
  feature:
date: 2014-09-17T23:46:08-07:00
---

By **best-case**, it means that, when choosing a node to expand, the algorithm always chooses in the optimal order. i.e.

+ order that returned scores rank from greatest to least for max nodes
+ order that returned scores rank from least to greatest for min nodes

Let the branching factor be $$b$$ and solution depth be $$m$$.

Before diving into explanation, we should first establish one crucial observation and review the pruning condition:

+ **Crucial observation:** if a node, among a group of nodes, is chosen first to be searched, it is the one that will be returned to the upper level.
+ **Pruning condition:** at any time, $$\alpha \geqslant \beta$$.

1. The algorithm will first expand the solution with optimal score, $$v$$, for current player. Without loss of generality, assume that current player is a max player.
    + **Note:** The algorithm doesn't know that the optimal solution has found.
    + All max nodes first chosen to expand should return scores less than $$v$$, otherwise $$v$$ wouldn't be the optimal (since all that min nodes actually do is returning these scores).
    + All min nodes first chosen to expand also have this property for the same reason.
        + Actually all min nodes meet this condition because otherwise its parent max node will return a value greater than $$v$$.
2. Then, divide into two cases:
    + for each max node, every child has to be searched (since no score greater than $$v$$ can be found and thus no pruning condition is met).
    + for each min node, after it get the first returned score from its child max node, it should immediately prune its other children because pruning condition is met.
3. Expanding $$b$$ nodes every two levels leads to a $$\sqrt{b}$$ branching factor.