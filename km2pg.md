---
layout: page
title: km2pg - Kissmetrics to Postgres
permalink: /km2pg/
---
{% include JB/setup %}

[Kissmetrics](https://www.kissmetrics.com/) (abbreviated as KM when convenient)
is a web/app analytics provider.
They have a 
[data export](http://support.kissmetrics.com/apis/data/index.html)
feature,
which basically enables programmatic access
to the raw stream of analytics data from your website/app.
**km2pg** is an open source library that
lets you set up and (continuously) update
a [Postgres](http://www.postgresql.org/) database with this data,
enabling real time insights over your entire data
using the full power of SQL.

Source code: _Coming soon_

### How does it work?

The basic idea is to map the Kissmetrics analytics
data model into a relational schema.
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
  join event_actions using (actionid) -- an event in KM can have more than one action
  join actions a using (actionid)
 where a.name = 'checkout successful';
   and e.t >= now() - interval '1 week';
~~~

-----