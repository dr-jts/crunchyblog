## Using pg_featureserv with QGIS

My colleague Kat Batuigas recently wrote a [blog post](https://blog.crunchydata.com/blog/arcgis-feature-service-to-postgis-the-qgis-way) about how to use QGIS with the ArcGIS Feature Service to import data into PostGIS.  This is a great first step towards moving your geospatial stack onto the performant, open source platform provided by PostGIS.  But there's no need to stop there!  Crunchy Data provides a suite of tools that work natively with PostGIS to expose your spatial data to the web, using industry-standard protocols.  These include:

* [**pg_tileserv**](https://github.com/CrunchyData/pg_tileserv) - a web service to allow visualizing spatial data using the [MVT](https://github.com/mapbox/vector-tile-spec) vector tile format
* [**pg_featureserv**](https://github.com/CrunchyData/pg_featureserv) - a web service to provide access to spatial data using the [*OGC API for Features*](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) 

QGIS supports using the *OGC API for Features* protocol (previously known as WFS3) as a vector data source.  So it should be able to source data from `pg_featureserv`.

As it happens, there was a recent [issue](https://github.com/CrunchyData/pg_featureserv/issues/63) logged in the `pg_featureserv` GitHub repo concerning this requirement.  The submitter was having trouble getting QGIS to connect to PostGIS via `pg_featureserv`.  After a bit of sleuthing we determined that the problem lay with a couple of places where `pg_featureserv` was not quite meeting the letter of the OGC API for Features specification.  After fixing those we were happy to find that QGIS was able to display `pg_featureserv` datasets perfectly.

Here's how it works:  

