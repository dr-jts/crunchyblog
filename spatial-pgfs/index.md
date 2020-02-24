# Crunchy Spatial: Feature Serving

In addition to viewing spatial data from PostGIS on a web map using `pg_tileserv`, 
it's often useful to be able to access the spatial data directly.
This supports use cases such as:

* showing or using feature data under a user click or in an area of interest
* querying data features or fields using attribute filters
* downloading features for use in the web application (e.g. for tabular or map display)
* downloading features for use in other applications

For the past twenty years this need has been met by the OCG Web Feature Service (WFS) standard.
Recently this standard has been revamped to align it with modern best practices
for web services.
The [OGC API for Features](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html) standard 
provides a RESTful API, JSON and GeoJSON as the primary data formats,
and [OpenAPI](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md) support.
We realized that this new specification was a perfect fit for the 
microservice architecture of Crunchy Spatial, and had the added benefit
of being easily extensible to expose the rich spatial capabilities of PostGIS.
This is the origin story of `pg_featureserv`.

`pg_featureserv` has the following features:

* Written in [Go](https://golang.org/) to allow for simple deployment of binaries with no complex dependency chains.  Go also provides a very effective platform for building simple services with a minimum of development effort and software defects.
* Ready-to-run defaults so that basic deployment just requires setting a database configuration string and running the program.
* Simple web user interface to explore the published feature collections, and view the features on maps.
* Support for most of the OGC Features API, including `limit` and `offset` paging, `bbox` filtering, and `properties` response shaping
* Additional query parameters to expose the power of PostGIS, including `orderBy` and `transform`.
* Function-based data sources, so you can generate feature collections from spatial functions


