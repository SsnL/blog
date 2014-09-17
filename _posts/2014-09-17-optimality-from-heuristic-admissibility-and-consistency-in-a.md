---
layout: post
title: "Optimality From Heuristic Admissibility and Consistency in A*"
modified:
categories: 
excerpt: Admissible and consistent heuristics and optimality of the A* algorithm.
tags: []
image:
  feature:
date: 2014-09-17T11:33:29-07:00
---

Consider the function $$f$$ used as priority key in A* algorithm.

$$
f() = w() + h()
$$

# Admissible $$h$$

+ Guarantees optimality if before adding a node $$n$$ to fringe,
    + Check if $$n$$ is traversed.
        + If so, check if current path is better than the one traversed
            + If so, add to fringe
    + Check if $$n$$ is in fringe.
        + If so, check if current path is better than the one in fringe
            + If so, add to fringe
    + Add $$n$$ to fringe
+ The above algorithm resembles tree search.    
        
# Consistent $$h$$

+ Implies non-decreasing $$f$$ on every path.
+ Guarantees optimality by ensuring, whenever a node $$n$$ is popped out of the fringe, it is from the shortest path to it.
+ Thus, new node can be ignored if it was traversed.
    + Cannot ignore if it is in the fringe and to be traversed.
    + In the above case, replace the one in the fringe if current path is better. i.e. lower $$f$$.
+ The above algorithm is graph search.  