---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.11.5
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

# Retrieving OpenStreetMap data

![](../img/OSM_logo.png)


## What is OpenStreetMap?

OpenStreetMap (OSM) is a global collaborative (crowd-sourced) dataset and project that aims at creating a free editable map of the world containing a lot of information about our environment.
It contains data for example about streets, buildings, different services, and landuse to mention a few. You can view the map at www.openstreetmap.org. You can also sign up as a contributor if you want to edit the map. More details about OpenStreetMap and its contents are available in the [OpenStreetMap Wiki](https://wiki.openstreetmap.org/wiki/Main_Page).

OSM has a large userbase with more than 4 million users and over a million contributers that update actively the OSM database with 3 million changesets per day. In total OSM contains 5 billion nodes and counting! ([stats from November 2019](http://wiki.openstreetmap.org/wiki/Stats)).

OpenStreetMap is used not only for integrating the **OSM maps** as background maps to visualizations or online maps, but also for many other purposes such as **routing**, **geocoding**, **education**, and **research**. OSM is also widely used for humanitarian response e.g. in crisis areas (e.g. after natural disasters) and for fostering economic development. Read more about humanitarian projects that use OSM data from the [Humanitarian OpenStreetMap Team (HOTOSM) website](https://www.hotosm.org).



## OSMnx

This week we will explore a Python module called [OSMnx](https://github.com/gboeing/osmnx)
that can be used to retrieve, construct, analyze, and visualize street networks from OpenStreetMap, and also retrieve data about Points of Interest such as restaurants, schools, and lots of different kind of services. It is also easy to conduct network routing based on walking, cycling or driving by combining OSMnx functionalities with a package called [NetworkX](https://networkx.github.io/documentation/stable/).

To get an overview of the capabilities of the package, see an introductory video given by the lead developer of the package, Prof. Geoff Boeing: ["Meet the developer: Introduction to OSMnx package by Geoff Boeing"](https://www.youtube.com/watch?v=Q0uxu25ddc4&list=PLs9D4XVqc6dCAhhvhZB7aHGD8fCeCC_6N).

There is also a scientific article available describing the package:

- Boeing, G. 2017. ["OSMnx: New Methods for Acquiring, Constructing, Analyzing, and Visualizing Complex Street Networks."](https://www.researchgate.net/publication/309738462_OSMnx_New_Methods_for_Acquiring_Constructing_Analyzing_and_Visualizing_Complex_Street_Networks) Computers, Environment and Urban Systems 65, 126-139. doi:10.1016/j.compenvurbsys.2017.05.004


## NetworkX

We will also use [NetworkX](https://networkx.github.io/documentation/stable/) to for manipulating and analyzing the street network data retrieved from OpenSTreetMap. NetworkX is a Python package that can be used to create, manipulate, and study the structure, dynamics, and functions of complex networks. Networkx module contains algorithms that can be used to calculate [shortest paths](https://networkx.github.io/documentation/networkx-1.10/reference/algorithms.shortest_paths.html)
along road networks using e.g. [Dijkstra's](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) or [A\* algorithm](https://en.wikipedia.org/wiki/A*_search_algorithm).


## Download and visualize OpenStreetMap data with OSMnx

One the most useful features that OSMnx provides is an easy-to-use way of retrieving [OpenStreetMap](http://www.openstreetmap.org) data (using [OverPass API](http://wiki.openstreetmap.org/wiki/Overpass_API)).

In this tutorial, we will learn how to download and visualize OSM data covering a specified area of interest: a district of Kamppi in Helsinki, Finland.

### Street network

OSMnx makes it really easy to do that as it allows you to specify an address to retrieve the OpenStreetMap data around that area. In fact, OSMnx uses the same Nominatim Geocoding API tthat we used for geocoding in Lesson 2.

```python
import osmnx as ox
import matplotlib.pyplot as plt
```

Let's start by specifying ``"Kamppi, Helsinki, Finland"`` as the place from where the data should be downloaded. The place name should be *geocodable* which means that the place name should exist in the OpenStreetMap database (you can do a test search at https://www.openstreetmap.org/ or at https://nominatim.openstreetmap.org/ to verify that the place name is valid).  

```python
# Specify the name that is used to seach for the data
place_name = "Kamppi, Helsinki, Finland"
```

Next, we will read in the OSM street network using OSMnx using the [graph_from_place](https://osmnx.readthedocs.io/en/stable/osmnx.html?highlight=graph%20from#osmnx.graph.graph_from_place) function:

```python
# Fetch OSM street network from the location
graph = ox.graph_from_place(place_name)
```

Check the data type of the graph:

```python
type(graph)
```

What we have here is a networkx [MultiDiGraph](https://networkx.org/documentation/networkx-1.10/reference/classes.multidigraph.html) object. 

Let's have a closer look a the street nework. OSMnx has its own function [plot_graph()](https://osmnx.readthedocs.io/en/stable/osmnx.html?highlight=plot_graph#osmnx.plot.plot_graph) for visualizing graph objects. The function utilizes Matplotlib for visualizing the data,
hence as a result it returns a matplotlib figure and axis objects:


```python
# Plot the streets
fig, ax = ox.plot_graph(graph)
```

Great! Now we can see that our graph contains nodes (the points) and edges (the lines) that connects those nodes to each other.



### Graph to GeoDataFrame

We can now plot all these different OSM layers by using the familiar `plot()` function of Geopandas. As you might remember, the street network data is not a GeoDataFrame, but a graph object. Luckily, OSMnx provides a convenient function `graph_to_gdfs()` that can convert the graph into two separate GeoDataFrames where the first one contains the information about the nodes and the second one about the edge.

Let's extract the nodes and edges from the graph as GeoDataFrames:

```python
# Retrieve nodes and edges
nodes, edges = ox.graph_to_gdfs(graph)
```

```python
nodes.head()
```

```python
edges.head()
```

<!-- #region -->
Nice! Now, as we can see, we have our graph as GeoDataFrames and we can plot them using the same functions and tools as we have used before.

<div class="alert alert-info">

**Note**

There are also other ways of retrieving the data from OpenStreetMap with OSMnx such as passing a Polygon to extract the data from that area, or passing Point coordinates and retrieving data around that location with specific radius. Take a look of this [tutorial to find out how to use those features of OSMnx](https://github.com/gboeing/osmnx-examples/blob/master/notebooks/01-overview-osmnx.ipynb).


</div>
<!-- #endregion -->

### Place polygon

Let's also plot the Polygon that represents our area of interest (Kamppi, Helsinki). We can retrieve the Polygon geometry using the [geocode_to_gdf()](https://osmnx.readthedocs.io/en/stable/osmnx.html?highlight=geocode_to_gdf(#osmnx.geocoder.geocode_to_gdf) -function.

```python
# Get place boundary related to the place name as a geodataframe
area = ox.geocode_to_gdf(place_name)
```

As the name of the function already tells us, `gdf_from_place()`returns a GeoDataFrame based on the specified place name query.
Let's still verify the data type: 

```python
# Check the data type
type(area)
```

Let's also have a look at the data:

```python
# Check data values
area
```

```python
# Plot the area:
area.plot()
```

### Building footprints

It is also possible to retrieve other types of OSM data features with OSMnx such as buildings or points of interest (POIs). Let's download the buildings with `OSMnx` [geometries_from_place()](https://osmnx.readthedocs.io/en/stable/osmnx.html?highlight=geometries_from_place#osmnx.geometries.geometries_from_place) -function and plot them on top of our street network in Kamppi. 

*Note: in OSMnx versions < 0.9 we used the `buildings_from_place` method to retrieve building footprints.*


When fetching spesific types of geometries from OpenStreetMap using OSMnx `geometries_from_place` we also need to specify the correct tags. For getting [all types of buildings](https://wiki.openstreetmap.org/wiki/Buildings), we can use the tag `building=yes`.

```python
# List key-value pairs for tags
tags = {"building": True}
```

```python
buildings = ox.geometries_from_place(place_name, tags)
```

Let's check how many building footprints we received:

```python
len(buildings)
```

Let's also have a look at the data:

```python
buildings.head()
```

As you can see, there are several columns in the buildings-layer. Each column contains information about a spesific tag that OpenStreetMap contributors have added. Each tag consists of a key (the column name), and several potential values (for example `building=yes` or `building=school`). Read more about tags and tagging practices in the [OpenStreetMap wiki](https://wiki.openstreetmap.org/wiki/Tags). 

```python
buildings.columns
```

### Points-of-interest

It is also possible to retrieve other types of geometries from OSM using the `geometries_from_place` by passing different tags. Point-of-interest (POI) is a generic concept that describes point locations that represent places of interest. 

In OpenStreetMap, many POIs are described using the [amenity-tags](https://wiki.openstreetmap.org/wiki/Key:amenityhttps://wiki.openstreetmap.org/wiki/Key:amenity). 
We can, for excample, retrieve all restaurat locations by referring to the tag `amenity=restaurant`. See all available amenity categories from [OSM wiki](https://wiki.openstreetmap.org/wiki/Key:amenity). 

*Note: We used the `pois_from_place()` method to retrieve POIs in older versions of OSMnx.*

Let's retrieve restaurants that are located in our area of interest:

```python
# List key-value pairs for tags
tags = {"amenity": "restaurant"}
```

```python
# Retrieve restaurants
restaurants = ox.geometries_from_place(place_name, tags)

# How many restaurants do we have?
len(restaurants)
```

As we can see, there are quite many restaurants in the area.

Let's explore what kind of attributes we have in our restaurants GeoDataFrame:

```python
# Available columns
restaurants.columns.values
```

As you can see, there is quite a lot of (potential) information related to the amenities. Let's subset the columns and inspect the data further. Useful columns include at least `name`, `address information` and `opening_hours` information:

```python
# Select some useful cols and print
cols = [
    "name",
    "opening_hours",
    "addr:city",
    "addr:country",
    "addr:housenumber",
    "addr:postcode",
    "addr:street",
]

# Print only selected cols
restaurants[cols].head(10)
```

As we can see, there is a lot of useful information about restaurants that can be retrieved easily with OSMnx. Also, if some of the information need updating, you can go over to www.openstreetmap.org and edit the source data! :)


### Plotting the data

Let's create a map out of the streets, buildings, restaurants, and the area Polygon but let's exclude the nodes (to keep the figure clearer).

```python
fig, ax = plt.subplots(figsize=(12, 8))

# Plot the footprint
area.plot(ax=ax, facecolor="black")

# Plot street edges
edges.plot(ax=ax, linewidth=1, edgecolor="dimgray")

# Plot buildings
buildings.plot(ax=ax, facecolor="silver", alpha=0.7)

# Plot restaurants
restaurants.plot(ax=ax, color="yellow", alpha=0.7, markersize=10)
plt.tight_layout()
```

Cool! Now we have a map where we have plotted the restaurants, buildings, streets and the boundaries of the selected region of 'Kamppi' in Helsinki. And all of this required only a few lines of code. Pretty neat! 



<div class="alert alert-info">

**Check your understading**

Retrieve OpenStreetMap data from some other area! Download these elements using OSMnx functions from your area of interest:
    
- Extent of the area using `geocode_to_gdf()`
- Street network using `graph_from_place()`, and convert to gdf using `graph_to_gdfs()`
- Building footprints (and other geometries) using `geometries_from_place()` and appropriate tags.
    
*Note, the larger the area you choose, the longer it takes to retrieve data from the API! Use parameter `network_type=drive` to limit the graph query to filter out un-driveable roads.*

</div>

```python
# Specify the name that is used to seach for the data. Check that the place name is valid from https://nominatim.openstreetmap.org/ui/search.html
my_place = ""
```

```python
# Get street network
```

```python
# Get building footprints
```

```python
# Plot the data
```

### Extra: Park polygons
Notice that we can retrieve all kinds of different features from OpenStreetMap using the [geometries_from_place()](https://osmnx.readthedocs.io/en/stable/osmnx.html?highlight=geometries_from_place#osmnx.geometries.geometries_from_place) method by passing different OpenStreetMap tags.

Let's try to fetch all public parks in the Kamppi area. In OpenStreetMap, parks are often tagged as `leisure=park`. We can also add other green surfaces, such as `landuse=grass`. see OpenStreetMap, and OSM wiki for more details.

- We need to start by fetching all footprints from the tag `leisure`:

```python
# List key-value pairs for tags
tags = {"leisure": "park", "landuse": "grass"}
```

```python
# Get the data
parks = ox.geometries_from_place(place_name, tags)

# Check the result
print("Retrieved", len(parks), "objects")
```

let's check the first rows:

```python
parks.head(3)
```

Check all column headers:

```python
parks.columns.values
```

plot the parks:

```python
parks.plot(color="green")
```

Finally, we can add the park polygons to our map:

```python
fig, ax = plt.subplots(figsize=(12, 8))

# Plot the footprint
area.plot(ax=ax, facecolor="black")

# Plot the parks
parks.plot(ax=ax, facecolor="green")

# Plot street edges
edges.plot(ax=ax, linewidth=1, edgecolor="dimgray")

# Plot buildings
buildings.plot(ax=ax, facecolor="silver", alpha=0.7)

# Plot restaurants
restaurants.plot(ax=ax, color="yellow", alpha=0.7, markersize=10)
plt.tight_layout()
```
