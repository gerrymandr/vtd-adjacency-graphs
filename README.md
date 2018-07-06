# VTD Adjacency Graphs

A repository of rook- and queen-adjacency graphs of state VTDs, made from 2012 Tiger/Line data.

## What's inside

Besides Rhode Island, all of these graphs were created using [v0.1.0](https://github.com/gerrymandr/graphmaker/releases/tag/v0.1.0) of [graphmaker](https://github.com/gerrymandr/graphmaker). This is the exact version that we used to build these graphs. Up to configuring the dependencies, the results should be exactly reproducible using that version of the graphmaker code.

The nodes are 'Voting Tabulation Districts,' which are an idealized notion of a voting precinct. They almost never line up perfectly, because VTDs are fixed at the time of the decennial Census, while precincts are living organisms that grow and change along with the population.

The edges describe geographic adjacency relationships between the nodes. Two VTDs are rook adjacent if they have some shared boundary. Two VTDs are queen adjacent if they have shared boundary or meet at a corner.

### Folder structure

In the root directory, you'll find a folder for [almost every](#whats-missing) state, named by the 2-digit FIPS code for that state. You can find a matching of state names to FIPS codes [here](https://www.mcc.co.mercer.pa.us/dps/state_fips_code_listing.htm).

Within each state's directory you'll find three files:

- `rook.json`: The rook-adjacency graph of the state VTDs.
- `queen.json`: The queen-adjacency graph of the state VTDs.
- `report.json`: A JSON document containing some notable statistics (e.g. the distribution of the degrees of the nodes), for each of the graphs, along with a brief comparison of the queen- and rook-adjacency graphs.

### The data included in the graphs

As of now, each graph contains the following data:

- **Identifying data**
  - Nodes are named by their GEOIDs, read from the GEOID10 column in the original shapefile.
- **Boundary data**
  - Each node has a `'boundary_node'` attribute that is `True` if the node touches the border of the state, and `False` if it does not. If it does, the node also has a `'boundary_perim'` attribute specifying the length in meters of the boundary shared by the state and VTD corresponding to that node.
  - Each edge has a `'shared_perim'` attribute. This gives the length in meters of the shared boundary of the adjacent VTDs connected by the edge.
- **Information from the Census shapefiles**. Each node has the following attributes:
  - `'COUNTYFP10'`: The FIPS code of the county containing the VTD.
  - `'ALAND10'`: The land area, in square meters, as computed by the US Census Bureau.
  - `'AWATER10'`: The water area, in square meters, as computed by the US Census Bureau.
  - `'NAME10'`: A name for the VTD.
- **Districting data**. Each node has a `'CD'` attribute giving its assignment to a congressional district. We used the US Census Bureau [block assignment files](https://www.census.gov/geo/maps-data/data/baf.html) to identify which CD each VTD belonged to. We think these districting plans are those drawn immediately after the 2010 Census, but have not completely verified that this is the case.

#### How we computed lengths

The `'boundary_perim'` and `'shared_perim'` lengths are in meters, computed in an appropriate UTM projection. For each state, we chose the UTM zone that contained the majority of the VTDs and computed the lengths in that projection. The code for this process is in the `graphmaker/geospatial.py` module.

#### What's missing?

We do not have VTD or precinct records for Kentucky (FIPS code 21).

#### Rhode Island

Rhode Island did not contribute their VTD shapefiles to the US Census Bureau's 2012 Tiger/Line dataset. Luckily, Rhode Island has a GIS organization that provides [actual precinct shapefiles](http://www.rigis.org/datasets/voting-precincts). We generated the adjacency graphs for Rhode Island from those 2016 precinct shapefiles. For now, the Rhode Island graphs do not have area measurements, but we will compute these soon.

### Source shapefiles

The source shapefiles used to construct the graphs can be found at [https://www2.census.gov/geo/tiger/TIGER2012/VTD/](https://www2.census.gov/geo/tiger/TIGER2012/VTD/). The [graphmaker](https://github.com/gerrymandr/graphmaker/) package includes a module `get_vtds` with a function `download_state_vtds` that will download and unzip the shapefiles for a given state (specificed by FIPS) for you.

## Loading a graph in python

These graphs can be parsed into a [NetworkX](https://networkx.github.io/documentation/stable/index.html) Graph object using the `networkx.readwrite.json_graph.adjacency_graph` method. Here's an example of how to load the rook-adjacency graph for Alaska (FIPS code 02):

```python
import networkx
import json

with open('./02/rook.json') as f:
    data = json.load(f)

graph = networkx.readwrite.json_graph.adjacency_graph(data)
```

## Usage with RunDMCMC

Our [RunDMCMC](https://github.com/gerrymandr/RunDMCMC) module gives you the power to run a Markov chain on the space of districting plans for a given state. You can use a graph from this repository with RunDMCMC to conduct a local sampling of districting plans.
