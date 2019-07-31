---
layout: post
title: "On the Countability of a Particular Equivalence"
description: "A minor proof of the uncountability of a certain equivalence defined on the reals"
tags: [math, measure theory]
---
{% include JB/setup %}

Let's define two real numbers to be equivalent if their difference is a
rational number. Denoting the equivalence by $$\sim$$, we get:

$$
x \sim y ~ \textrm{for} ~ x, y \in \mathbb{R} ~ \textrm{if and only if} ~ x - y \in \mathbb{Q}.
$$

Now consider the resulting _equivalence classes_, where each class is the set of
real numbers that are equivalent under $$\sim$$. So, for any real number $$x$$:

$$
[x] = \{ y \in \mathbb{R} ~ \mid ~ x \sim y \}
$$

For example, all the rational numbers trivially fall into
$$[0]$$.

_How **many** such equivalence classes are there?_
The answer to this question is a key (if small) step along the way to a proof about
the nonexistence of a particular type of "measure" on the real numbers, which I came
across recently while watching this
[intro video about Measure Theory](https://www.youtube.com/watch?v=llnNaRzuvd4).
It took me a while to work out the answer, which may seem obvious to others:

First, note that each equivalence class is _countable_.
In fact, each class is exactly as large as the set of
rational numbers, which is of course countable.
This becomes obvious if we rewrite the class like so:

$$
[x] = \{ x + r ~ \mid ~ r \in \mathbb{Q} \}.
$$


Next, notice that the _union_ of all these equivalence classes
needs to cover the real numbers. After all, every real number belongs to (exactly)
one of these classes.

It follows that there must be an **uncountable** number
of these classes, since otherwise, we would have the union of a countable number of
countable sets, which cannot possibly cover $$\mathbb{R}$$ (which is uncountable).

Neat!
