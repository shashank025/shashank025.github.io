---
layout: post
title: "Bloom filters for set intersections?"
description: ""
category:
tags: [bloom filter,probability,math,computer science]
---
{% include JB/setup %}

A Bloom filter is a probabilistic data structure $$B$$ that
approximately and efficiently answers the _set membership_
question. Given a Bloom filter B constructed out of the elements
of a set $$S$$, $$B(x)$$ returns $$YES$$ or $$NO$$,
indicating the membership of $$x$$ in $$S$$.
Bloom filters are _approximate_ since they can return
false positives. The false positive rate $$\epsilon$$ for a Bloom filter
can be expressed by:

$$
Pr(B(x) = \text{YES} \mid x \notin S) = \epsilon.
$$

An interesting question is:
_Can Bloom filters be used to compute whether two sets are **disjoint**?_

That is, given a set $$S$$ represented by a Bloom filter $$B$$,
and another set $$Q$$, can we use the Bloom filter to accurately
determine whether $$Q \cap S = \varnothing$$?
An obvious procedure to compute disjointness is:

> _Procedure $$P$$:_
Iterate through each element $$q$$ of the set $$Q$$.
If $$B(q) = YES$$ for any element $$q$$,
then return $$NO$$, else return $$YES$$.

Before we can reason about the accuracy of this procedure, let's define
the following events:

* $$D$$: the event that $$S$$ and $$Q$$ are disjoint
(and $$D'$$ to be its complement), and
* $$A$$: the event that $$P$$ is accurate (and $$A'$$ to be its complement).

Now, we make the following claims about our procedure $$P$$:

> _Claim 1_: $$ Pr(A' \mid D') = 0 $$.

In other words, our procedure $$P$$ is guaranteed to be correct when
$$S$$ and $$Q$$ are _not_ disjoint.
This claim follows easily from the fact that a Bloom filter cannot have
false negatives, _i.e._,  if $$q$$ does exist in $$S$$, then $$B(q)$$ will
return $$YES$$.

> _Claim 2_: $$ Pr(A' \mid D) \ge 0 $$.


In other words, $$P$$ _can_ be inaccurate when
$$S$$ and $$Q$$ are disjoint.
Specifically, this procedure can be inaccurate if
the Bloom filter returns $$YES$$ for at least one element in $$Q$$,
given that no element in $$Q$$ belongs to $$S$$.
But _how_ inaccurate?
It is often easier to compute the probability of the complementary event
$$ A \mid D $$,
which is the event that the Bloom filter returns $$NO$$ for every element of $$Q$$.
The probability that the Bloom filter returns $$NO$$ for a given element
$$q \in Q$$ can be expressed by:

$$
Pr(B(q) = \text{NO}~\mid~q \notin S) = 1 - \epsilon.
$$

Assuming independence of Bloom filter output for the different elements of
$$Q$$, and assuming that $$Q$$ has $$n$$ elements, we can write:

$$
Pr(A \mid D) = (1 - \epsilon)^n.
$$

Further, notice that the likelihood of the sets being disjoint has nothing
whatsoever to do with the Bloom filter accuracy (I like to picture it as:
$$Pr(D \mid A) = Pr(D)$$). In other words, $$D$$ and $$A$$ are independent!
If we wanted to be explicit, we could apply Bayes' Theorem:

$$
Pr(A \mid D) = \frac {Pr(D \mid A) \cdot Pr(A)}{ Pr(D) } = Pr(A),
$$

from which it follows that:

$$
Pr(A) = (1 - \epsilon)^n.
$$

and:

$$
Pr(A') = 1 - (1 - \epsilon)^n.
$$

Since $$0 \le \epsilon \le 1$$, the inaccuracy grows with $$n$$.
This makes intuitive sense since the larger the set $$Q$$,
the more the chances that we will run into a Bloom filter false positive.
In plain English: Bloom filters are probably **not** a good idea for computing
whether two sets are disjoint.
