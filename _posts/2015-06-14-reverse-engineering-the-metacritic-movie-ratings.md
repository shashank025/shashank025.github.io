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
For example, assume that the (excellent) movie
[Ex Machina](http://www.imdb.com/title/tt0470752/)
has the index $$i = 4$$ in our data set.
Assume that only two critics,
with weights $$\theta_1$$ and $$\theta_2$$
have currently rated this movie.
We denote their ratings $$r_{41}$$ and $$r_{42}$$ respectively.
The metascore for this movie is then

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

How does this function vary with $$\theta$$ (once we fix the $$r_{ij}$$ values)?
I wrote up
[a little script to plot $$y_4 (\theta_1, \theta_2)$$](https://github.com/shashank025/metacritic-weights/blob/master/ytheta.py)
for the example involving the movie _Ex Machina_
(I fixed the critic ratings to
$$r_{41} = 79$$ and $$r_{42} = 67$$;
I know. Stupid critics!)
The following image of the plot
hopefully makes it clear that
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

### The Implementation

With (most of the) annoying math out of the way, lets write code!
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

We first eliminate from our data set
the long tail of critics who've rated very few movies.
Not only are these critics not very influential
to the overall optimization routine,
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

There are numerous "solvers" available for
constrained optimization problems of the type we
described above, but not all of them are
freely available.

I tried the following two solvers, 
available as part of
[scipy.optimize](http://docs.scipy.org/doc/scipy-0.13.0/reference/optimize.html):

| Solver | Differentiability requirements | Allows bounds? | Allows equality constraints? | Allows inequality constraints? |
|---------|
| [Sequential Least Squares Programming (SLSQP)](http://docs.scipy.org/doc/scipy-0.13.0/reference/generated/scipy.optimize.fmin_slsqp.html) | The objective function and the constraints should be twice [continuously differentiable](https://en.wikipedia.org/wiki/Differentiable_function#Differentiability_classes) | Yes | Yes | Yes |
| [Constrained Optimization By Linear Approximations (COBYLA)](http://docs.scipy.org/doc/scipy-0.14.0/reference/generated/scipy.optimize.fmin_cobyla.html) | None | No | No | Yes |
|---------|


.

Note that $$y(\theta)$$ (and therefore $$f(\theta)$$) satisfies
the differentiability requirement of SLSQP.

Also, COBYLA does not allow you to specify bounds on
$$\theta$$ values or equality constraints.
So, we employ a common technique in optimization formulations,
which is to push the constraints _into the objective function_.
Consider the "tub" function $$\tau(x, l, u)$$ defined as:

$$
\tau (x, l, u) =
  \begin{cases}
    0       & \quad \text{if } l \leq x \leq u, \\
    1       & \quad \text{otherwise}.
  \end{cases}
$$

Our modified objective function (for use with COBYLA) becomes:

$$
f(\theta) = \Vert p - y(\theta) \Vert
     + P_h \cdot \Vert 1 - \sum_{j=1}^n \theta_j \Vert
     + P_b \cdot \sum_{j = 1}^n \tau(\theta_j, 0, 1),
$$

where $$P_h$$ and $$P_b$$ are configurable weights that decide
how much we should _penalize_ the optimization algorithm when it
chooses a $$\theta$$ that doesn't lie on the affine hyperplane,
and when it chooses $$\theta_j$$ values outside the interval
$$[0, 1]$$ respectively.
Setting both $$P_h$$ and $$P_b$$ to 0 reduces our objective
function to its original form, so we can use the same
function for both solvers by simply tweaking these weights.

### Results

After removing ratings from insignificant critics,
I constructed a training set of about 2800 ratings
of 190 movies by 188 critics.
Here are some findings from running the solver
on this training set:

* I was actually unable to get either SLSQP or COBYLA
to ever successfully converge on a solution.
Most of the times, both routines finished their iterations
and failed with errors of this form:

~~~
optimization failed [8]: Positive directional derivative for linesearch
optimization failed [2]: Maximum number of function evaluations has been exceeded
~~~

* The $$\theta$$ values (_i.e._, critic weights)
learned by these solvers
were often _way_ outside the interval $$[0, 1]$$.
I am not sure why this is so.

If you have experience with the numpy optimization
library, I'd love to hear about suggestions you may
have on how to fix these errors.

However, what was almost equally interesting was that the
learned $$\theta$$ values were still able to successfully
predict metascores for movies in the test set.

The following table lists
the critic weights learned using the above training set
with each optimization routine.
Note that the weights are expressed as a percentage
of the weight for the _top_ critic in each list.
So, for example, according to SLSQP,
a review by Noel Murray is
given 64% of the importance that is given
to a review by Mike McCahill.

| SLSQP || COBYLA ||
| Weight | Critic | Weight | Critic |
 :-: | :- | :-:|:- 
| $$\cdot$$ | Mike McCahill (The Telegraph)              | $$\cdot$$ | Entertainment Weekly
| 68.396474 | Dave Calhoun (Time Out London)             | 99.982343 | Jessica Kiang (The Playlist)
| 64.046236 | Noel Murray (The Dissolve)                 | 64.634147 | Mike D'Angelo (The A.V. Club)
| 44.510982 | Fionnuala Halligan (Screen International)  | 51.022980 | Marc Savlov (Austin Chronicle)
| 41.091489 | Katie Walsh (Los Angeles Times)            | 46.038287 | Tom Russo (Boston Globe)
| 36.862072 | Stephen Farber (The Hollywood Reporter)    | 33.931881 | Mike McCahill (The Telegraph)
| 34.713074 | Mike Scott (New Orleans Times-Picayune)    | 29.630390 | Leslie Felperin (The Hollywood Reporter)
| 31.991243 | Jesse Cataldo (Slant Magazine)             | 27.551365 | Jay Weissberg (Variety)
| 29.335468 | Roger Moore (Tribune News Service)         | 22.482233 | The Guardian
| 28.038652 | Nicolas Rapold (The New York Times)        | 21.749581 | David Parkinson (Empire)
| 27.924488 | Tom Russo (Boston Globe)                   | 19.680719 | Andy Webster (The New York Times)
| 27.193219 | Tirdad Derakhshani (Philadelphia Inquirer) | 16.637660 | Sheila O'Malley (RogerEbert.com)
| 24.961678 | Stephanie Merry (Washington Post)          | 16.253025 | Marjorie Baumgarten (Austin Chronicle)
| 24.093929 | Slant Magazine                             | 16.076699 | Bob Mondello (NPR)
| 22.892765 | Carson Lund (Slant Magazine)               | 15.954304 | Dennis Harvey (Variety)
| 21.849781 | Jesse Hassenger (The A.V. Club)            | 15.867666 | Steve Macfarlane (Slant Magazine)
| 20.202264 | Neil Genzlinger (The New York Times)       | 15.829029 | Simon Abrams (RogerEbert.com)
| 19.658871 | Jen Chaney (The Dissolve)                  | 15.726588 | Gary Goldstein (Los Angeles Times)
| 19.328043 | Joe Leydon (Variety)                       | 15.685218 | Noel Murray (The Dissolve)
| 19.110881 | Mark Olsen (Los Angeles Times)             | 15.604939 | Richard Brody (The New Yorker)

.

I should clarify that the top 20 list can change from one run to the next,
since it depends on the training set we choose, and the kind of explorations
of the search space performed by the solvers in their iterations.

We next show the _accuracy_ of the learned critic weights
in predicting the metascore for movies in the test set.
Our test set had about 60 movies.

The following table lists the RMSE error between
the predicted and the actual metascores for each optimization
routine.

| SLSQP | COBYLA |
|----------------|
| 1.042976 | 0.719814 |

.

For example, the excellent Scientology documentary from HBO,
[Going Clear](http://www.imdb.com/title/tt4257858/),
has
[a metascore of 81](http://www.metacritic.com/movie/going-clear-scientology-and-the-prison-of-belief).
SLSQP predicted a score of 81.32 while COBYLA predicted a score of 84.16
for this documentary.
Not bad!



## Takeaways

Working out the math was obviously fun!
I got to brush up on my long defunct skills with numpy, scipy, matplotlib, etc.
But I also realized that even a toy project
like this one can end up demanding
substantial time and attention, especially if it is something you want to share
with the rest of the world.
For example, I ended up using `setup.py` and creating a proper
Python package for all the code.
Writing out this blog post was also extremely useful because
it clarified my own thinking on the topic. I was actually
able to go back and refactor the code to better match
the implementation pipeline I described above.
If anything, this exercise has strengthened my resolve to write
about a lot more frequently.
