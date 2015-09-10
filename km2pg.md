---
layout: page
title: km2pg - Kissmetrics Analytics to Postgres
permalink: /km2pg/
---
{% include JB/setup %}

**km2pg** is open source software to load
your website/app analytics data stream from
[Kissmetrics](https://www.kissmetrics.com/)
into a
[Postgres](http://www.postgresql.org/) database
of your choice,
enabling real time business insights
using the full power of SQL.


##### Table of Contents
{:.no_toc}

* TOC
{:toc}


#### Source code

_Coming soon_


#### Intended Audience

* _Small and Medium Businesses_ (SMB's)
who have signed up for Kissmetrics
and would like to ask
more sophisticated questions than the ones allowed
via the Kissmetrics interfaces.
 
* _Enterpreneurs_ with ideas for building
solutions for SMB's atop web/app analytics data.

In its current form,
this software will most likely be a better fit
for SMB's rather than really large advertisers,
because no provision is made to account for
the case when the total data size exceeds
the storage capacity of a single machine.
Of course, the library comes with a _pruner_
that can remove older data (_how old_ is configurable,
of course), which mitigates the issue somewhat.
I have ideas on how to scale this up
to support storing larger volumes of
analytics data, but I wanted to
get this version out there first.

Note that this software was already successfully used
by
[Fracture](http://www.fractureme.com/),
an innovative photo printing company
out of Florida, to enable targeted Facebook marketed campaigns.


#### Why Postgres?

I know all the cool kids are hawking their NoSQL solutions
for analytics these days, but it is actually quite possible
to use Postgres, which is a relational database,
to store and query analytics data, for the following reasons:

* Postgres is open source, which means you won't just wake up
one day to find the database your business runs on is
[no longer available for download](https://news.ycombinator.com/item?id=9259986),
or worked on, or developed in any way,
and you don't even have access to the code if you'd
like to make changes yourself.

* Postgres is high quality software, generally well thought out,
and mature, with an active development community.

* The high quality query planner/optimizer means
you write idiomatic SQL, and your queries will
generally be well behaved.

* New, NoSQL-like features allow you to have your cake
and eat it too.

* Tons of exciting upcoming improvements, especially
related to scaling, can actually make Postgres
viable for "big data" usecases as well.

In some ways, this software is an _experiment_ to see
how far the envelope can be pushed.


#### How does it work?

Kissmetrics has a
[data export](http://support.kissmetrics.com/apis/data/index.html)
feature,
which basically enables programmatic access
to the raw stream of analytics data from your website/app.

The basic idea is to map the Kissmetrics analytics
data model into a (quasi) relational schema.
Our library creates a new _schema_ called `km2pg`
on the specified Postgres database
as part of the setup.
The library then uploads analytics data
into tables in this schema.
For example, `events` is
the central [_fact table_](https://en.wikipedia.org/wiki/Fact_table),
with one row for each recorded analytics event.

If you wanted to know
_how many orders were placed in the last week_,
a query like the following could do the job:

~~~
select count(1)
  from events e
  join event_actions using (eventid) -- an event in KM can have more than one action
  join actions a using (actionid)
 where a.name = 'checkout successful'
   and e.t >= now() - interval '1 week';
~~~


#### Future Work

* Scale up the solution for the large advertiser use case.
In particular, allow loading of data into one of
multiple physical machines (sharding).

-----
