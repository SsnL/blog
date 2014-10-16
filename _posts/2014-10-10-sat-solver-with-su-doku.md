---
layout: post
title: "SAT Solver with Su Doku!"
modified: 2014-10-10 10:40:26 -0700
comments: true
excerpt: How computer beats me on Su Doku
tags: [Artificial Intelligence, Su Doku]
image:
  feature:
  credit:
  creditlink:
date: 2014-10-10 10:40:26 -0700
---
Once when I broke my wrist before coming to Berkeley, I had to lay on bed for about two weeks. Bored of sleeping and endless TV shows, I got addicted to Su Doku. I learnt a whole lot about interesting terminologies and abbreviation, such as fishes, ALC, ALS, etc.

(here is [the Wikipedia page](http://en.wikipedia.org/wiki/Sudoku) if you are not familiar with the game)

At the time, as someone who had a few (not extremely successful) experience with AI, I naturally thought about using one of few algorithms I knew to solve Su Doku, DFS. I soon realized that naive DFS without optimization takes ages to solve one problem.

One and a half years later, I am now taking an artificial intelligence course here at UC Berkeley. I now know about CSP searching techniques, such as variable ordering, AC propagation, forward chaining, as well as propositional logic and SAT solvers. All these didn't remind me much of the Su Doku problem until one day I accidentally read Peter Norvig's [article](http://norvig.com/sudoku.html) on a CSP approach to solve Su Doku.

An idea just stroke me: why not try solving Su Doku as a SAT problem?

With a few files from course project, I was able to make a Su Doku solver work with a few rules expressed in logic sentences (which are really nice since all of them are in CNF without any extra manipulation).

Symbols are fairly straightforward. I have symbol $$C[i][j][k]$$ representing number $$k$$ is chosen in the block at row $$i$$ and col $$j$$. That is the best I can do with propositional logic.

In order to have the most concise expressions of the rule (at least IMO), I choose the following sentences:

  + Each block must have exactly one of the symbols set to true.
  + In each 3-by-3 square, all numbers (1 to 9) must be present.
      + For example, in the top left 3-by-3 square, to ensure number 2 is present, I have

          $$C[0][0][2] \lor C[0][1][2] \lor \cdots C[3][2][2] \lor C[3][3][2]$$

        There is no need to ask for there to be only one number 2 since we ask **each** of the nine numbers to appear in the square.
  + In each row or column, all numbers must be present.

Here comes the actual code that builds rule expressions.
*(Actually $$C[i][j][k]$$ considers number $$k + 1$$ because we count from 0 in computer science.)*

{% highlight python %}
def rules():
    s = set()
    # Exclusion in each block
    for i in xrange(9):
        for j in xrange(9):
            s.add(exactlyOne(symbols[i][j]))
    # All numbers must present in each 3-by-3 square
    for li in xrange(3):
        for lj in xrange(3):
            for n in xrange(9):
                e = None
                for si in xrange(3):
                    for sj in xrange(3):
                        e = tryOr(e, symbols[li * 3 + si][lj * 3 + sj][n])
                s.add(e)
    # All numbers must present in each row / col
    for i in xrange(9):
        for n in xrange(9):
            row = None
            col = None
            for j in xrange(9):
                row = tryOr(row, symbols[i][j][n])
                col = tryOr(col, symbols[j][i][n])
            s.add(row)
            s.add(col)
    return s
{% endhighlight %}

Finally, let us see how my Su Doku solver behaves on a "hard" problem (one that got me stuck for hours):

    >>> solve(".163.....3.95.....4.7..69...8.2.......3.9.7.......8.3...16..2.4.....23.6.....157.")
    Problem:
    . 1 6 | 3 . . | . . .
    3 . 9 | 5 . . | . . .
    4 . 7 | . . 6 | 9 . .
    ------+-------+------
    . 8 . | 2 . . | . . .
    . . 3 | . 9 . | 7 . .
    . . . | . . 8 | . 3 .
    ------+-------+------
    . . 1 | 6 . . | 2 . 4
    . . . | . . 2 | 3 . 6
    . . . | . . 1 | 5 7 .

    Solution:
    8 1 6 | 3 2 9 | 4 5 7
    3 2 9 | 5 7 4 | 8 6 1
    4 5 7 | 8 1 6 | 9 2 3
    ------+-------+------
    7 8 5 | 2 6 3 | 1 4 9
    6 4 3 | 1 9 5 | 7 8 2
    1 9 2 | 7 4 8 | 6 3 5
    ------+-------+------
    5 3 1 | 6 8 7 | 2 9 4
    9 7 8 | 4 5 2 | 3 1 6
    2 6 4 | 9 3 1 | 5 7 8

    Expression Build + Sanity Check: 0.36s
    Solving: 0.30s
    ---------------
    Total: 0.66s

Clearly, it is at least $$\frac{2\text{h} \cdot 3600\text{s/h}}{0.66\text{s}} = 10909$$ times faster than me, an intellectual human being. The solver can be made even faster since

  + a lot of sanity checking is done along the way, and
  + a wrapper of SAT solver and logic sentences does exist.

I am quite amazed how the SAT solver ([PycoSAT](http://fmv.jku.at/picosat/) here) deals with Su Doku in such an efficient fashion. Artificial intelligence is one of the most intriguing areas of computer science to me.

Next time, I will try to solve another AI problem that I attempted before, designing an AI agent for the card game, [*Ascension*](http://ascensiongame.com/).

Here is the Su Doku SAT solver source code on [GitHub](https://github.com/SsnL/SAT-Su-DoKu).