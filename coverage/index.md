# Simple Polygonal Coverages in PostGIS

An important kind of spatial data model is a polygon coverage.  This is a set of polygonal areas which fully or partially cover a spatial extent.  
In alternate terms, a polygonal coverage models a region in which every point has a single value out of a set of values.
Polygonal coverages are used to model a wide variety of natural phenomena, such as:

* biogeoclimatic zones
* surficial geology
* land use
* hydrological basins

as well human-defined categorizations, such as:

* cadastral boundaries
* jurisdictional boundaries (e.g. country and adminstrative borders)

The traditional way of representing polygonal coverages is to use a topological structure containing faces, edges and nodes.
PostGIS provides the Topology model to support this representation.  

An alternative way of storing polygonal coverages is to simply store the discrete polygons which make up the dataset. 
This has some advantages in terms of management and querying.  
It also provides better compatibility with external datasets and tools.
We call this the **Simple Polygonal Coverage** format, referring both to its conceptual simplicity as well as the fact that 
it is defined using the polygons defined by the Simple Features Model.

Obviously this format imposes some stringent requirements on the polygons in a dataset.  They must be non-overlapping, etc...
In order to support this, we are providing a function in PostGIS to validate that a set of polygons forms a clean coverage.
It also reports where polygons violate the coverage semantics, to aid in detecting and fixing problems.

A major benefit of simple polygon coverage topology is that some operations can be executed much faster and more effectively.
Two important ones are now supported:  coverage union and coverage simplification.  These are described below.







Prior art
- Mapshaper
- TopoJSON
- GRASS
- pprepair

- how does this compare to PostGIS Topology?


## Coverage Operations

### Validation

### Union

### Simplification

## Future functionality
- Cleaning
- Overlay
- 
