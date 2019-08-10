---
layout: post
title: "Interval length via discretization"
description: "A minor proof for computing interval length"
tags: [math, measure theory]
---
{% include JB/setup %}

I found an interesting exercise in Terry Tao's Measure Theory book:
for any interval $$I = [a, b]$$ over $$\mathbb{R}$$ and $$N > 0$$,
prove that:

$$
|I| = \lim_{N \to \infty} \frac{|I \cap \frac{\mathbb{Z}}{N} |}{N}.
$$

where
$$\frac{\mathbb{Z}}{N} := \{ \frac{n}{N} \mid n \in \mathbb{Z} \}$$.

Let us first understand the intuition behind the formula above, by
plugging in some numbers.

For example, assume $$a = 3$$, $$b = 8$$, and $$N = 25$$.
Clearly, $$|I| = 5$$, which takes care of the left hand side of our equation.
What about the right hand side? $$I \cap \frac{\mathbb{Z}}{N}$$
is the set of points in $$\frac{\mathbb{Z}}{N}$$ which lie within
the interval $$I$$. We want to know the size of this set.
For our example, its easy to see that this set is basically:
$$
\{ {75}/{25}, {76}/{25}, \ldots, {200}/{25} \},
$$
which means $$ \vert I \cap \frac{\mathbb{Z}}{N} \vert = (200 - 75) + 1 = 126$$.
More generally, if
$$\frac{l}{N}$$ and $$\frac{h}{N}$$ are the smallest and largest elements
respectively in $$ I \cap \frac{\mathbb{Z}}{N} $$,
then:

$$
\vert I \cap \frac{\mathbb{Z}}{N} \vert = h - l + 1.
$$

If $$ I \cap \frac{\mathbb{Z}}{N} $$ can be expressed as
$$N(b - a) + \epsilon$$ where $$\epsilon$$ is some constant,
then the limit looks like it pretty much works out.
To do so, note that
$$\frac{l}{N} \ge a ~\textrm{and} ~ \frac{(l - 1)}{N} \lt a$$,
which can be rewritten as:

$$
Na \le l \lt (Na + 1).
$$

Similarly, we get:

$$
(Nb - 1) \lt h \le Nb.
$$

Plugging these inequalities into the expression for the interval size, we get:

$$
\vert I \cap \frac{\mathbb{Z}}{N} \vert = h - l + 1 \gt Nb - Na - 1.
$$

which we can rewrite as:

$$
\vert I \cap \frac{\mathbb{Z}}{N} \vert = N(b - a) + \epsilon
$$

where $$\epsilon > -1$$ is a constant.
Plugging this back into the right hand side of the original assertion, we get:

$$
\lim_{N \to \infty} \frac{|I \cap \frac{\mathbb{Z}}{N} |}{N}
  = \lim_{N \to \infty} \frac{N(b - a) + \epsilon}{N}
  = (b - a) +  \lim_{N \to \infty} \frac{\epsilon}{N}
  = (b - a).
$$
