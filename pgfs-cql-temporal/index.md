# Temporal filtering in pg_featureserv with CQL


## Querying by Time

```
http://localhost:9000/collections/public.trop_storm/items.html?filter=time_start%20between%202005-01-01%20and%20NOW()&limit=100
```

## Querying by Time and Area

```
http://localhost:9000/collections/public.trop_storm/items.html?filter=time_start%20between%202005-01-01%20and%20NOW() AND intersects(geom, POLYGON ((-81.4067 30.8422, -79.6862 25.3781, -81.1609 24.7731, -83.9591 30.0292, -85.2258 29.6511, -87.5892 29.9914, -87.5514 31.0123, -81.4067 30.8422)) )&limit=100
```
