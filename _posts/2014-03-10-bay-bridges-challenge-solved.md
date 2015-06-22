---
layout: post
title: Bay Bridges Challenge (solved!)
date: 2014-03-10 01:44:15.000000000 -07:00
tags:
- algorithms
- Bay Bridges Challenge
- complexity
- computer science
- graph theory
- programming
status: publish
type: post
published: true
meta:
  geo_public: '0'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _edit_last: '4244661'
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
<p>I <a title="previous post on Bay Bridges Challenge" href="http://shashankr.wordpress.com/2013/12/16/a-graph-theoretic-approach-to-the-bay-bridges-challenge/">previously wrote</a> about the <a title="Bay Bridges Challenge" href="https://www.codeeval.com/open_challenges/109/">Bay Bridges Challenge</a>, hosted at <a title="CodeEval" href="https://www.codeeval.com/">CodeEval</a>. In my last post, I showed that the problem could be modeled as a <a title="Wikipedia page on Vertex Cover" href="http://en.wikipedia.org/wiki/Vertex_cover">minimum vertex cover</a> problem, and wondered if we can do better than iterating through the <a title="Wikipedia page on Power Sets" href="http://en.wikipedia.org/wiki/Power_set">power set</a> of bridges, and picking the highest cardinality subset that is feasible. I said that most likely,</p>
<blockquote><p>there is additional structure inherent in the problem, that can be exploited to make the problem more tractable.</p></blockquote>
<p>Spurred by some <a title="helpful comments on previous post about Bay Bridges challenge" href="http://shashankr.wordpress.com/2013/12/16/a-graph-theoretic-approach-to-the-bay-bridges-challenge/#comment-68">helpful recent comments</a>, I spent some more time on the problem. As it turns out, we <em>can</em> do better than an exhaustive search. But first, remember that a <strong><em>feasible solution</em></strong> is any set of bridges with <strong>no</strong> intersections. Our task is to find <em>an</em> <strong><em>optimal solution</em></strong>, which is simply the <em>largest</em> such feasible solution (note that there can be more than one).</p>

<strong><em>Claim 1:</em></strong> If there is no feasible solution with $$ k $$ bridges, then there cannot be a larger feasible solution.

<em>Proof (by contradiction):</em> Assume that there <em>is</em> a solution with $$ l = k + 1 $$ bridges. The removal of any bridge from this solution should still be a set of non-intersecting bridges, <em>i.e.</em>, a feasible solution, but of size $$ k $$, which is a contradiction. By induction, we can see that this is also true for higher values of $$l$$. QED.

With this claim in hand, let us partition the power set of $$ n $$ bridges so that $$ p(i) $$ represents
the set of all sets of bridges of length $$ i $$. We can then make the following observation about the
$$ n $$ partitions (we can safely ignore the empty partition $$ p(0) $$):

> If any set in $$p(n/2)$$ is <em>feasible</em>, the <em>optimal</em> solution is
> in one of the partitions $$p(n/2)$$ through $$p(n)$$.
> Otherwise, it is in one of the partitions $$ p(1) $$ through $$ p(n/2 - 1) $$.

In other words, we can do a <em>binary search</em> on the partition index $$ i $$
until we find the partition with the optimal solution. While this seems very promising,
the partition size $$ |p(i)| = {n \choose i} $$ unfortunately has a
<em>maximum</em> at $$ i = n/2 $$ (note that $$ |p(n/2)| = {n \choose n/2} \approx 2^n $$).
So, in the worst case, we still need to search an exponential number of bridge sets.d
 But in practice, this should still be significantly better than exhaustively searching each partition.

Anyway, I didn't really feel like coding up this binary search, so I tried submitting a simpler,
more naive solution: Search each partition $$ p(i) $$, starting with $$ p(n) $$,
in decreasing order of $$ i $$, until you find the <em><strong>first</strong></em> feasible solution $$ f $$, and return $$ f $$.
Because of <em>Claim 1</em>, it is easy to see that $$ f $$ <em>will</em> be
<em>an</em> optimal solution to our problem.
It turns out that this approach was enough to get a score of 100 (with a ranking of 86).
So, now I am even less inclined to implement binary search.

<p>Also, I should really post the code for all of my solutions to github, which I hope to do soon.</p>
<p><em><strong>Update:</strong></em> I have checked in <a title="bay bridges solution code (Python)" href="https://github.com/shashank025/codeeval/blob/master/bridges.py">solution code for the Bay Bridges challenge</a> and other codeeval challenges at github. Check it out at <a title="my github repo for codeeval challenges" href="https://github.com/shashank025/codeeval">my github</a>.</p>

----