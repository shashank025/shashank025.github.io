---
layout: post
title: "Interval length via discretization"
description: "A minor proof for computing interval length"
tags: [math, measure theory]
---
{% include JB/setup %}

I found an interesting exercise in [Terry Tao](https://www.math.ucla.edu/~tao/)'s
[Measure Theory book](https://terrytao.wordpress.com/books/an-introduction-to-measure-theory/):

> Proposition: for any $$N \in \mathbb{R}$$ and interval $$I = [a, b]$$ over
$$\mathbb{R}$$,
$$
|I| = \lim_{N \to \infty} \frac{|I ~ \cap ~ \frac{\mathbb{Z}}{N} |}{N},
$$
where
$$\frac{\mathbb{Z}}{N} := \{ \frac{n}{N} \mid n \in \mathbb{Z} \}$$.

The interesting thing about this proposition is that it expresses the length of
an interval $$|I| = (b - a)$$ (which is a _continuous_ measure) in terms of the
cardinality of a _discrete_ set, $$I \cap \frac{\mathbb{Z}}{N}$$.
To develop an intuition for how this proposition could work, first notice that
$$I \cap \frac{\mathbb{Z}}{N}$$ is the set of points in
$$\frac{\mathbb{Z}}{N}$$ which lie within the interval $$I$$.
For example, if $$a = 3$$, $$b = 8$$, and $$N = 25$$, then
$$
I \cap \frac{\mathbb{Z}}{N} = \{ {75}/{25}, {76}/{25}, \ldots, {200}/{25} \}
$$,
which means its cardinality is
$$(200 - 75) + 1 = 126$$.

Plugging this back into the right-hand side of the proposition, we see that:

$$
\frac{|I ~ \cap ~ \frac{\mathbb{Z}}{N} |}{N} = \frac{126}{25} = 5 + 0.04.
$$

As we can see, this is pretty close to $$|I| = 5$$.
This example makes it clear that
if the set $$ I \cap \frac{\mathbb{Z}}{N} $$ can be expressed as
$$N(b - a) + \epsilon$$ where $$\epsilon$$ is some constant,
then the proposition pretty much holds _at the limit_.
To get there, consider $$\frac{l}{N}$$ and $$\frac{h}{N}$$, the smallest and
largest elements of the set ($$l, h \in \mathbb{Z}$$).
Then, from our prior example, it is clear that the set cardinality can be
expressed as:

$$
\vert I \cap \frac{\mathbb{Z}}{N} \vert = h - l + 1.
$$

Now, note that $$\frac{l}{N} \ge a$$ and $$\frac{(l - 1)}{N} \lt a$$,
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
\begin{align*}
\lim_{N \to \infty} \frac{|I \cap \frac{\mathbb{Z}}{N} |}{N}
& = \lim_{N \to \infty} \frac{N(b - a) + \epsilon}{N} \\
& = (b - a) +  \lim_{N \to \infty} \frac{\epsilon}{N} \\
& = (b - a).
\end{align*}
$$
