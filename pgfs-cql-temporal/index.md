# Temporal filtering in pg_featureserv with CQL

In previous posts we presented the new **CQL filtering** capability in `pg_featureserv`.
It provides powerful functionality for attribute and spatial querying against data in PostgreSQL and PostGIS.

The other datatype which is often used in queries is **temporal** - dates and timestamps.
PostgreSQL has extensive capabilities for including time-based attributes in queries.
CQL provides a small but functional subset of these.
This final post in the series will show some examples of temporal filtering in pg_featueserv using CQL.

## CQL Temporal filters

Temporal filtering in CQL is enabled via temporal literals and conditions.

**Temporal literal** values may be dates or full timestamps:
```
date
timestamp
```

**Temporal conditions** allow time-valued properties and literals to be compared via the boolean ordering operators
`<`,`>`,`<=`,`>=`,`=`,`<>`, as well as the `BETWEEN operator:
```
TBD
```

Publishing Tropical Storm tracks

We'll demonstrate temporal filters using a dataset with a strong time linkage: tracks of tropical storms (or hurricanes).
This dataset is available from [here](https://hifld-geoplatform.opendata.arcgis.com/datasets/geoplatform::historical-tropical-storm-tracks)...

- Preparation
- 

## Querying by Time

```
http://localhost:9000/collections/public.trop_storm/items.html?filter=time_start between 2005-01-01 and NOW()&limit=100
```

- image

## Querying by Time and Area

Temporal conditions can be combined with other kinds of filters. For instance, we can execute a spatio-temporal query
by using a temporal condition along with a spatial condition.
In this example, we'll query the tropical storms which occurred in the 1990s, and whose track took them over the Florida mainland.


```
http://localhost:9000/collections/public.trop_storm/items.html?filter=time_start between 2005-01-01 and NOW() AND intersects(geom, POLYGON ((-81.4067 30.8422, -79.6862 25.3781, -81.1609 24.7731, -83.9591 30.0292, -85.2258 29.6511, -87.5892 29.9914, -87.5514 31.0123, -81.4067 30.8422)) )&limit=100
```

- image
