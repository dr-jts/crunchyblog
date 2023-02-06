# Temporal filtering in pg_featureserv with CQL

In a [previous post](https://blog.crunchydata.com/blog/cql-filtering-in-pg_featureserv) we announced the **CQL filtering** capability in [`pg_featureserv`](https://github.com/CrunchyData/pg_featureserv).
It provides powerful functionality for attribute and [spatial](https://blog.crunchydata.com/blog/spatial-filters-in-pg_featureserv-with-cql) querying of data in PostgreSQL and PostGIS.

Another important datatype which is often present in datasets is **temporal** - i.e. data containing columns which are dates or timestamps.
PostgreSQL has extensive capabilities for specifying queries against time-valued attributes.
CQL provides acccess to a small but useful subset of these.
This post in the CQL series will show some examples of temporal filtering in `pg_featureserv` using CQL.

## CQL Temporal filters

Temporal filtering in CQL is provided using **temporal literals** and **conditions**.

**Temporal literal** values may be **dates** or **timestamps**:
```
2001-01-01
2010-04-23T01:23:45
```

> ***Note: - the temporal literal syntax is based on an early version of the OGC API [Filter and CQL standard](https://portal.ogc.org/files/96288).  
> The current [draft CQL standard](https://docs.ogc.org/DRAFTS/21-065.html) has a different syntax: `DATE('1969-07-20')` and `TIMESTAMP('1969-07-20T20:17:40Z')`.  
> A subsequent version of `pg_featureserv` will support this syntax as well.***
 
**Temporal conditions** allow time-valued properties and literals to be compared via the boolean ordering operators
`<`,`>`,`<=`,`>=`,`=`,`<>`, and the `BETWEEN..AND` operator:
```
start_date >= 2001-01-01
event_time BETWEEN 2010-04-22T06:00 AND 2010-04-23T12:00
```



## Publishing Tropical Storm tracks

We'll demonstrate temporal filters using a dataset with a strong time linkage: tracks of tropical storms (or hurricanes).
This dataset is available from [here](https://hifld-geoplatform.opendata.arcgis.com/datasets/geoplatform::historical-tropical-storm-tracks)...

- Preparation
- 

![](pgfs-cql-temporal-trop-storm.png)

## Querying by Time

That's obviously too many tracks to visualize conveniently.  A natural way to subset the data is by querying over a time range.


```
http://localhost:9000/collections/public.trop_storm/items.html?filter=time_start BETWEEN 2005-01-01 AND 2009-12-31&limit=100
```

- image

## Querying by Time and Space

Temporal conditions can be combined with other kinds of filters. For instance, we can execute a spatio-temporal query
by using a temporal condition along with a spatial condition.
In this example, we query the storms which occurred in 2005 and after in Florida.
The temporal condition is expressed as `time_start > 2005-01-01`.
The spatial condition uses the `INTERSECTS` predicate to test whether the line geometry of a hurricane track intersects a polygon representing the (simplified) coastline of Florida.  The polygon is provided as a geometry literal using WKT.
(For more information about spatial filtering with CQL in `pg_featureserv` see this [blog post](https://www.crunchydata.com/blog/spatial-filters-in-pg_featureserv-with-cql).)

```
POLYGON ((-81.4067 30.8422, -79.6862 25.3781, -81.1609 24.7731, -83.9591 30.0292, -85.2258 29.6511, -87.5892 29.9914, -87.5514 31.0123, -81.4067 30.8422))
```

![](pgfs-cql-temporal-poly-fla.png)

Putting these conditions together, the request to retrieve the desired tracks from `pg_featureserv` is:

```
http://localhost:9000/collections/public.trop_storm/items.html?filter=time_start > 2005-01-01 AND INTERSECTS(geom, POLYGON ((-81.4067 30.8422, -79.6862 25.3781, -81.1609 24.7731, -83.9591 30.0292, -85.2258 29.6511, -87.5892 29.9914, -87.5514 31.0123, -81.4067 30.8422)) )&limit=100
```

- image of result tracks
