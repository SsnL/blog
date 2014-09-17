---
layout: post
title: General Birthday Problem
modified: null
categories: null
excerpt: Discuss the classic birthday problem generally.
tags: 
  - CS 70
image: 
  feature: null
date: 2014-09-17T10:32:56-07:00
published: true
---

## Problem:

Assume that everyone's birthday is uniformly distributed in 365 days of a year. What is the probability that no two people have birthdays with difference less than or equal to $$k$$ days among $$n$$ people? 

## Solution:

If $$n(k + 1) > 365$$, the probability is clearly 0. 

$$
\mathbb{P} = 0
$$
        
If $$k = 0$$, the problem degenerates into the classic birthday problem. The solution is as follows. 

$$
\mathbb{P} = \frac{365!}{365^n \cdot (365 - k + 1)!}
$$

Otherwise, we number the 365 days as $$d_i$$, $$i \in \{1, \cdots, 365\}$$ and divide the problem into two cases:

+ No one is born during $$\{d_{365-k+1}, \cdots, d_{365}\}$$.

    Let the $n$ people be $$P_i$$, $$i \in \{1, \cdots, n\}$$. Then we combine each $$P_i$$'s  birthday with the $$k$$ days right after it. Then we have $$n$$ combined elements and $$365 - n(k + 1)$$ days. The problem transforms into picking $n$ days from a row of $$365 - nk$$ days and replacing them with our $$n$$ combined elements.

    We have $$n! {365 - nk \choose n} = (365 - nk)_n$$ possible cases each with equal probability.

+ Someone is born during $$\{d_{365 - k + 1}, \cdots, d_{365}\}$$. 

    Apparently, no two people can both have birthdays in that range. Therefore, the problem becomes how to put $n - 1$ people's birthdays in the rest $$365 - k - 1$$ days. 

    Notice that this situation is different with the situation in the problem where the 365 days form a cycle. So, using the method in the first case, We have $$(n - 1)! {365 - k - 1 - (n - 1)k \choose n - 1} = (365 - nk - 1)_{n - 1}$$ possible cases each with equal probability.

    However, we have $$365 - (365 - k + 1) + 1 = k$$ days and $$n$$ people to choose from. Therefore, there are $$nk \cdot (365 - nk - 1)_{n - 1}$$ cases in total.

Each case is of probability $$\frac{1}{365^n}$$. So the probability that no two people have birthdays with difference less than or equal to $$k$$ days among $$n$$ people is 

$$
\mathbb{P} = \frac{(365 - nk)_n + nk \cdot (365 - nk - 1)_{n - 1}}{365^n}
$$

By inspecting the formula, we find out that it also works for the first two cases. Therefore, this formula give correct answer for all cases.


## R test code:

    experiment = function (n, k) {
        days = sample(365, n, replace = T)
        if (k == 0) {
            return(!any(duplicated(days)))
        }
        dict = rep(-1, 365 %/% k + 1)
        if (365 %% k != 0) {
            offset = k - 365 %% k
        } else {
            offset = 0
        }
        for (day in days) {
            div = day %/% k + 1
            rem = day %% k
            if ((dict[div] >= 0) ||
                (((div + 1) < 365 %/% k + 2) && (dict[div + 1] > -1) && (dict[div + 1] <= rem)) ||
                ((div == 365 %/% k + 1) && (dict[1] > -1) && (dict[1] - offset <= rem)) ||
                ((div > 1) && (dict[div - 1] >= rem)) ||
                ((div == 1) && (dict[365 %/% k + 1] > -1) && (dict[365 %/% k + 1] + offset >= rem))) {
                return(F)
            }
            dict[div] = rem
        }
        return(T)
    }

    run = function(times, n, k) {
        result = sum(replicate(times, experiment(n, k))) / times
        prob = (choose(365 - n * k, n) + k * choose(365 - n * k - 1, n - 1)) * factorial(n)/(365^n)
        print(sprintf("experiment: %f, probability: %f", result, prob))
    }    

## Sample output: 

    > run(100000, 15, 3)
    [1] "experiment: 0.112590, probability: 0.113578"
    > run(100000, 15, 3)
    [1] "experiment: 0.113670, probability: 0.113578"
    > run(100000, 15, 32)
    [1] "experiment: 0.000000, probability: 0.000000"
    > run(100000, 1, 32)
    [1] "experiment: 1.000000, probability: 1.000000"
    > run(100000, 15, 0)
    [1] "experiment: 0.746700, probability: 0.747099"
    > run(100000, 15, 1)
    [1] "experiment: 0.406600, probability: 0.409946"
    > run(100000, 15, 1)
    [1] "experiment: 0.409680, probability: 0.409946"