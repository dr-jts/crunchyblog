## Using pg_featureserv with QGIS

My colleague Kat Batuigas recently wrote a [blog post](https://blog.crunchydata.com/blog/arcgis-feature-service-to-postgis-the-qgis-way) about how to use QGIS with the ArcGIS Feature Service to import data into PostGIS.  This is a great first step towards moving your geospatial stack onto the performant, open source platform provided by PostGIS.  But there's no need to stop there!  Crunchy Data provides a suite of tools that work natively with PostGIS to expose your spatial data to the web, using industry-standard protocols.  These include:

* [**pg_tileserv**](https://github.com/CrunchyData/pg_tileserv) - a web service to allow visualizing spatial data using the [MVT](https://github.com/mapbox/vector-tile-spec) vector tile format
* [**pg_featureserv**](https://github.com/CrunchyData/pg_featureserv) - a web service to provide access to spatial data using the [*OGC API for Features*](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) 

QGIS supports using the *OGC API for Features* protocol (previously known as WFS3) as a vector data source.  So it should be able to source data from `pg_featureserv`.  As it happens, there was a recent [issue](https://github.com/CrunchyData/pg_featureserv/issues/63) logged in the `pg_featureserv` GitHub repo concerning this requirement.  The submitter was having trouble getting QGIS to connect to PostGIS via `pg_featureserv`.  After a bit of sleuthing we determined that the problem lay with a couple of places where `pg_featureserv` was not quite meeting the `OGC API for Features` specification.  After fixing those we were happy to find that QGIS was able to load `pg_featureserv` datasets perfectly!

Lets's see how it works. 

## Serve PostGIS data with pg_featureserv

For demo purposes, we are using a [Crunchy Bridge](https://www.crunchydata.com/products/crunchy-bridge/) Postgres/PostGIS instance, loaded with a dataset of British Columbia wildfire extent polygons (available here).  We're running `pg_featureserv` in a separate environment, pointing to the Bridge Postgres instance.

In the `pg_featureserv` Admin UI we can see the dataset published as a collection:


We can also access the the collection metadata via the OGC API for Features `collection` endpoint:

And we can verify that a data query works:

We can also see the data on a map in the Admin UI:


## Load pg_featureserv collections in QGIS

In QGIS we can create a connection to the `pg_featureserv` instance.  This is done in the **Data Source Manager** (available from the **Layer** menu).  The **WFS/OGC API-Features** tab displays the following dialog:




 

-   
- QGIS makes it simple to add the ArcGIS service as a layer. In QGIS, you should see "ArcGIS Feature Service" in the Browser panel. 
- Right-click that and we see the option to add a new connection. That opens up to the following dialog:   

