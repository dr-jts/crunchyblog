# Filtering pg_featureserv results with CQL

The goal of [`pg_featureserv`](https://github.com/CrunchyData/pg_featureserv)
is to provide easy and efficient access to [PostGIS](https://postgis.net/) data from web clients.  
To do this it uses the emerging [*OGC API for Features*](https://ogcapi.ogc.org/features/)
(OAPIF) REST protocol, which is a natural fit for systems which need to query and communicate spatial data.
The [core OAPIF](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) specification
provides a basic framework for querying spatial datasets, but it has only limited capability 
to express filtering subsets of spatial tables.  In particular, it only allows filtering on single attribute values,
and it only supports spatial filtering via the `bbox` parameter (in PostGIS terms, this is equivalent to using the `&&` operator on a `box2d`).

Of course, PostGIS and PostgresQL provide much more powerful filtering capabilities. 
It would be very nice to be able to access them via `pg_featureserv`.
Luckily, the OGC has defined the Common Query Language (CQL) which (as the name suggests) is a close match to SQL filtering capabilities.
This is being issued under the OGC API umbrella as CQL2 (currently in draft).
Recently we added support for most of CQL2 to pg_featureserv.
This blog post will describe the power it now provides.

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


 
)
