# Using PostGIS and pg-svg to create a Distance map

As a declarative language with powerful set-processing capabilities, SQL is an ideal language for generating graphical images
(which often involve many nearly-similar elements).
When combined with the geospatial processing capabilities of PostGIS and the SVG output enabled by pg_svg, 
it makes for an easy way to generate interesting map visualizations.

For example, here's how to create a map showing distances from US cities, with some interesting visual effects.

![](us-city-distance.svg)

The first step is to create an outline of the continental USA for use as the map base.  
In this case we have created a polygon offline and included it as data in the query.
You could also source this from a dataset such as Natural Earth, perhaps loaded in a table.
It's best to use a polygon with a reasonably small number of vertices (say about 1000).
If the source data has more vertices, you can reduce it using the `ST_SimplifyPreserveTopology` function.

Next is to obtain locations for the cities displayed on the map.  
Here the data is sourced from [Wikipedia](https://en.wikipedia.org/wiki/List_of_United_States_cities_by_population), and provided as in-query data.  
Of course, points could also be sourced from any desired table.






