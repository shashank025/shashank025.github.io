---
layout: post
title: A Graph Theoretic Approach to the Bay Bridges Challenge
date: 2013-12-16 12:55:34.000000000 -08:00
tags:
- Bay Bridges Challenge
- intersection graph
- minimum vertex cover
- computer science
- math
status: publish
type: post
published: true
meta:
  geo_accuracy: '0'
  switch_like_status: '1'
  _publicize_pending: '1'
  geo_public: '1'
  _edit_last: '4244661'
  geo_latitude: '0.000000'
  geo_longitude: '0.000000'
author:
  login: shashankr
  email: shashank.ramaprasad@gmail.com
  display_name: shashankr
  first_name: ''
  last_name: ''
excerpt: !ruby/object:Hpricot::Doc
  options: {}
---

I recently came across the <a title="Bay Bridges Challenge" href="http://blog.codeeval.com/bridges/">Bay Bridges Challenge</a>,
over at
<a title="CodeEval" href="https://www.codeeval.com/">CodeEval</a>.
The challenge is to pick the largest set of bridges for construction,
such that no two bridges cross or <em>intersect</em>.

My first instinct was to model this as a
<a title="Wikipedia page on Graph Theory" href="http://en.wikipedia.org/wiki/Graph_theory">graph</a> problem.
Suppose we construct an <a title="Wikipedia page on Intersection Graphs" href="http://en.wikipedia.org/wiki/Intersection_graph"><em>intersection graph</em></a>
<strong>G</strong>, as follows:
create a graph node (or a <em>vertex</em>) corresponding to each bridge,
and add an edge between two vertices <em>u</em> and <em>v</em>
if and only if the corresponding bridges cross.
Figure 1 below shows an example.
Then, its easy to see that the original problem is equivalent to
finding the <a title="Wikipedia page on Vertex Cover" href="http://en.wikipedia.org/wiki/Vertex_cover">minimum vertex cover</a> on G.

<a href="https://docs.google.com/drawings/d/1bWFPkc1C-RjBSOSsaK_VUUA4FZ6EaKpABrT7iuN9BL4/pub?w=960&amp;h=720">
<img
    width="480" height="360"
    src="/assets/images/bay_bridges.png"
    title="A Sample Intersection Graph"
    alt="A Sample Intersection Graph" />
</a>
_Figure 1:_ A Sample Intersection Graph

<p>It turns out that finding the minimum vertex cover is <a title="Wikipedia page on NP-hard problems" href="http://en.wikipedia.org/wiki/NP-hard">NP-hard</a>. For this problem, it means (roughly) that we can do <em>no better</em> than to iterate through the <a title="Wikipedia page on Power Sets" href="http://en.wikipedia.org/wiki/Power_set">power set</a> of bridges, and pick the highest cardinality subset that is <em>feasible</em> (i.e., no two bridges in that subset cross). That was a bit disheartening. But it is unlikely that a run-of-the-mill coding challenge requires coming up with acceptable approximation algorithms to hard problems. Most likely, there is additional structure inherent in the problem, that can be exploited to make the problem more tractable.</p>
<p>Since the bridges exist in a 2-D plane (roughly), we can use the <a title="Euclidean Distance" href="http://en.wikipedia.org/wiki/Euclidean_distance">distance metric</a> to automatically rule out large portions of the search space. For example, if a bridge does not cross any other bridge, then it is always included in every optimal solution. By eliminating such bridges from consideration, we might reduce the problem size considerably. Spatial data structures like bounding rectangles, <a title="k-d trees" href="http://en.wikipedia.org/wiki/K-d_tree">k-d trees</a>, or their cousins can be used to partition the set of bridges into smaller subsets, and solve many smaller, independent sub-problems (maybe even in parallel). But none of these approaches reduce the essential <em>strength</em> of the problem, since we are still looking at exponential worst-time solutions.</p>
<p>At this point, I was feeling a bit stuck. My attempts at finding additional structure in the original problem space hadn't really helped. But going over the Wikipedia page on <a title="Wikipedia page on Vertex Cover" href="http://en.wikipedia.org/wiki/Vertex_cover">vertex covers</a>, I read that a minimum vertex cover can be found in polynomial time for <a title="Wikipedia page on Bipartite Graphs" href="http://en.wikipedia.org/wiki/Bipartite_graph">bipartite graphs</a>. Finally, some progress: If the intersection graph G is bipartite (a graph's bipartite-ness can be tested in <em>linear</em> time), we have an efficient solution. But it is not hard to come up with real world inputs where the intersection graph is <em>not</em> bipartite. So, we still don't have a solution that always works.</p>
<p><strong>Update: </strong><em>As you can see in Figure 1 above, the intersection graph is not actually bipartite. However, if you imagined that bridge b4 was removed from the input, the remaining graph would indeed become bipartite.</em></p>
<p>As of this point, I haven't written a line of code. My hunch is that I need to look for a solution in the original problem space: the roughly Euclidean plane on which the line segments corresponding to the bridges exist, and not in the intersection graph space.</p>
