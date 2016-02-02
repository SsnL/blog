---
layout: post
title: "An Interesting Math Problem"
modified: 2016-01-30 21:30:18 -0800
comments: true
excerpt:
tags: [Mathematics]
image:
  feature:
  credit:
  creditlink:
date: 2016-01-30 21:30:18 -0800
---

Hello! It's been quite a while since my last post. This post I'll present you a math problem, and a clever proof.

Here is the problem stated as follows:

> A set $$S_n$$ of points in $$\mathbb{R}^n$$ is called <span style='text-decoration:underline;'>special</span> if the distance between any pair of points takes one of two values. Formally, $$\lvert \{\lvert a - b \rvert : a, b \in S_n, a \neq b\} \rvert \leq 2$$.
>
> Prove that for any $$n$$ there exists a <span style='text-decoration:underline;'>special</span> set $$S_n$$ of size $$\frac{n(n+1)}{2}$$.

It is probably obvious that one can easily construct special sets of size $$\frac{n(n-1)}{2}$$ without much trouble:

> <span style='color:gray;'>
hover $$\rightarrow$$
</span>
<span
    style='color:white;'
    onmouseover="this.style.color='black';"
    onmouseout="this.style.color='white';"
    markdown="1">
Let $$S'_n$$ be the set of all points with $$1$$ on two of the $$n$$ dimensions and $$0$$ on all others (there are $${n \choose 2}$$ such points). Since the pairwise distance can only be $$1$$ or $$\sqrt{2}$$, $$S'_n$$ is special.
</span>

But how should we extend from $${n \choose 2}$$ to $${n + 1 \choose 2}$$?

Allow me to first introduce a related and useful concept:

> A <span style='text-decoration:underline;'>regular simplex</span> of $$\mathbb{R}^n$$ is a collection of points in $$\mathbb{R}^n$$ whose pairwise distance is constant.
>
> e.g. $$\{(0, 0), (1, 0), (\frac{1}{2}, \frac{\sqrt{3}}{2})\}$$ is a <span style='text-decoration:underline;'>regular simplex</span> in $$\mathbb{R}^n$$ since it forms an equilateral triangle.

As a fact, we know that regular simplexes of size $$n+1$$ exists in $$\mathbb{R}^n$$ for any $$n$$, shown by this easy-to-verify construction:

> <span style='color:gray;'>
hover $$\rightarrow$$
</span>
<span
    style='color:white;'
    onmouseover="this.style.color='black';"
    onmouseout="this.style.color='white';"
    markdown="1">
Consider a sequence of points on, well, $$\mathbb{R}^{\infty}$$, $$A_0, A_1, \dots$$ etc. Let $$A_0$$ be positioned at origin, and $$A_{n + 1} = \text{average of $A_{0:n}$} + \sqrt{\frac{n+2}{2n+2}} \text{ on $(n + 1)$-th dimension}$$.
</span>

Look at the numbers. We have $$n + 1$$ points. We want $${n + 1 \choose 2}$$ points. Do you see what we should try?

> <span style='color:gray;'>
hover $$\rightarrow$$
</span>
<span
    style='color:white;'
    onmouseover="this.style.color='black';"
    onmouseout="this.style.color='white';"
    markdown="1">
Yes! For a regular simplex $$R_n$$ in $$\mathbb{R}^n$$, let $$S_n = \{\text{midpoint of $i$ and $j$}: i, j \in R_n, i \neq j\}$$.
</span>

Why does this $$S_n$$ satisfy the requirements? Let's first take a step back and answer these two questions:

1. What are all possible shapes of regular simplexes of size $$3$$ in $$\mathbb{R}^2$$?
2. What about size $$4$$ regular simplexes in $$\mathbb{R}^3$$?

With a pencil and a piece of paper and some thinking, we discover that

> <span style='color:gray;'>
hover $$\rightarrow$$
</span>
<span
    style='color:white!important;'
    onmouseover="this.style.color='black';"
    onmouseout="this.style.color='white';"
    markdown="1">
Size $$3$$ regular simplexes in $$\mathbb{R}^2$$ must be an <a style='text-decoration:underline;color:inherit;' href="http://mathworld.wolfram.com/EquilateralTriangle.html">equilateral triangle</a>. Similarly, size $$4$$ regular simplexes in $$\mathbb{R}^3$$ must be a <a style='text-decoration:underline;color:inherit;' href="http://mathworld.wolfram.com/RegularTetrahedron.html">regular tetrahedron</a>.
</span>

Finally, here comes the final proof:

> <span style='color:gray;'>
hover $$\rightarrow$$
</span>
<span
    style='color:white!important;'
    onmouseover="this.style.color='black';"
    onmouseout="this.style.color='white';"
    markdown="1">
Let $$S_n$$ be constructed as above, through $$R_n$$. Let $$a$$ be $$R_n$$'s edge length. Consider any two edges $$i, j$$ in $$R_n$$. If they share a vertex, then $$i$$ and $$j$$ must be edges of an equilateral triangle, and thus their midpoints will be $$\frac{a}{2}$$ apart. Otherwise, $$i$$ and $$j$$ must form a regular tetrahedron, and it can be calculated that the midpoints be $$\frac{\sqrt{2}}{2}$$ apart.
</span>

