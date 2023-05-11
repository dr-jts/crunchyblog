# Creating Maps and Images with PostGIS and `pg-svg`

One of the most powerful and informative things to do with geospatial data is to visualize it on a map.  There are many ways of doing this.  Data can be rendering to a raster image using a map server like GeoServer or MapServer; the raw data can be shipped to a Web browser and rendered using a library such as OpenLayers or Leaflet; or a client GIS application such as QGIS can connect to the database and create richly-styled maps from the data.

This post is about a way to generate maps entirely within the database, with no external application required.  This is done by using the SVG (Scabable Vector Graphics) format, which is a graphical format that is widely supported by web browsers and other tools.  This is facilitated by a PL/pgSQL library called pg-svg.  The library provides a set of high-level functions which make it easy to convert PostGIS data into styled SVG documents.

- discuss how SVG requires shapes, scaling, styling and optionally identifiers and CSS

To demonstrate using pg-svg, we'll create a map 

