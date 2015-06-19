---
layout: post
title: "Reverse engineering Metacritic"
description: ""
category: 
tags: [optimization,metacritic,machine learning,visualization,math,computer science]
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
using standard machine learning/optimization techniques.
In fact, if we do this right, not only will we able to
tell what critics are more important than others,
but we should also be able to correctly predict the metascore
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
     (why even consider a critic whose rating does not affect the metascore at all?).
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

But there is a problem with this definition. Remember:
 _Not all critics rate all movies_.
In other words, the summation above may be invalid,
since not all $$r_{ij}$$ values are necessarily defined.
How do we deal with this _incomplete_ matrix $$r_{ij}$$?
My best guess is that metacritic normalizes the metascore
over the available critic weights.
For example,
assume that  the movie
[Mad Max: Fury Road](http://www.imdb.com/title/tt1392190/),
has an index $$i = 4$$ in our data set.
Assume that only
two critics, with weights $$\theta_1$$ and $$\theta_2$$
have rated this movie,
and their ratings are $$r_{41} = 79$$ and $$r_{42} = 67$$ respectively.
Its metascore is then

$$
    y_4(\theta_1, \theta_2) = \frac{\theta_1 r_{41} + \theta_2 r_{42}}{\theta_1 + \theta_2}.
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

How does this function vary with $$\theta$$?
I wrote up
[ a little script to plot $$y_4 (\theta_1, \theta_2)$$](https://github.com/shashank025/metacritic-weights/blob/master/ytheta.py)
for the Mad Max example from above.
The following image of the plot makes it clear that
$$y_i (\theta)$$ is _not_ linear in $$\theta$$.
But the function is still _smooth_ (_i.e._, _differentiable_).

<a href="/assets/images/plot-of-y-theta.png">
<img
    width="400" height="300"
    src="/assets/images/plot-of-y-theta.png"
    title="Plot of y(theta) - click to zoom"
    alt="Plot of y(theta) - click to zoom" />
</a>

Now consider the $$m$$-vector $$d(\theta) = p - y(\theta)$$.
This vector represents a measure of how _off_ the predictions
are from actual metascores for
a given $$\theta$$.
We will try to find a $$\theta$$
that minimizes the value of the function
$$f(\theta) = \Vert d(\theta) \Vert$$,
where $$\Vert \cdot \Vert$$ represents the
[$$L^2$$ norm](https://en.wikipedia.org/wiki/Lp_space).
Formally,

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
Notice that $$d$$ is not a _linear_ function of $$\theta$$
because $$y(\theta)$$ isn't either.
So, we have to use a
[nonlinear solver](https://en.wikipedia.org/wiki/Nonlinear_programming).
Further, many solvers only work on _unconstrained_ problems.
So, we employ a common technique in optimization formulations,
which is to push the constraints _into the objective function_.
Consider the "tub" function $$t(x, l, u)$$ defined as:

$$
t(x, l, u) =
  \begin{cases}
    0       & \quad \text{if } l \leq x \leq u, \\
    1       & \quad \text{otherwise}.
  \end{cases}
$$

Our modified objective function becomes:

$$
f(\theta) = \Vert p - y(\theta) \Vert
     + P_h \cdot \Vert 1 - \sum_{j=1}^n \theta_j \Vert
     + P_b \cdot \sum_{j = 1}^n t(\theta_j, 0, 1),
$$

where $$P_h$$ and $$P_b$$ are configurable weights that decide
how much we should _penalize_ the optimization algorithm when it
chooses a $$\theta$$ that doesn't lie on the affine hyperplane,
and when it chooses $$\theta_j$$ values outside the interval
$$[0, 1]$$ respectively.

### The Implementation

With all that annoying math out of the way, lets write code!
The implementation pipeline consists of the following stages:

1. Collect movie ratings data from metacritic.
2. Preprocess the data:
    * Remove ratings from critics who've rated very few movies, and
    * Create the $$r'_{ij}$$ and $$e_{ij}$$ matrices.
3. Partition the data into a _training_ set and a _test_ set.
4. Find a best fit $$\theta$$ by running the optimization routine on the training set.
5. Compute accuracy against the test set.
6. Output the results.

It turns out that a Makefile is really well suited to
building these kinds of pipelines,
where each stage produces a _file_ that can be used as a Make target
for that stage.
Each stage can be dependent on files produced in one or more previous stages.

#### Collecting ratings data from metacritic

Unfortunately, metacritic does not, as far as I know,
have any API's to make this data available easily.
So I periodically scrape metacritic's
[New Movie Releases page](http://www.metacritic.com/browse/movies/release-date/theaters/metascore?view=condensed)
for links to actual metacritic movie pages,
which I then scrape to get the overall metascore,
and the individual critic ratings.

I used a combination of
[xpathtool](http://www.semicomplete.com/projects/xpathtool/)
and
the [lxml Python library](http://lxml.de/)
for the scraping.

The output of this stage is a
[Python cPickle](https://docs.python.org/2/library/pickle.html)
dump file that represents a dictionary of the form:

    { movie_url -> (metascore, individual_ratings), ... }

where `individual_ratings` is itself a dictionary of the form

    { critic_name -> numeric_rating, ... }.

For example, this structure could look like:

~~~
{
    'http://www.metacritic.com/movie/mad-max-fury-road/critic-reviews' ->
         (89,
          {'Anthony Lane (The New Yorker)': 100,
           'A.A. Dowd (TheWrap)': 95,
           ...}),
    'http://www.metacritic.com/movie/ex-machina/critic-reviews' ->
         (78,
          {'Steven Rea (Philadelphia Inquier)': 100,
           'Manohla Dargis (The New York Times)': 90,
           ...}),
    ...
}
~~~
I know cPickle is not exactly the most portable format,
but it works well at this early stage.
In the long run, I want to persist all of the ratings data
in a database (sqlite? Postgres?).

#### Preprocessing

The first thing we do is to eliminate from our data set
the long tail of critics who've rated very few movies.
Not only are these critics not very influential,
eliminating them also
helps reduce $$n$$ (the matrix width).
Accordingly, there is a configurable _rating count threshold_,
currently set to $$5$$.
We do one pass over the ratings data and construct
a dictionary of the form:

    { critic_name -> movies_rated, ... }

We then do another pass through the data and remove ratings
from critics whose `movies_rated` value is lower than the
threshold.

The second preprocessing step is to construct the
$$r'_{ij}$$ and $$e_{ij}$$ matrices, which of course
is
[a simple matter of programming](https://en.wikipedia.org/wiki/Small_matter_of_programming).
I store these values as
[numpy matrices](http://docs.scipy.org/doc/numpy/reference/generated/numpy.matrix.html).

#### Partitioning the data set

This is straightforward.
I use a configurable `training_frac` parameter
(a value in the interval $$[0, 1]$$) to probabilistically
split the cleaned up data into a test set and a training
set.

#### Optimization routine

I tried the following solvers, 
available as part of
[scipy.optimize](http://docs.scipy.org/doc/scipy-0.13.0/reference/optimize.html):

* [Sequential Least Squares Programming (SLSQP)](http://docs.scipy.org/doc/scipy-0.13.0/reference/generated/scipy.optimize.fmin_slsqp.html)
* [Constrained Optimization By Linear Approximations (COBYLA)](http://docs.scipy.org/doc/scipy-0.14.0/reference/generated/scipy.optimize.fmin_cobyla.html)

SLSQP allows us to specify both
_bounds_ and _constraints_ on the optimization system.
In fact, one can specify an arbitrary number of
equality and inequality constraints.
COBYLA, on the other hand, doesn't support bounds,
or equality constraints.

### Results

_Coming soon._




