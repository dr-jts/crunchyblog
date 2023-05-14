# Creating Maps and Images with PostGIS and `pg-svg`

One of the most powerful and informative things to do with geospatial data is to visualize it on a map.  There are many ways of doing this.  Data can be rendering to a raster image using a map server like GeoServer or MapServer; the raw data can be shipped to a Web browser and rendered using a library such as OpenLayers or Leaflet; or a client GIS application such as QGIS can connect to the database and create richly-styled maps from the data.  

This post is about a way to generate maps entirely within the database, with no external application required.  This is done by using the SVG (Scabable Vector Graphics) format, which is a graphical format that is widely supported by web browsers and other tools.  This is facilitated by a PL/pgSQL library called pg-svg.  The library provides a set of high-level functions which make it easy to convert PostGIS data into styled SVG documents.

PostGIS has the function [`ST_AsSVG`](https://postgis.net/docs/manual-3.3/ST_AsSVG.html), but an attempt to use it will quickly make you realize that it is only one part (albeit an essential one) of generating an SVG image.  `ST_AsSVG` produces the [**path data**](https://svgwg.org/svg2-draft/paths.html#PathData) that specifies the outline of a shape to be rendered in SVG.  However, much more is required:

* the path data must be encoded as the `d` attribute of a `path` element
* the element needs to be styled
* Polygons with holes require multiple path elements
* Multigeometries are represented as groups of paths
* Simple shapes such as points and lines require different elements
* The SVG image needs to be provided as an SVG `<svg>` element
* the extent of the rendered data must be specified, along with (optionally) the size of the rendered image

- discuss how SVG requires shapes, scaling, styling and optionally identifiers and CSS

To demonstrate using pg-svg, we'll create a map 

For another option for database-generated maps, see this PostGIS Day 2022 [presentation](https://www.youtube.com/watch?v=5Zg8j9X2f-Y) showing how to define and render bitmap images. 

