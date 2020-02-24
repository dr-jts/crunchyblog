# Crunchy Spatial: Feature Serving

In addition to viewing spatial data from [PostGIS](https://postgis.net/) on a web map using (`pg_tileserv`), 
it's often useful to be able to access the spatial data directly.
This supports use cases such as:

* showing or using feature data under a user click or in an area of interest
* querying data features or fields using attribute filters
* downloading features for use in the web application (e.g. for tabular or map display)
* downloading features for use in other applications

For the past twenty years this need has been met by the **OCG Web Feature Service (WFS)** standard.
Recently this standard has been completely rewritten to align it with modern best practices
for web services.
The new **[OGC API for Features](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html)** standard 
now provides a RESTful API with HATEOAS links, JSON and GeoJSON as the primary data formats,
and [OpenAPI](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md) support.

We realized that this new specification was a perfect fit for the 
microservice architecture of (Crunchy Spatial).  
It has the additional benefit
of being easily extensible, which allows us to expose more of the rich spatial capabilities of PostGIS.
This is the origin story of `pg_featureserv`.

`pg_featureserv` has the following features:

* Written in [Go](https://golang.org/) to allow for simple deployment of binaries with no complex dependency chains.  Go provides a very effective platform for building simple services with a minimum of development effort and software defects.
* Ready-to-run defaults so that basic deployment just requires setting a database configuration string and running the program.
* Simple web user interface to explore the published feature collections, and view the feature data on maps.
* Support for most of the OGC Features API, including `limit` and `offset` paging, `bbox` filtering, and `properties` response shaping
* Additional query parameters to expose the power of PostGIS, including `orderBy` and `transform`.
* Function-based data sources, so you can generate feature collections from spatial functions

`pg_featureserv` is easy to try out.  Here's how to run it yourself.  (Most of the steps just involve getting some spatial data in a PostGIS database; if you already have a database, just skip down to step 3 and input your own database connection information).

1. Make a database, and enable PostGIS.
   

    ```sh
    createdb postgisftw
    psql -d postgisftw -c 'create extension postgis'
    ```

2. Download some spatial data, and load it into PostGIS.

    ```sh
    curl -o https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/50m/cultural/ne_50m_admin_0_countries.zip
    unzip ne_50m_admin_0_countries.zip
    shp2pgsql -S 4326 -D -I ne_50m_admin_0_countries | psql -d postgisftw
    ```

3. Download and unzip the pg_tileserv binary for your platform

    * [Linux](https://postgisftw.s3.amazonaws.com/pg_featureserv_latest_linux.zip)
    * [Windows](https://postgisftw.s3.amazonaws.com/pg_featureserv_latest_windows.zip)
    * [MacOS](https://postgisftw.s3.amazonaws.com/pg_featureserv_latest_osx.zip)

4. Set the `DATABASE_URL` environment variable to point to your database, and start the service.

    ```sh
    export DATABASE_URL=postgresql://postgres@localhost:5432/postgisftw
    ./pg_featureserv
    ```

5. Point your browser to the service web interface URL.

    * http://localhost:9000

6. Explore the data!

The service provides a JSON-based API for programatic service discovery and data access
following the OGC API - Features standard.
The JSON API landing page is:

* http://localhost:9000/index.json

The feature collections can be listed using:

* http://localhost:9000/collections.json
    
The metadata for the Natural Earth countries table is at:

* http://localhost:9000/collections/ne_50m_admin_0_countries.json
    
The simplest query for country features is:

* http://localhost:9000/collections/ne_50m_admin_0_countries/items.json
   
Some more realistic queries are:

   * Query the 20 smallest countries: http://localhost:9000/collections/ne_50m_admin_0_countries/items.json
   * Query the country at a given point: 

A human-viewable web user interface is also provided, allowing easy browsing of feature collections and data. 
The top-level page is at:

   * http://localhost:9000/index.html
   
*image: top-level page?*
   
The user interface allows listing the feature collections and functions exposed by the service out of the database.
The metadata for each collection and function can be displayed.
A simple web map interface allows you to view the results of queries for features. 
It provides some simple controls to allow setting query parameters and function arguments.

*image: web map showing NE countries*






