---
layout: post
title: "Proof of Heap Algorithms Complexities"
modified:
categories:
excerpt: Mathematical proof of algorithms including heapify, heap construction and heap sort.
comments: true
tags: []
image:
  feature:
date: 2014-09-17T15:35:10-07:00
---

## Heapify(array $$A$$ with size $$N$$, index $$i$$ that contains node $$n$$):

Assuming subtrees of node $$n$$ are max heaps.

+ Idea: correcting one violation of heap invariant at $$n$$. Swap $$n$$ with one of its child and recursively call Heapify on that child.

It can be easily derived that

+ Number of levels below $$n$$: $$L = \lceil \lg{\frac{N}{i}} \rceil = \lceil \lg{N} - \lg{i} \rceil$$

Let $$T_{Heapify}(L)$$ be the time complexity. We have the simple recurrence relation:

$$
\begin{cases}
    T_{Heapify}(L) = T_{Heapify}(L - 1) + O(1) \\
    T_{Heapify}(0) = 0
\end{cases}
$$

Therefore, $$T_{Heapify}(N, i) \in O(\lg{N} - \lg{i})$$

## Build_max_heap(Array $$A$$ with size $$N$$):

+ Idea: the second half of a heap is consisted of leaves. So constructing a heap is calling Heapify from bottom to top on all of the nodes.

Without loss of generality, assume $$N$$ is a power of 2. Let $$T(N)$$ be the time complexity.

Pseudocode (note that here the heap starts from index 1):

{% highlight python %}
    for i in range(N/2, 1):
        Heapify(A, i)
{% endhighlight %}

$$
\begin{array} {lcl}
T_{Build\_heap}(N) & = & \sum_{i = 1}^{\frac{N}{2}} T_{Heapify}(N, i) \\
                   & = & \sum_{i = 1}^{\frac{N}{2}} O(\lg{N} - \lg{i}) \\
                   & = & O(\lg{N}) + \sum_{i = 2}^{\frac{N}{2}} O(\lg{N} - \lg{i}) \\
                   & \leqslant & O(\lg{N} + \int_{1}^{\frac{N}{2}} (\lg{N} - \lg{x}) dx) \\
                   & \leqslant & O(\lg{N} + (\frac{N}{2} - 1) \lg{N} - (\frac{N}{2} \lg{\frac{N}{2}} - \frac{N}{2}) + (\lg{1} - 1)) \\
                   & \in & O(N)
\end{array}
$$

## Heap_sort(Array $$A$$ with size $$N$$):

+ Idea: after popping out the first (max or min) element, move one of the leaves to $$A[1]$$ and call Heapify($$A$$, $$1$$). Continue to do so and get a sorted list.

Let $$T(N)$$ be the time complexity.

$$
\begin{array} {lcl}
T_{Heap\_sort}(N) & = & \sum_{i = 2}^{N - 1} T_{Heapify}(i, 1) \\
                  & = & \sum_{i = 2}^{N - 1} O(\lg{i}) \\
                  & \leqslant & O(\int_{2}^{N} \lg{x} dx) \\
                  & \leqslant & O(N \lg{N} - N - (2 \lg{2} - 2)) \\
                  & \in & O(N \lg{N})
\end{array}
$$