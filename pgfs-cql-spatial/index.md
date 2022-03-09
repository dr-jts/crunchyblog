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

Spatial filtering in CQL consists of using spatial predicates to test a condition on the geometry property of features.
Spatial predicates include the familiar OGC Simple Features predicates for spatial relationships:

* `INTERSECTS` - tests whether two geometries intersect
* `DISJOINT` - tests whether two geometries have no points in common
* `CONTAINS` - tests whether a geometry contains another
* `WITHIN` - tests whether a geometry is within another
* `EQUALS` - tests whether two geometries are topologically equal
* `CROSSES` - tests whether the geometries cross
* `OVERLAPS` - tests whether the geometries overlap
* `TOUCHES` - tests whether the geometries touch

pg_featureserv also provides the distance-based predicate `DWITHIN`.

The conditions are typically used to compare the feature geometry property against a geometry value, 
which is expressed in Well-Known Text (WKT):

```
POINT (1 2)
LINESTRING (0 0, 1 1)
POLYGON ((0 0, 0 9, 9 0, 0 0))
POLYGON ((0 0, 0 9, 9 0, 0 0),(1 1, 1 8, 8 1, 1 1))
MULTIPOINT ((0 0), (0 9))
MULTILINESTRING ((0 0, 1 1),(1 1, 2 2))
MULTIPOLYGON (((1 4, 4 1, 1 1, 1 4)), ((1 9, 4 9, 1 6, 1 9)))
GEOMETRYCOLLECTION(POLYGON ((1 4, 4 1, 1 1, 1 4)), LINESTRING (3 3, 5 5), POINT (1 5))
ENVELOPE (1, 2, 3, 4)
```

For example, these are spatial conditions:
```
INTERSECTS(geom, ENVELOPE(-100, 49, -90, 50) )
CONTAINS(geom, POINT(-100 49) )
DWITHIN(geom, POINT(-100 49), 0.1)
```

## Examples

things intersected by a road?  or in a polygon?
```
http://localhost:9000/collections/public.geonames/items.html?filter=type%20IN%20(%27LK%27,%27RSV%27)%20AND%20INTERSECTS(geom,%20POLYGON%20((-122.722%2048.7054,%20-122.715%2048.6347,%20-122.7641%2048.6046,%20-122.7027%2048.3885,%20-123.213%2048.4536,%20-123.2638%2048.6949,%20-123.0061%2048.7666,%20-122.722%2048.7054))%20)&limit=10000
```
![](pgfs-cql-spatial-sanjuan-lkrsv.png)

Lakes geonames within 100 km of New York ?

filter-crs example

try out geography table and DWITHIN


## Try it out!


CQL is also supported by `pg_tileserv`, and spatial filtering works there as well.

