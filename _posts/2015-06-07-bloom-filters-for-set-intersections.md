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
question. So, for a set $$S$$, and for an element $$x$$ from
the same domain as $$S$$, $$B(x)$$ returns $$YES$$ or $$NO$$,
indicating the membership of $$x$$ in $$S$$.
Bloom filters are _approximate_ since they can return
false positives. The false positive rate $$\epsilon$$ for a Bloom filter
can be expressed by:

$$
Pr(B(x) = \text{YES} \mid x \notin S) = \epsilon.
$$

An interesting question is:
_Can Bloom filters be used to compute **set intersections** with decent accuracy?_
One could apply the following procedure to find
the intersection between a set $$S$$ (represented by a bloom filter $$B$$)
and some other set $$Q$$:

> _Procedure $$P$$:_
Iterate through each element $$q$$ of the set $$Q$$.
If $$B(q) = YES$$ for any element $$q$$,
then return $$YES$$, else return $$NO$$.

In order to reason about the accuracy of this procedure, let's define $$D$$
as the event that $$S$$ and $$Q$$ are disjoint, and $$D'$$ its complement.
Now, we make the following claims about $$P$$:

> _Claim 1_: $$ Pr(P ~ \textrm{is inaccurate} \mid D') = 0 $$.

In other words, our procedure $$P$$ is guaranteed to be correct when
$$S$$ and $$Q$$ are _not_ disjoint.
This claim follows easily from the fact that a Bloom filter cannot have
false negatives, _i.e._,  if $$q$$ does exist in $$S$$, then $$B(q)$$ will
return $$YES$$.

> _Claim 2_: $$ Pr(P ~ \textrm{is inaccurate} \mid D) \ge 0 $$.


In other words, $$P$$ _can_ be inaccurate when
$$S$$ and $$Q$$ _are_ disjoint.
Specifically, this procedure can be inaccurate if
the Bloom filter returns $$YES$$ for at least one element in $$Q$$,
even when no element in $$Q$$ belongs to $$S$$.
How likely is this event?
It is often easier to compute the probability of the complementary event.
So, we can compute $$ Pr(P ~ \textrm{is accurate} \mid D) $$.
This is the event that the Bloom filter is accurate, _i.e._,
it returns $$NO$$ for every element in $$Q$$,
given that no element in $$Q$$ belongs to $$S$$.
It follows directly from the definition of the error rate that:

$$
Pr(B(q) = \text{NO}~\mid~q \notin S) = 1 - \epsilon.
$$

Assuming independence of Bloom filter output for the different elements of
$$Q$$, and assuming that $$Q$$ has $$n$$ elements, we can write:

$$
Pr(P ~ \textrm{is accurate} \mid D) = (1 - \epsilon)^n.
$$

In other words,

$$
Pr(P ~ \textrm{is inaccurate} \mid D) = 1 - (1 - \epsilon)^n.
$$

Since $$1 - \epsilon$$ is a fraction less than $$1$$, the probability of
inaccuracy is significant.
This derivation assumes that the Bloom filter output
for the elements of $$Q$$ were independent events. But this might not be true,
since the order of operations can affect the likelihood of false positives.
I am not sure how to compute how non-independence affects the above calculation,
but I can’t imagine it would materially alter it all that much in many scenarios.
