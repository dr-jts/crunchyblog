# Temporal filtering in pg_featureserv with CQL

In a [previous post](https://blog.crunchydata.com/blog/cql-filtering-in-pg_featureserv) we announced the new **CQL filtering** capability in [`pg_featureserv`](https://github.com/CrunchyData/pg_featureserv).
It provides powerful functionality for attribute and [spatial](https://blog.crunchydata.com/blog/spatial-filters-in-pg_featureserv-with-cql) querying of data in PostgreSQL and PostGIS.

Another important datatype which is often present in datasets is **temporal** - i.e. data containing columns which are dates or timestamps.
PostgreSQL has extensive capabilities for specifying queries against time-valued attributes.
CQL provides a small but useful subset of these.
This final post in the series will show some examples of temporal filtering in `pg_featureserv` using CQL.

## CQL Temporal filters

Temporal filtering in CQL is provided using **temporal literals** and **conditions**.

**Temporal literal** values may be dates or timestamps:
```
2001-01-01
2010-04-23T01:23:45
```

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

```
http://localhost:9000/collections/public.trop_storm/items.html?filter=time_start BETWEEN 2005-01-01 AND NOW()&limit=100
```

- image

## Querying by Time and Space

Temporal conditions can be combined with other kinds of filters. For instance, we can execute a spatio-temporal query
by using a temporal condition along with a spatial condition.
In this example, we query the storms which occurred in Florida in the 1990s.
The temporal condition is expressed as `time_start BETWEEN 1990-01-01 AND 1999-12-31`.
The spatial condition uses the `INTERSECTS` predicate to test whether the line geometry of a track intersects a polygon representing the (simplified) coastline of Florida.

![](pgfs-cql-temporal-poly-fla.png)

The `pg_featureserv` request to retrieve the tracks meeting the conditions is:
```
http://localhost:9000/collections/public.trop_storm/items.html?filter=time_start BETWEEN 2005-01-01 AND NOW() AND INTERSECTS(geom, POLYGON ((-81.4067 30.8422, -79.6862 25.3781, -81.1609 24.7731, -83.9591 30.0292, -85.2258 29.6511, -87.5892 29.9914, -87.5514 31.0123, -81.4067 30.8422)) )&limit=100
```
```
http://localhost:9000/collections/public.trop_storm/items.html?filter=time_start BETWEEN 1990-01-01 AND 1999-12-31 AND INTERSECTS(geom, POLYGON ((-81.4067 30.8422, -79.6862 25.3781, -81.1609 24.7731, -83.9591 30.0292, -85.2258 29.6511, -87.5892 29.9914, -87.5514 31.0123, -81.4067 30.8422)) )&limit=100
```

- image of result tracks
