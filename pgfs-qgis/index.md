## Using PostGIS and pg_featureserv with QGIS

My colleague Kat Batuigas recently wrote a [blog post](https://blog.crunchydata.com/blog/arcgis-feature-service-to-postgis-the-qgis-way) about how to use the powerful open-source QGIS geospatial GUI to import data into [PostGIS](https://postgis.net/) from an ArcGIS Feature Service.  This is a great first step towards moving your geospatial stack onto the performant, open source platform provided by PostGIS.  But there's no need to stop there!  Crunchy Data provides a suite of tools that work natively with PostGIS to expose your spatial data to the web, using industry-standard protocols.  These include:

* [**pg_tileserv**](https://github.com/CrunchyData/pg_tileserv) - a web service to allow visualizing spatial data using the [MVT](https://github.com/mapbox/vector-tile-spec) vector tile format
* [**pg_featureserv**](https://github.com/CrunchyData/pg_featureserv) - a web service to provide access to spatial data using the [*OGC API for Features*](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) 

QGIS supports using the *OGC API for Features* protocol (previously known as WFS3) as a vector data source.  So it should be able to source data from `pg_featureserv`.  As it happens, there was a recent [issue](https://github.com/CrunchyData/pg_featureserv/issues/63) logged in the `pg_featureserv` GitHub repo concerning this requirement.  The submitter was having trouble getting QGIS to connect to PostGIS via `pg_featureserv`.  After a bit of sleuthing we determined that the problem lay with a couple of places where `pg_featureserv` was not quite meeting the `OGC API for Features` specification.  After fixing those we were happy to find that QGIS was able to load `pg_featureserv` datasets perfectly!

Lets's see how it works. 

## Load data into PostGIS

We are using a [Crunchy Bridge](https://www.crunchydata.com/products/crunchy-bridge/) Postgres/PostGIS instance. For demo purposes we'll load a dataset of British Columbia wildfire perimeter polygons (available for download [here](https://catalogue.data.gov.bc.ca/dataset/fire-perimeters-current)).  The data is provided as a shapefile, so we can use the PostGIS `shp2pgsql` utility to load it into a table.  I like do this in two steps using an intermediate SQL file, or it can be done in a single command by piping the `shp2pgsql` output to `psql`.  The data is in the BC-Albers coordinate system, which has SRID = 3005.

```
shp2pgsql -c -D -s 3005 -i -I prot_current_fire_polys.shp bc.wildfire_poly > bc_wf.sql
psql -h <db url> -U postgres < bc_wf.sql
```

Using `psql` we can connect to the database and verify that the table has been created:
```
postgres=# \d bc.wildfire_poly
                                            Table "bc.wildfire_poly"
   Column   |            Type             | Collation | Nullable |                    Default                    
------------+-----------------------------+-----------+----------+-----------------------------------------------
 gid        | integer                     |           | not null | nextval('bc.wildfire_poly_gid_seq'::regclass)
 objectid   | double precision            |           |          | 
 fire_year  | integer                     |           |          | 
 fire_numbe | character varying(6)        |           |          | 
 version_nu | double precision            |           |          | 
 fire_size_ | numeric                     |           |          | 
 source     | character varying(50)       |           |          | 
 track_date | date                        |           |          | 
 load_date  | date                        |           |          | 
 feature_co | character varying(10)       |           |          | 
 fire_stat  | character varying(30)       |           |          | 
 fire_nt_id | integer                     |           |          | 
 fire_nt_nm | character varying(50)       |           |          | 
 fire_nt_lk | character varying(254)      |           |          | 
 geom       | geometry(MultiPolygon,3005) |           |          | 
Indexes:
    "wildfire_poly_pkey" PRIMARY KEY, btree (gid)
    "wildfire_poly_geom_idx" gist (geom)
 ```

## Serve PostGIS data with pg_featureserv

We're running `pg_featureserv` in a separate environment, pointing to the Bridge Postgres instance.

In the `pg_featureserv` Admin UI we can see the data table published as a collection:


We can also access the the collection metadata via the OGC API for Features `collection` endpoint:

And we can verify that a data query works:

We can also see the data on a map in the Admin UI:


## Load pg_featureserv collections in QGIS

In QGIS we can create a connection to the `pg_featureserv` instance.  This is done in the **Data Source Manager** (found under the **Layer** menu).  The **WFS/OGC API-Features** tab displays the following dialog:

![QGIS Data Source Manager/Connection](qgis_dataman_connect.png)

We fill in the following fields:

When we click **Connect** we see the collections published by the service (in this case there is only one):

![QGIS Data Source Manager/OAPIF/Collections](qgis_ds_list.png)

Now we can select the bc.wildfire_poly collection, and click **Add** to add it as a layer to the QGIS map layout:

![QGIS Map](qgis_map.png)

This is a live connection; the data is requeried as the map extent is changed by zooming or panning.





 

-   
- QGIS makes it simple to add the ArcGIS service as a layer. In QGIS, you should see "ArcGIS Feature Service" in the Browser panel. 
- Right-click that and we see the option to add a new connection. That opens up to the following dialog:   

