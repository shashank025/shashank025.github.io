---
layout: post
title: 'Understanding the big O: a note on abuse of notation'
date: 2013-07-16 10:55:35.000000000 -07:00
tags:
- algorithms
- complexity
- computer science
status: publish
type: post
published: true
meta:
  _edit_last: '4244661'
  geo_public: '0'
  _publicize_pending: '1'
author:
  login: shashankr
  email: shashank.ramaprasad@gmail.com
  display_name: shashankr
  first_name: ''
  last_name: ''
excerpt: !ruby/object:Hpricot::Doc
  options: {}
---

A friend of mine, who is currently enrolled in the
[Algorithms course](https://www.coursera.org/course/algo)
over at
[Coursera](https://www.coursera.org/)
reached out to me with the following problem:

> Assume two (positive) nondecreasing functions $$f$$ and $$g$$.
> If $$f(n) = O(g(n))$$, is it also true that
> $$ 2^{f(n)} = O(2^{g(n)})$$?

My friend felt unable to apply his understanding of
<a title="Wikipedia article on Big-O" href="https://en.wikipedia.org/wiki/Big_O_notation">big-O notation</a>
to this question.
In fact, he felt tripped up by the $$=$$ signs in the above expression.
I remembered being similarly at a loss a decade earlier doing my first Algorithms course at my alma mater,
<a title="BITS, Pilani" href="http://www.bits-pilani.ac.in/">BITS Pilani</a>,
and I now realize that my confusion was at least partly due to bad notation.
Computer Science-folks (as opposed to mathematicians),
when teaching complexity, do not emphasize
<em>at all</em>
that big-O actually refers to a
<strong>class</strong>, or a <strong>family</strong> of functions,
rather than a single one:

> $$O(f(n))$$ is the set of all functions whose absolute value (asymptotically)
> grows no faster than $$ \vert f(n) \vert $$,
> by up to a constant factor.

Armed with the above informal definition,
we (the mathematically trained) immediately see an abuse of notation
in the above problem with:

> $$ f(n) = O(g(n)) $$

and realize that the expression would be better written as

> $$ f(n) \in O(g(n)) $$

The <strong>belongs-to</strong> symbol ($$ \in $$) is more appropriate
than the <strong>equality</strong> symbol ($$ = $$) since the left hand side
is a single function, while the right hand side is a family of functions.
In other words, we are making <em>a statement about set membership</em>.
Unfortunately, this abuse of notation is widespread in Computer Science literature.
Newcomers, as always, bear the brunt.

<p>Before answering the original problem my friend posed, consider the more general problem:</p>

> Given two functions $$ f $$ and $$ g $$,
> if $$ f(n) \in O(g(n)) $$, is it also true that
> $$ 2^{f(n)} \in O(2^{g(n)}) $$?

The answer is <strong>no</strong>, since $$ f(n) = -n $$ and $$ g(n) = -n^2$$ is a counterexample.
Since $$ |-n| = |n| $$ grows no faster than $$ |-n^2| = n^2 $$, we get that $$ f(n) \in O(g(n)) $$.
But $$ |2^{-n}| = (0.5)^n $$ _does_ grow faster than $$ | 2^{-n^2} | = (0.5)^{n^2} $$,
and so $$ 2^{f(n)} \notin O(2^{g(n)}) $$.

The nature of the counterexample however indicates to us that
if we imposed the restriction that the functions were positive and non-decreasing,
things might be different.
In fact, if $$ x > 0 $$, $$ y > 0 $$, and
$$ M > 0 $$, then $$\vert x \vert < M |y|$$

$$
\begin{align*}
 &\implies \qquad {}  x     < My         & \qquad {} (|a| = a \text{ when } a > 0) \\
 &\implies \qquad {}  2^x   < 2^{My}     & \\
 &\implies \qquad {}  2^x   < k 2^{y}    & \qquad {} \text{ where } k = 2^{(M - 1)y} > 0 \\
 &\implies \qquad {}  |2^x| < k |2^{y}|. &
\end{align*}
$$

In other words, <strong>yes</strong>,
if $$ f $$ and $$ g $$ are positive, and
$$ f(n) \in O(g(n)) $$, then
$$ 2^{f(n)} \in O(2^{g(n)}) $$.
