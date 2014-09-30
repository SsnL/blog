---
layout: post
title: "Local Search Advantages"
modified:
categories:
comments: true
excerpt: Comparing local search algorithms with path-constructing algorithms
tags:
  - Artificial Intelligence
  - Algorithms
image:
  feature:
  credit:
  creditlink:
date: 2014-09-22T14:45:16-07:00
---

In searching algorithms, local search is a class of algorithms that propose a solution and continue modifying it until it looks good enough. Although being not always optimal (mostly due to local optimum and randomness), these algorithms possess several advantages over other path-constructing algorithms, such as A*, breadth-first, IDA, etc.

By its nature of randomness, local search reduces complexity at the cost of possible suboptimal solutions. Local search is good for:

1. problems with memory constraints,
    + Local search algorithms keep only one current state in memory (or a fixed number of states, if using algorithms like local beam search and genetic algorithm)
    + However, path-constructing algorithms usually take exponential order of memory for large search trees.
2. approximations to computationally difficult problems, including NP-hard ones,
    + NP-hard problems cannot be easily solved. But approximation is possible in polynomial time.
3. problems with changes in state space, for instance, online search,
    + Online search refers to searching dynamically. In other words, the agent is searching in an evolving environment.
    + If, for instance, A* is adopted, by the time it gives out a solution for a particular state, environment has already changed to a state where the solution can not apply.
    + However, local search algorithms can continuously modify its solution according to the changing environment as they run.
    + One other advantage of local search is that once a search is complete, the algorithm can still be invoked at a later time when the environment has changed so as to optimize the previous solution in the new setting. Apparently, this is not possible for path-constructing algorithms.
+ Problems which we only care about results rather than process.
