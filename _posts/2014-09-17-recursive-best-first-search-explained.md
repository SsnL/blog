---
layout: post
title: "Recursive Best First Search Explained"
modified:
categories:
excerpt: Inspect the pseudo-code of recursive best first search
tags: [CS 188]
image:
  feature:
date: 2014-09-17T12:18:15-07:00
---

The idea of recursive best first search is to simulate A* search with $$O(bd)$$ memory, where $$b$$ is the branching factor and $$d$$ is the solution depth. `g` is cost so far and `h` is the heuristic. The algorithm is reproduced below:

    function RECURSIVE-BEST-FIRST-SEARCH(problem) returns a solution, or failure
        return RBFS(problem, MAKE-NODE(problem.INITIAL-STATE), INFINITY)

    function RBFS(problem, node, f_limit) returns a solution, or failure and a new f-cost limit
        if problem.GOAL-TEST(node.STATE) then
            return SOLUTION(node)
        successors <- [ ]
        for each action in problem.ACTIONS(node.STATE) do
            add CHILD-NODE(problem,node,action) into successors
        if successors is empty then
            return failure, INFINITY
        /* update f with value from previous search, if any */
        for each s in successors do
            s.f <- max(s.g + s.h, node.f))
        loop do
            best <- the lowest f-value node in successors
            if best.f > f_limit then
                return failure, best.f
            alternative <- the second-lowest f-value among successors
            result, best.f <- RBFS(problem, best, min(f_limit, alternative))
            if result != failure then
                return result

Think of `node.f` as the best `bot_node.g + bot_node.h` where `bot_node` is a node from the bottom layer in the subtree from `node`.

The part that perplexed me the most is,

    /* update f with value from previous search, if any */
    for each s in successors do
        s.f <- max(s.g + s.h, node.f)

If we visit `node` in `RBFS(p_node)`, it is probable that `RBFS(node)` returns `failure, new_f`. Then the `f`s of `p_node.successors` changed order, and in the next iteration, i.e., in the next recursive call of `RBFS`, we might have a different `min(f_limit, alternative)`. Thus, continuing to do so, it could visit `node` twice in `RBFS(p_node)`.

If it has already expanded `node` and got `failure, node.f = new_f`, we know that from `node`, it will reach a "barrier", a frontier of at least `f = node.f`. Then each children, `child_node` of `node`, will meet a frontier of at least `f = node.f` as well. If `child_node` has `child_node.g + child_node.h` less than `new_f`, it is underestimating. The algorithm can safely replace its `f` with `node.f`.

With consistent heuristic, the instruction is perfectly fine sine each path has non-decreasing f-values and thus `child_node.f` won't be replaced by `node.f` in first `RBFS(node)`. Problem is, with inconsistent heuristic, sometimes when `node` is expanded at the first time, `node.f = f.g + f.h` isn't a value from frontier, moreover, it didn't even reach its successors. Then the `max` action makes `child_node` get a higher `f`. Therefore, the algorithm doesn't necessarily behave like naive A* search and expand the node with least `g + h`.
