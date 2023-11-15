# Using PostGIS and pg-svg to create a distance map for US Cities

The first step is to create an outline of the continental USA for use as the map base.  
In this case we have created a polygon offline and included it as data in the query.
You could also source this from a dataset such as Natural Earth, perhaps loaded in a table.
It's best to use a polygon with a reasonable number of vertices, say no more than 1000.
If the source data is larger, you can reduce it using the `ST_SimplifyPreserveTopology` function.

Next is to obtain locations for the cities displayed on the map.  
Here the data is sourced from [Wikipedia](https://en.wikipedia.org/wiki/List_of_United_States_cities_by_population), and provided as in-query data.  
Of course, points could also be sourced from any desired table.

![](us-city-distance.svg)




