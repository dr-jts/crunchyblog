## Using PostGIS and pg_featureserv with QGIS

My colleague Kat Batuigas recently wrote a [blog post](https://blog.crunchydata.com/blog/arcgis-feature-service-to-postgis-the-qgis-way) about using the powerful open-source [QGIS](https://www.qgis.org/en/site/) desktop GIS to import data into [PostGIS](https://postgis.net/) from an ArcGIS Feature Service.  This is a great first step towards moving your geospatial stack onto the performant, open source platform provided by PostGIS.  But there's no need to stop there!  Crunchy Data provides a suite of tools that work natively with PostGIS to expose your spatial data to the web, using industry-standard protocols.  These include:

* [**pg_tileserv**](https://github.com/CrunchyData/pg_tileserv) - a web service to allow visualizing spatial data using the [MVT](https://github.com/mapbox/vector-tile-spec) vector tile format
* [**pg_featureserv**](https://github.com/CrunchyData/pg_featureserv) - a web service to provide access to spatial data using the [*OGC API for Features*](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) protocol

Recent versions of QGIS support using the *OGC API for Features* (previously known as WFS3) as a vector data source.  So it should be able to source data from `pg_featureserv`.  As it happens, there was a recent `pg_featureserv` [issue](https://github.com/CrunchyData/pg_featureserv/issues/63) concerning this requirement.  The submitter was having trouble getting QGIS to connect to PostGIS via `pg_featureserv`.  After a bit of sleuthing we determined that the problem lay with a couple of places where `pg_featureserv` was not quite meeting the `OGC API for Features` specification.  After fixing those we were happy to find that QGIS was able to load `pg_featureserv` datasets perfectly!

Lets's see how it works. 

## Load data into PostGIS

We are using a [Crunchy Bridge](https://www.crunchydata.com/products/crunchy-bridge/) Postgres/PostGIS instance. For demo purposes we'll load a dataset of British Columbia wildfire perimeter polygons (available for download [here](https://catalogue.data.gov.bc.ca/dataset/fire-perimeters-current)).  The data is provided as a shapefile, so we can use the PostGIS [`shp2pgsql`](https://postgis.net/docs/manual-3.1/postgis_usage.html#shp2pgsql_usage) utility to load it into a table.  The data is in the BC-Albers coordinate system, which has SRID = 3005.  We use the `-c` option so that the loader creates a table appropriate for the dataset, and the -I` option to specify the table should have a spatial index created for it (always a good idea!).  Here we do the load in two steps using an intermediate SQL file, or it can be done in a single command by piping the `shp2pgsql` output to `psql`.

```
shp2pgsql -c -D -s 3005 -i -I prot_current_fire_polys.shp bc.wildfire_poly > bc_wf.sql
psql -h p.asdfghjklqwertyuiop12345.db.postgresbridge.com -U postgres < bc_wf.sql
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

The Crunchy spatial services are lightweight native applications (written in Go), so it's easy to install them on a local or hosted platform.  They can be downloaded as applications, a Docker container, or built from source - see the [Installation guide](https://access.crunchydata.com/documentation/pg_featureserv/1.2.0/installation/installing/) for details.

We're running `pg_featureserv` in a local environment.  To connect it to the Bridge Postgres instance, we specify the database connection information in the `config/pg_featureserv.toml` file. The connection string can also be specified in an environment variable - see the [documentation](https://access.crunchydata.com/documentation/pg_featureserv/1.2.0/installation/configuration/) for more information.

```
[Database]
# Database connection
# postgresql://username:password@host/dbname
DbConnection = "postgres://postgres:password@p.asdfghjklqwertyuiop12345.db.postgresbridge.com:5432/postgres"
```

In the `pg_featureserv` Admin UI we can see the data table published as a collection:

![PG_FS Collections](pgfs-home.png)

We can access the the collection metadata via the OGC API for Features `collection` endpoint:

And we can verify that a data query works:

The Admin UI also lets us see the data on a map:

![BC Wildfire polygons Map](pgfs_wildfire_map.png)


## Display pg_featureserv collection as a QGIS layer

In QGIS we can create a layer that displays the data coming from the `pg_featureserv` instance.  This is a live connection - the layer data is requeried each time the view extent changes via zooming or panning. 

To do this, under the **Layer** menu choose **Add Layer > Add WFS Layer...**.  This displays the **Data Source Manager** window at the **WFS/OGC API-Features** tab.  First, we click **New** to define the connection to the `pg_featureserv` service.  The **Connection Details** dialog let's us enter the following information:

* We use `pg_fs` as the name of the connection
* The connection URL is the URL of the service home page
* The **WFS Version** is *OGC API - Features*
* We'll specify the **Max. number of features** as 200, since we know that will let us load the entire dataset  

![QGIS Data Source Manager/Connection](qgis_dataman_connect.png)

When we click **Connect** we see the collections published by the service (in this demo there is only one):

![QGIS Data Source Manager/OAPIF/Collections](qgis_ds_list.png)

Now we can select the `bc.wildfire_poly` collection, and click **Add** to add it as a layer to the QGIS map layout.  The screenshot also shows the result of using the **Identify Features** tool on the map, showing that all attribute data is loaded as well. 

![QGIS Map](qgis_map.png)

Of course, QGIS is able to connect to PostGIS directly, and provides full query and update capability when doing that.  But it can be simpler, more flexible and more secure to expose PostGIS data via a web service.  That way, it can be easily accessed by many different tools which might not be able to or allowed to connect to PostGIS directly.

We're also exploring ways to be able to run `pg_tileserv` and `pg_featureserv` directly in Crunchy Bridge, to provide a turn-key solution for exposing spatial data on the web.  If this sounds interesting to you, get in touch!


