# Filtering results with CQL in pg_featureserv

The goal of [`pg_featureserv`](https://github.com/CrunchyData/pg_featureserv)
is to provide easy and efficient access to [PostGIS](https://postgis.net/) data from web clients.  
To do this it uses the emerging [*OGC API for Features*](https://ogcapi.ogc.org/features/)
(OAPIF) REST protocol, which is a natural fit for systems which need to query and communicate spatial data.
The [core OAPIF](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) specification
provides a basic framework for querying spatial datasets, but it has only limited capability 
to express filtering subsets of spatial tables.  In particular, it only allows filtering on single attribute values,
and it only supports spatial filtering via the `bbox` query parameter (in PostGIS terms, this is equivalent to using the `&&` operator with a `box2d`).

Of course, PostGIS and PostgresQL provide much more powerful filtering capabilities. 
It would be very nice to be able to access them via `pg_featureserv`.
Luckily, the OGC has defined the Common Query Language (CQL) which (as the name suggests) is a close match to SQL filtering capabilities.
This is being issued under the OGC API umbrella as CQL2 (currently in draft).

Recently we added `pg_featureserv` support for most of CQL2.
Here we'll describe the powerful new capability it provides.

## Overview of CQL

CQL is a language which describes boolean expressions which determine which features are included in the result of a query.
It is *very* similar to SQL.
A CQL expression can include feature property values, and constants including numbers, booleans and text values.
Conditions can be expressed using comparisons (`<`,`>`,`<=`,`>=`,`=`,`<>`) and predicates:
```
prop IN (val1, val2, ...)
prop BETWEEN val1 AND val2
prop IS [NOT] NULL
prop LIKE | ILIKE pattern
```
Conditions can be combined with the boolean operators `AND`,`OR` and `NOT`.

CQL also defines syntax for spatial and temporal filters. We'll discuss those in a future blog post.

## Filtering with CQL

A CQL expression can be used in a `pg_featureserv` request in the `filter` parameter.  
This is converted to SQL and included in the `WHERE` clause of the underlying database query.
(Of course, this allows the database to use its query planner and any defined indexes to execute the query very efficiently.)
Here's an example:

* query world continent IN ('Europe','Asia') AND pop_est BETWEEN 1000000 AND 9000000

* example of ILIKE to replace function in previous blog post



## Stay tuned

As promised above, we'll be publishing a follow-up post on the spatial filtering capabilities of CQL soon.
And there's some other interesting capabilites in `pg_featureserv` which we'll discuss in a further post.


