# Spatial filters with CQL in pg_featureserv

In a [previous post](https://blog.crunchydata.com/blog/cql-filtering-in-pg_featureserv) 
we announced support for **CQL filtering** in [`pg_featureserv`](https://github.com/CrunchyData/pg_featureserv).
This capability allows much better access to the power of PostgreSQL.
CQL (Common Query Language) is part of the Open Geospatial Consortium's (OGC)
[OGC API](https://ogcapi.ogc.org/#standards) suite of standards.
As such, naturally CQL provides the ability to filter geospatial data via spatial filters as well.
Of course, we implemented this to ensure that `pg_featureserv` is able to take full advantage of 
the spatial capabilities of PostGIS.

## CQL Spatial Filters


geometry WKT literals

spatial predicates

distance predicates

## Examples

things intersected by a road?  or in a polygon?

Lakes geonames within 100 km of New York ?

fiiter-crs example

try out geography table and DWITHIN


## Try it out!


CQL is also supported by `pg_tileserv`, and spatial filtering works there as well.

