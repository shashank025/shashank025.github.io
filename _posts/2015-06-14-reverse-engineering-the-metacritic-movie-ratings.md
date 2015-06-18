---
layout: post
title: "Reverse engineering Metacritic"
description: ""
category: 
tags: []
---
{% include JB/setup %}

_Note: All related code is available on [my github](https://github.com/shashank025/metacritic-weights)._

[metacritic](http://www.metacritic.com/) is a popular site that computes an
aggregate _metascore_ for each movie as a weighted average of individual critic scores.
Reviewers from major periodicals may have a greater effect on a movie review
on average than niche ones.
Tantalizingly, the [metacritic FAQ Page](http://www.metacritic.com/faq) says:

> Q: Can you tell me how each of the different critics are weighted in your formula?
>
> A: Absolutely not. 

That sounds like a challenge to me.
It _should_ be possible to infer the relative weights for movie critics
using standard machine learning techniques.
In fact, if we do this right, we should be able to correctly predict the metascore
for any new movie (given the individual critic ratings).

This post describes my attempt to build such a system.

### The Model

We introduce some notations and assumptions about the problem:

* metacritic uses ratings from $$n$$ critics.
* Our data set has $$m$$ movies.
* $$r_{ij}$$ is the rating of movie $$i$$ by critic $$j$$.
This forms an $$m \times n$$ matrix.
   * Not all critics rate all movies!
     In other words, $$r_{ij}$$ may not be defined for all $$i$$ and $$j$$.
   * Where defined, the values are constrained: $$0 \leq r_{ij} \leq 100$$.
* $$\theta_j$$ is the _relative weight_ (or importance) of critic $$j$$
(this is what we are trying to _learn_).
   * There is no point in having a critic weight of $$0$$
     (why even consider a critic whose rating isn't worth anything).
   * In light of the previous point, we constrain
     critic weights to be _positive_,
     _i.e._, $$\theta_j > 0$$ for all $$j$$,
   * Since these weights are relative, they must add up to one,
     _i.e._, $$\sum_{j=1}^n \theta_j = 1$$.
   * Critic weights stay constant across movies (but may get updated over time).
   * The $$n$$-dimensional vector
     $$\theta = (\theta_1, \theta_2, \ldots, \theta_n)$$
     represents _a solution_, a possible assignment of weights to critics.
   * Due to the above constraints, the solution space of $$\theta$$ forms
     a _bounded, [affine hyperplane](https://en.wikipedia.org/wiki/Hyperplane#Affine_hyperplanes)_.
* $$p_i$$ is the _published_ metascore for movie $$i$$.
   * These values are also constrained: $$0 \leq p_i \leq 100$$ for all $$i$$.
* $$y_i(\theta)$$ is the _predicted_ metascore for movie $$i$$
for a given choice of relative weights.
   * We will drop the $$\theta$$ when it is obvious.
* $$p = (p_1, p_2, \ldots, p_m)$$ and $$y(\theta) = (y_1(\theta), y_2(\theta), \ldots, y_m(\theta))$$
are vectorized forms we will use for conciseness later.

An obvious definition of $$y_i(\theta)$$
is simply a weighted sum:

$$
y_i(\theta) = \sum_{j=1}^n \theta_j r_{ij}.
$$

But remember:
 _Not all critics rate all movies_.

In other words, the summation above may not be valid,
since not all $$r_{ij}$$ values are necessarily _defined_.
How do we deal with this _incomplete_ matrix $$r_{ij}$$?
My best guess is that metacritic _normalizes_ its scores
using the available ratings.
For example, if the movie with $$i = 4$$
was only rated by two critics ($$j = 2$$ and $$j = 7$$),
metacritic computes a metascore like so:

$$
    y_4 = \frac{\theta_2 r_{42} + \theta_7 r_{47}}{\theta_2 + \theta_7}.
$$

In fact, the metacritic FAQ page says they wait until
a movie has at least 4 reviews before computing a metascore.
So they want at least 4 defined $$r_{ij}$$ values
for a given $$i$$.
Lets define the following additional variables:

* $$r'_{ij} = r_{ij}$$ if movie $$i$$ is rated by critic $$j$$
and $$0$$ otherwise.
* $$e_{ij} = 1$$ if movie $$i$$ is rated by critic $$j$$, and $$0$$ otherwise.

Note that $$r'_{ij}$$ and $$e_{ij}$$ are both $$m \times n$$ matrices,
but unlike $$r_{ij}$$, they are fully defined.

Using these, we modify the definition for $$y_i$$:

$$
y_i(\theta) = \frac{ \sum_{j=1}^n \theta_j r'_{ij} }{ \sum_{j=1}^n \theta_j e_{ij} }.
$$

With these definitions in place,
we will try to find a $$\theta$$
satisfying:

$$
\DeclareMathOperator*{\argmin}{\arg\!\min}

\begin{gather*}
\argmin_{\theta} \Vert p - y(\theta) \Vert \\
\text{subject to} \\
\sum_{j=1}^n \theta_j = 1, \text{and}\\
\theta_j > 0~\text{for all}~j.
\end{gather*}
$$

This is a standard
[constrained minimization problem](https://en.wikipedia.org/wiki/Constrained_optimization).
Our expectation is that any solution $$\theta$$
of the above system
_(a)_ fits the training set well, and
_(b)_ also predicts metascores for new movies.

The $$m$$-vector $$d(\theta) = p - y(\theta)$$
represents the _error_ or distance between
the actual metascores and predicted ones for
a given $$\theta$$.
Notice that $$d$$ is not a _linear_ function of $$\theta$$
(because of the reciprocal term in $$y(\theta)$$).
So, we have to use a
[nonlinear solver](https://en.wikipedia.org/wiki/Nonlinear_programming).
I used the well known
[Sequential Least Squares Programming solver](http://docs.scipy.org/doc/scipy-0.13.0/reference/generated/scipy.optimize.fmin_slsqp.html)
available as part of
[scipy.optimize](http://docs.scipy.org/doc/scipy-0.13.0/reference/optimize.html).

Also, because a lot of solvers work on _unconstrained_ problems,
we modify our objective function $$f(\theta) = \Vert p - y(\theta) \Vert$$
by also include the constraints.

Consider the _tub_ function $$t(x, l, u)$$ defined as:

$$
t(x, l, u) =
  \begin{cases}
    1       & \quad \text{if } l \leq x \leq u, \\
    0       & \quad \text{otherwise}.
  \end{cases}
$$

So, our modified objective function becomes:

$$
f(\theta) = \Vert p - y(\theta) \Vert + P_h \times \Vert \sum_{j=1}^n 1 - \theta_j \Vert + P_b \times \sum_{j = 1}^n t(\theta_j, 0, 1).
$$

### The Implementation

~~~
# preliminaries
r = <m x n rating matrix>
r_dash = <m x n rating matrix with holes filled>
e = <m x n indicator matrix>
x = <m vector of published metascores>

def y(theta):
    """theta is an n x 1 column vector"""
    numerator = r_dash * theta   # each of these is an m-vector:
    denom = e * theta            # (m x n) times (n x 1).
    # element-wise division
    return ewd(numerator, denom)

def error_fn(theta):
    """returns an m vector"""
    return x - y(theta)

def objective(theta):
    """ Objective function """
    return scipy.linalg.norm(error_fn(theta))

# all theta values are positive
theta_bound = (1e-9, None)
theta_bounds = theta_bound * m

constraints = ({
    'type': 'eq',
    'fun' : lambda theta: sum(theta) - 1},)

theta_initial = 1.0/n * n
res = minimize(objective,
               theta_initial,
               bounds=theta_bounds,
               constraints=constraints,
               method='SLSQP')
~~~
