---
layout: post
title: "Reverse engineering the Metacritic weights for movie critics"
description: ""
category: 
tags: []
---
{% include JB/setup %}

[metacritic](http://www.metacritic.com/) is a popular site that computes an
aggregate _metascore_ for each movie as a weighted average of individual critic scores.
The [Wikipedia article on metacritic](http://en.wikipedia.org/wiki/Metacritic) says:

> Reviews from major periodicals may have a greater effect on average than niche ones ...

Tantalizingly, the [metacritic FAQ Page](http://www.metacritic.com/faq) says:

> Q: Can you tell me how each of the different critics are weighted in your formula?
>
> A: Absolutely not. 

That sounds like a challenge to me.
It _should_ be possible to infer the weights using standard machine learning techniques.
In fact, if we do this right, we should be able to correctly predict the metascore
for any new movie (given the individual critic ratings).

This post describes my attempt to build such a system.

### Notation and Constraints

We introduce the following notations and assumptions:

* metacritic uses ratings from $$m$$ critics.
* Our data set has $$n$$ movies.
* $$x_{ij}$$ is the rating of movie $$i$$ by critic $$j$$.
   * Not all critics rate all movies!
     In other words, $$x_{ij}$$ is not defined for all $$i$$ and $$j$$.
   * Where defined, the values are constrained: $$0 \leq x_{ij} \leq 100$$.
* $$\theta_j$$ is the _relative weight_ (or importance) of critic $$j$$
(this is what we are trying to _learn_).
   * Critic weights must be _positive_ (otherwise, why consider the critic at all),
     _i.e._, $$\theta_j > 0$$ for all $$j$$,
   * Critic weights must add up to one, _i.e._, $$\sum_{j=1}^m \theta_j = 1$$.
     In other words, the solution space forms a _bounded hyperplane_.
* $$p_i$$ is the _published_ metascore for movie $$i$$.
   * These values are also constrained: $$0 \leq p_{i} \leq 100$$ for all $$i$$.
* $$y_i$$ is the _predicted_ metascore for movie $$i$$.

While it is clear that $$y_i$$ is a function of the $$x_{ij}$$ and $$\theta_j$$ values,
we will defer the exact definition until a little later.

## Objective

The $$m$$-dimensional vector
$$\theta = (\theta_1, \theta_2, \ldots, \theta_m)$$
represents _a_ possible assignment of weights to critics.
Our task is to find a $$\theta$$ that (a) fits our data set
well, and (b) is capable of predicting ratings for movies
not in the training set.
On the face of it, this seems to be a standard
least squares problem of finding a $$\theta$$ that
minimizes the value of the following expression
(subject to the stated constraints):

$$
\sum_{i = 1}^{n} (p_i - y_i)^2.
$$

In fact, it looks like
$$y_i$$, the predicted metascore for movie $$i$$ is
simply a weighted sum:

$$
y_i = \sum_{j=1}^m \theta_j x_{ij}.
$$

But remember:
 _Not all critics rate all movies_.

In other words, the summation above is not valid, since
not all the $$x_{ij}$$ values are _defined_.
How do we deal with this _incomplete_ matrix $$x_{ij}$$?
My best guess is that metacritic _normalizes_ its scores
using the available ratings.
For example, if the movie with $$i = 4$$
was only rated by two critics ($$j = 2$$ and $$j = 7$$), we would get:

$$
    y_4 = \frac{\theta_2 x_{42} + \theta_7 x_{47}}{\theta_2 + \theta_7}.
$$

In fact, the metacritic FAQ page says they wait until
a movie has at least 4 reviews before computing a metascore.
In other words, they want at least 4 defined $$x_{ij}$$ values
for a given $$i$$ before computing $$y_{ij}$$.
Lets define the following additional variables:

* $$x'_{ij} = x_{ij}$$ if movie $$i$$ is rated by critic $$j$$
and $$0$$ otherwise.
* $$e_{ij} = 1$$ if movie $$i$$ is rated by critic $$j$$, and $$0$$ otherwise.

We can now generalize the _modified_ definition for the
predicted metascore for movie $$i$$ as:

$$
y_i = \frac{ \sum_{j=1}^m \theta_j x'_{ij} }{ \sum_{j=1}^{m} \theta_j e_{ij} }.
$$

The point of discussing the complication above is that
we _cannot_ use standard linear regression, since
there are _holes_ in the $$x_{ij}$$ matrix.

### Complications


$$
\theta_1 x_{i1} + \theta_2 x_{i2} + \ldots + \theta_n x_{in}.
$$



Formal model: Constrained Linear Least Squares

With some simple algebra involving rearranging of terms above, we realize that we can formulate our problem thus:

With a training set of m movies, find a value for theta (a vector) that best fits this series of equations:

  theta[1] * z[i][1] + theta[2] * z[i][2] + ... + theta[n] * z[i][n] = 0     for i = 1, 2, ..., m,

where:

    z[i][j] = x[i][j] - y[i]     if movie i was rated by critic j
    z[i][j] = 0                  otherwise

Expressed in matrix-vector form, our task is to find a value for the vector theta that best fits the equation

    z . theta = 0

subject to the constraints described above. This is the classic linear least squares model with constraints.

Note that in the above formulation:

    z, theta and 0 are vectors of dimensions m x n, n x 1 and m x 1 respectively,
    the dot (.) represents matrix multiplication.
    m >> n: there are way more movies than critics. In other words, the system is overdetermined.
    Quite a few elements of z are zeros. 