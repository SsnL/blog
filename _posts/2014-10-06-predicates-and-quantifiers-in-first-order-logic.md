---
layout: post
title: "Predicates and Quantifiers in Logic"
modified: 2014-10-06 17:43:31 -0700
comments: true
excerpt: No quantifiers in propositional logic?
tags: [Artificial Intelligence, Logic]
image:
  feature: 
  credit: 
  creditlink: 
date: 2014-10-06 17:43:31 -0700
---

**First-order logic**, compared to propositional logic (or zeroth-order logic), has the ability to describe relations among objects. The main difference is usually believed to be that first-order logic uses predicates and quantifiers in expressions.

In this post, I will try to discuss what **order** really means and how quantifiers fit in this model.

Let us first have informal definitions of *predicate* and *function*.

+ ***Predicate***s are "functions" that evaluates to either True or False. For instance,   
  
  1. \$$HaveReadBook(p: Person, b: Book): bool$$
  2. \$$IsNumber(x: Number): bool$$
  3. \$$AliceIsSevenYearsOldAtAugSeventh2014: bool$$
  
+ ***Function***s are "functions" that evaluates to objects. For instance,

  1. \$$HatOnHead(p: Person): Hat$$
  2. \$$CatOfAlice: Cat$$

The meaning of **order** is all about argument types of predicates and quantifiers in one kind of logic.

+ In propositional logic, every object property is described by enumeration with atomic sentences. Each of these atomic sentences is essentially a predicate with **0** argument.

  $$P \land (\lnot Q \lor P)$$

+ In first-order logic, relations and properties are handled with predicate and function symbols. It might be useful to think that first-order logic is much more expressive to propositional logic in that it uses predicates to connect objects. Predicates and quantifiers in first-order logic can take arguments of object type.

  $$\exists x, \exists y, IsEqual(Descendant(x, y)), Son(x))$$

  , where $$Son$$ and $$Descendant$$ are functions and $$IsEqual$$ is a predicate.

+ Following this path, second-order logic should be describing predicates. Predicates and quantifiers in first-order logic should be able to operate on predicates of objects.

  $$\exists P, \forall x, P(x) \lor \lnot P(x)$$
  
  $$\forall P, IsCommutative(P) \implies \forall x, \forall y, \lnot P(x, y) \oplus P(y, x)$$

  , where $$P$$ is a predicate and $$x$$ is an object.

Now, the problem of where quantifiers come from is rather obvious.

It is usually said that quantifiers don't exist in zeroth-order logic. My understanding is that quite the opposite. There're quantifiers in every logic. However, since the argument of predicates is nothing in propositional logic, quantifiers are just useless. 