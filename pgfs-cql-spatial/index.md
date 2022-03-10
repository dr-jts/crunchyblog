# Spatial filters with CQL in pg_featureserv

In a [previous post](https://blog.crunchydata.com/blog/cql-filtering-in-pg_featureserv) 
we announced support for **CQL filtering** in [`pg_featureserv`](https://github.com/CrunchyData/pg_featureserv).
This capability provides better access to the power of PostgreSQL.
CQL (Common Query Language) is part of the Open Geospatial Consortium's (OGC)
[OGC API](https://ogcapi.ogc.org/#standards) suite of standards.
Naturally, CQL provides the ability to filter geospatial data via spatial filters as well.
Of course, we implemented this to ensure that `pg_featureserv` is able to take full advantage of 
the spatial capabilities of PostGIS.

## CQL Spatial Filters

Spatial filtering in CQL involves using spatial predicates to test a condition on the geometry property of features.
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

The conditions are typically used to compare the feature geometry property against a geometry value. 
Geometry values are expressed in Well-Known Text (WKT):

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
By default, the CRS of geometry values is geodetic (lon/lat).  
This can be changed by using the `filter-crs` parameter with the SRID of the CRS in use.

Here are some examples of spatial filter conditions:
```
INTERSECTS(geom, ENVELOPE(-100, 49, -90, 50) )
CONTAINS(geom, POINT(-100 49) )
DWITHIN(geom, POINT(-100 49), 0.1)
```
Of course, these can be combined with attribute conditions as well.

## Example of a filter using INTERSECTS

For these examples we'll use the U.S. [Geographic Names Information System](https://en.wikipedia.org/wiki/Geographic_Names_Information_System) (GNIS) dataset.
It contains more than 2 million named points for geographical features.
We've loaded this data into a spatial table called `public.geonames`.
We can now query this with pg_featureserv, and view query results on the include UI.

For this example we'll query water features on the San Juan islands.
Because there is no attribute providing that location information, we have to use a spatial query.
We used QGIS to create a polygon enclosing the San Juan islands.
We can convert the polygon to Well-Known Text (WKT) and use it in an INTERSECTS spatial predicate.
To retrieve only water features (Lakes and Reservoirs) we add a condition `type IN ('LK','RSV')`.
The query URL is:
```
http://localhost:9000/collections/public.geonames/items.html?filter=type IN ('LK','RSV') AND INTERSECTS(geom,POLYGON ((-122.722 48.7054, -122.715 48.6347, -122.7641 48.6046, -122.7027 48.3885, -123.213 48.4536, -123.2638 48.6949, -123.0061 48.7666, -122.722 48.7054)))
```
The result of the query is a dataset containing 33 GNIS points:

![](pgfs-cql-spatial-sanjuan-lkrsv.png)

## Example of a filter using DWITHIN

- create geography table, load GNIS (CREATE TABLE, INSERT, CREATE INDEX)
- discuss why geography better for DWITHIN
- [geography type](https://blog.crunchydata.com/blog/postgis-and-the-geography-type)
- 
Mountains within 100 km of Seattle
```
http://localhost:9000/collections/us.geonames_geo/items.html?filter=type = 'MT' AND DWITHIN(geom,Point(-122.34 47.6),100000)&limit=1000
```
![](pgfs-cql-spatial-dwithin-mt.png)

filter-crs example?


## Try it out!


CQL is also supported by `pg_tileserv`, and spatial filtering works there as well.

