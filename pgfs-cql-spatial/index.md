# Spatial filters with CQL in pg_featureserv

In a previous post we announced support for CQL filtering in pg_featureserv.
This is an capability that allows much better access to the power of PostgreSQL.
CQL (Common Query Language) is part of the Open Geospatial Consortium (OGC) emerging
OGC API for Features standard.
As such it's no surprise that CQL provides the ability to filter geospatial data via spatial filters as well.
Of course, we implemented this to ensure that pg_featureserv is able to take full advantage of 
the spatial capabilities of PostGIS.

## CQL Spatial Filters

geometry WKT literals

spatial predicates

distance predicates

## Examples

things intersected by a road?  or in a polygon?

Lakes geonames within 100 km of New York ?



## Try it out!

