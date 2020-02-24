# Crunchy Spatial: Feature Serving

In addition to viewing spatial data from PostGIS on a web map using `pg_tileserv`, 
it's often useful to be able to access the spatial data directly.
This supports use cases such as:

* showing or using feature data under a user click or in an area of interest
* querying data features or fields using attribute filters
* downloading features for use in the web application (e.g. for tabular or map display)
* downloading features for use in other applications

For the past twenty years this need has been met by the OCG Web Feature Service (WFS) protocol.
Recently this facility has been revamped to align it with modern best practices
for web services.
The [OGC API for Features](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) standard 
provides a RESTful API, JSON and GeoJSON as the primary data formats,
and [OpenAPI[(https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md) support.

