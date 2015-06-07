---
layout: post
title: "Bloom filters for set intersections?"
description: ""
category: 
tags: []
---
{% include JB/setup %}

For any set $$S$$, one may construct a Bloom filter,
which is a probabilistic data structure $$B$$ that
approximately and efficiently answers the _set membership_
question,
such that $$B(x)$$ returns $$YES$$ or $$NO$$,
indicating the membership of $$x$$ in $$S$$.
Bloom filters are _approximate_ since they can return
false positives.
The error rate $$\epsilon$$ for a Bloom filter is:

$$
Pr(\text{YES} | x \notin S) = \epsilon .
$$

To find an overlap
(_i.e._, a non-empty intersection) between a set $$S$$ (represented by a bloom filter $$B$$)
and some other set $$Q$$, I can apply the following procedure:

_Procedure 1:_ If $$B(q) = YES$$ for any element $$q$$ from the set $$Q$$, then return $$YES$$, else return $$NO$$.

How accurate is the above procedure? Since Bloom filters cannot have false negatives,
the procedure is inaccurate if and only if there is a false positive,
_i.e._, the Bloom filter returns $$YES$$ for at least one element in $$Q$$,
even though no element in $$Q$$ belongs to $$S$$.
How likely is this event?
It is often easier to compute the probability of the complementary event.
In our case, this is the event that the bloom filter is accurate, _i.e._,
it returns $$NO$$ for every element in $$Q$$,
given that no element in $$Q$$ belongs to $$S$$.
It follows directly from the definition of the error rate that:

$$
Pr(\text{NO} | q \notin S) = 1 - \epsilon .
$$

Assuming independence of Bloom filter output for the different elements of $$Q$$,
and assuming that $$Q$$ has $$n$$ elements,

$$
Pr(\text{intersection is accurate}) = (1 - \epsilon)^n .
$$

In other words,

$$
Pr(\text{intersection is inaccurate}) = 1 - (1 - \epsilon)^n .
$$

Since $$1 - e = f$$ is a fraction, $$f^n$$ is pretty small,
which means that the probability of inaccuracy is significant.
Note that this derivation assumed that the Bloom filter output
for the elements of $$Q$$ were independent events. But this might not be true,
since the order of operations can affect the likelihood of false positives.
I am not sure how to compute how non-independence affects the above calculation,
but I can’t imagine it would materially alter it all that much in many scenarios.
