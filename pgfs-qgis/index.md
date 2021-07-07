## Using pg_featureserv with QGIS

My colleague Kat Batuigas recently wrote a blog post about how to use QGIS with the ArcGIS Feature Service to import data into PostGIS.  This is a great first step towards moving your geospatial stack onto the performant, open source platform provided by PostGIS.  But there's no need to stop there!  Crunchy Data provides a suite of tools that work natively with PostGIS to expose your spatial data to the web, using industry-standard protocols.  These include:

* pg_tileserv - a web service to allow visualizing spatial data using the MVT vector tile format
* pg_featureserv - a web service to provide access to spatial data using the OGC API for Features 

QGIS can also use pg_featureserv as a vector data source.

As it happens, there was a recent issue logged in the pg_featureserv GitHub repo concerning this requirement.  The submitter was having trouble getting the QGIS to connect to PostGIS via pg_featureserv.  After a bit of sleuthing we determined that the problem lay with a couple of places where pg_featureserv was not quite meeting the letter of the OGC API for Features specification.  After fixing those we were happy to find that QGIS was able to display pg_featureserv datasets perfectly.

Here's how it works:  

