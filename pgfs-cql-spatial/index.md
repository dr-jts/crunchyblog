# CQL Spatial filters in pg_featureserv

In a [previous post](https://blog.crunchydata.com/blog/cql-filtering-in-pg_featureserv) 
we announced support for **CQL filters** in [`pg_featureserv`](https://github.com/CrunchyData/pg_featureserv).
This capability provides better access to the power of PostgreSQL.
CQL (Common Query Language) is part of the Open Geospatial Consortium's (OGC)
[OGC API](https://ogcapi.ogc.org/#standards) suite of standards.
Naturally, CQL provides the ability to filter geospatial data via spatial filters.
Of course, we implemented this to ensure that `pg_featureserv` is able to take full advantage of 
the spatial capabilities of PostGIS.

The companion project [`pg_tileserv`](https://github.com/CrunchyData/pg_tileserv) also supports CQL, 
and spatial filtering works there as well.

## CQL Spatial Filters

Spatial filtering in CQL involves using **spatial predicates** to test a condition on the geometry property of features.
Spatial predicates include the familiar OGC Simple Features predicates for spatial relationships:

* `INTERSECTS` - tests whether two geometries intersect
* `DISJOINT` - tests whether two geometries have no points in common
* `CONTAINS` - tests whether a geometry contains another
* `WITHIN` - tests whether a geometry is within another
* `EQUALS` - tests whether two geometries are topologically equal
* `CROSSES` - tests whether the geometries cross
* `OVERLAPS` - tests whether the geometries overlap
* `TOUCHES` - tests whether the geometries touch

`pg_featureserv` also implements the **distance predicate** `DWITHIN`.

The conditions are typically used to compare the feature geometry property against a geometry value. 
Geometry values are expressed in [Well-Known Text](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) (WKT):

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
By default, the coordinate reference system (CRS) of geometry values is geodetic (lon/lat).  
This can be changed by using the `filter-crs` parameter with the SRID of the CRS being used.
(PostGIS supports a large number of standard SRIDs.)

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
We've loaded this data into a spatial table called `public.geonames` with a geometry column called `geom`.
We can now query this with pg_featureserv, and view query results on the include UI.

For this example we'll query water features on the [San Juan islands](https://en.wikipedia.org/wiki/San_Juan_Islands)
in the state of Washington, USA.
Because there is no GNIS attribute providing region information, we have to use a spatial query.
We used [QGIS](https://www.qgis.org) to create a polygon enclosing the San Juan islands.

![](pgfs-cql-spatial-sanjuan-polygon.png)

We can convert the polygon to WKT and use it in an `INTERSECTS` spatial predicate.
To retrieve only water features (Lakes and Reservoirs) we add the condition `type IN ('LK','RSV')`.
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

CQL filtering will be included in the forthcoming `pg_featureserv` Version 1.3.
But you can try it out now by simply downloading the latest build. 
Let us know what use cases you find for CQL spatial filtering!

