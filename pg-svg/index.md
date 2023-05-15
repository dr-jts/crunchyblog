# Creating Maps with PostGIS and `pg-svg`

One of the most informative things to do with geospatial data is to visualize it on a map.  *Comment about how PostGIS data is generally just data* There are many ways of doing this.  Data can be rendering to a raster image using a map server like GeoServer or MapServer; the raw data can be shipped to a Web browser and rendered using a library such as OpenLayers or Leaflet; or a client GIS application such as QGIS can connect to the database and create richly-styled maps from the data.  

- For another option for database-generated maps, see this PostGIS Day 2022 [presentation](https://www.youtube.com/watch?v=5Zg8j9X2f-Y) showing how to define and render bitmap images. 

This post presents a way to generate maps entirely within the database, with no external application required.  This is done by using the SVG (Scalable Vector Graphics) format, which is widely supported by web browsers and other tools.  Generating SVG can be complex, but a PL/pgSQL library called `pg-svg` makes it easy.  The library provides a set of high-level functions which make it easy to convert PostGIS data into styled SVG documents.

- discuss how SVG requires shapes, scaling, styling and optionally identifiers and CSS

## Introducing `pg-svg`

PostGIS has had the function [`ST_AsSVG`](https://postgis.net/docs/manual-3.3/ST_AsSVG.html) for years.  But attempting to use it will quickly reveal that it is only a partial solution for generating an SVG image.  `ST_AsSVG` produces the [**path data**](https://svgwg.org/svg2-draft/paths.html#PathData) that specifies the outline of a shape to be rendered in SVG.  However, much more is required:

* the path data must be encoded as the `d` attribute of a `<path>` element
* the element needs to be styled
* Polygons with holes require multiple path elements
* Multigeometries are represented as groups of paths
* Simple shapes such as points require different elements
* The SVG image needs to be provided as an SVG `<svg>` element
* the extent of the rendered data must be specified, along with (optionally) the size of the rendered image

Doing this manually requires detailed knowledge of the SVG format, and is complex and error-prone.  
But the `pg-svg` library can do all this in a single SQL query with a simple API.

## Example

The easiest way to understand `pg-svg` is to see an example.  We'll create a map of the United States showing the highest point in each state.
To show the power of SVG and `pg-svg` for generating and styling shapes the map has the following features:

* All 50 states are shown, with Alaska and Hawaii transformed to better fit the map
* States are labelled, and filled with a gradient
* High points are shown at their location by triangles whose color and size indicate the height of the high point.
* Tooltips provide more information about states and highpoints.   

The resulting map looks like this (to see tooltips open the [raw image](https://raw.githubusercontent.com/dr-jts/crunchyblog/master/pg-svg/us-highpt.svg)):

![](us-highpt.svg)

The SQL query to generate the map is [here](https://github.com/dr-jts/pg_svg/blob/master/demo/map/us-highpt-svg.sql).  It can be downloaded and run using `psql`:
```
psql -A -t -o us-highpt.svg  < us-highpt-svg.sql
```
The SVG output `us-highpt.svg` can be view in any web browser.

## How it Works

Let's break the query down to see how the data is prepared and then rendered to SVG.  A dataset of US states in geodetic coordinate system (WGS84, SRID = 4326) is required.  We used the Natural Earth states and provinces data available [here](https://www.naturalearthdata.com/downloads/10m-cultural-vectors/10m-admin-1-states-provinces/).  It is loaded into a table `ne.admin_1_state_prov` with the following command:
```
shp2pgsql -c -D -s 4326 -i -I ne_10m_admin_1_states_provinces.shp ne.admin_1_state_prov | psql
```

The query uses the SQL `WITH` construct to organize the process into simple, modular steps.  We'll describe each one in turn.

### Select US state features

First, the US state features are selected from the Natural Earth boundaries table `ne.admin_1_state_prov`.
```
us_state AS (SELECT name, abbrev, postal, geom 
  FROM ne.admin_1_state_prov
  WHERE adm0_a3 = 'USA')
```

### Make a US state map
Next, the geometry for states Alaska and Hawaii is translated and scaled to make the map more compact.  
This is done using PostGIS [affine transformation functions](https://postgis.net/docs/manual-3.3/reference.html#Affine_Transformation).
The scaling is done around the location of the state high point, 
to make it easy to apply the same transformation to the high point feature.
```
,us_map AS (SELECT name, abbrev, postal, 
    -- transform AK and HI to make them fit map
    CASE WHEN name = 'Alaska' THEN 
      ST_Translate(ST_Scale(
        ST_Intersection( ST_GeometryN(geom,1), 'SRID=4326;POLYGON ((-141 80, -141 50, -170 50, -170 80, -141 80))'),
        'POINT(0.5 0.75)', 'POINT(-151.007222 63.069444)'::geometry), 18, -17)
    WHEN name = 'Hawaii' THEN 
      ST_Translate(ST_Scale(
        ST_Intersection(geom, 'SRID=4326;POLYGON ((-161 23, -154 23, -154 18, -161 18, -161 23))'), 
        'POINT(3 3)', 'POINT(-155.468333 19.821028)'::geometry), 32, 10)
    ELSE geom END AS geom
  FROM us_state)
```
### High Points of US states
Data for the highest point in each state is provided as an inline table of values:
```
,high_pt(name, state, hgt_m, hgt_ft, lon, lat) AS (VALUES
 ('Denali',              'AK', 6198, 20320,  -151.007222,63.069444)
,('Mount Whitney',       'CA', 4421, 14505,  -118.292,36.578583)
...
,('Britton Hill',        'FL',  105,   345,  -86.281944,30.988333)
)
```
### Prepare High Point symbols
The next query does several things:
* translates the `lon` and `lat` location for Alaska and Hawaii high points to match the transformation applied to the state geometry
* computes the `symHeight` attribute for the height of the high point triangle symbol 
* assigns a fill color value to each high point based on the height
* uses `ORDER BY` to sort the high points by latitude, so that their symbols overlap correctly when rendered 
```
,highpt_shape AS (SELECT name, state, hgt_ft, 
    -- translate high points to match shifted states
    CASE WHEN state = 'AK' THEN lon + 18
      WHEN state = 'HI' THEN lon + 32
      ELSE lon END AS lon,
    CASE WHEN state = 'AK' THEN lat - 17
      WHEN state = 'HI' THEN lat + 10
      ELSE lat END AS lat,
    (2.0 * hgt_ft) / 15000.0 + 0.5 AS symHeight,
    CASE WHEN hgt_ft > 14000 THEN '#ffffff'
         WHEN hgt_ft >  7000 THEN '#aaaaaa'
         WHEN hgt_ft >  5000 THEN '#ff8800'
         WHEN hgt_ft >  2000 THEN '#ffff44'
         WHEN hgt_ft >  1000 THEN '#aaffaa'
                             ELSE '#558800'
    END AS clr
  FROM high_pt ORDER BY lat DESC)
```  
### Generate SVG elements
The previous queries transformed the raw data into a form suitable for rendering.  
Now we get to see `pg-svg` in action!
The next query generates the SVG text for each output image element, 
as separate records in a result set called `shapes`.

The SVG elements are generated in the order in which they are drawn - states and labels first, 
with high-point symbols on top. 
Let's break it down:

### SVG for states
The first subquery produces SVG shapes from the state geometries.
The [`svgShape`](https://github.com/dr-jts/pg_svg#svgshape) function produces an SVG shape element for any PostGIS geometry.
It also provides optional parameters supporting other capabilities of SVG.
Here `title` specifies that the state name is displayed as a tooltip,
and `style` specifies the styling of the shape.
Styling in SVG can be provided using properties defined in the 
[Cascaded Style Sheets (CSS)](https://developer.mozilla.org/en-US/docs/Glossary/CSS) specification.
`pg-svg` provides the [`svgStyle`](https://github.com/dr-jts/pg_svg#svgstyle) function to make it easy to specify the 
names and values of CSS styling properties.

Note that the `fill` property value is a URL instead of a color specifier.
This refers to an SVG gradient fill which is defined later.

The state geometry is also included in the subquery result set, for reasons which will be discussed below.
```
,shapes AS (
  -- State shapes
  SELECT geom, svgShape( geom,
    title => name,
    style => svgStyle(  'stroke', '#ffffff',
                        'stroke-width', 0.1::text,
                        'fill', 'url(#state)',
                        'stroke-linejoin', 'round' ) )
    svg FROM us_map
```

### SVG for State Labels
Labels for state abbrevations are positioned at the point produced by the `ST_PointOnSurface` function.
(Alternatively, `ST_MaximumInscribedCircle` could be used.)
The SVG is generated by the [`svgText`](https://github.com/dr-jts/pg_svg#svgtext) function, using the specified styling.
```
  UNION ALL
  -- State names
  SELECT NULL, svgText( ST_PointOnSurface( geom ), abbrev,
    style => svgStyle(  'fill', '#6666ff', 'text-anchor', 'middle', 'font', '0.8px sans-serif' ) )
    svg FROM us_map
```
### SVG for High Point symbols
The high point features are displayed as triangular symbols.
We use the convenient [`svgPolygon`](https://github.com/dr-jts/pg_svg#svgpolygon) function with a simple array of ordinates 
specifying a triangle based at the high point location,
with height given by the previously computed `svgHeight` column.
The title is provided for a tooltip, and the styling uses the computed `clr` attribute as the fill.
```
  UNION ALL
  -- High point triangles
  SELECT NULL, svgPolygon( ARRAY[ lon-0.5, -lat, lon+0.5, -lat, lon, -lat-symHeight ],
    title => name || ' ' || state || ' - ' || hgt_ft || ' ft',
    style => svgStyle(  'stroke', '#000000',
                        'stroke-width', 0.1::text,
                        'fill', clr  ) )
    svg FROM highpt_shape
)
```
### Produce final SVG image

```
SELECT svgDoc( array_agg( svg ),
    viewbox => svgViewbox( ST_Expand( ST_Extent(geom), 2)),
    def => svgLinearGradient('state', '#8080ff', '#c0c0ff')
  ) AS svg FROM shapes;
```

## Conclusion

- You could even use it to generate charts of non-spatial data (although this would be better handled by a more task-specific API).
