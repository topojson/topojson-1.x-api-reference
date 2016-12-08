# API Reference

**The [TopoJSON 2.0 API Reference](https://github.com/topojson/topojson/blob/master/README.md) has moved. This page describes the TopoJSON 1.x API.**

The reference implementation of TopoJSON provides two APIs.

* The [client API](#client-api) supports conversion _from_ TopoJSON (to GeoJSON) for display in a browser.
* The [server API](#server-api) supports conversion _to_ TopoJSON (from various formats), as an alternative to the [command-line tool](Command-Line-Reference).

## Client API

The TopoJSON client API supports converting TopoJSON objects back into GeoJSON for use in a web browser. This functionality is provided by topojson.js, which you can obtain by downloading the [latest TopoJSON release](https://github.com/mbostock/topojson/releases/latest), or by installing `topojson` using npm. You can also download [topojson.js](https://d3js.org/topojson.v1.js) from [d3js.org](http://d3js.org), or add it to your page as follows:

```html
<script src="http://d3js.org/topojson.v1.min.js"></script>
```

This library exposes a single `topojson` object; the version is available as topojson.version per [semantic versioning](http://semver.org). The TopoJSON client API supports so-called “modern” browsers, which generally means everything except IE8 and below.

The TopoJSON client API is implemented using ES2015 modules. In compatible environments, you can import the client methods as a namespace, like so:

```js
import * as topojson from "topojson";
```

See [client.js](https://github.com/mbostock/topojson/blob/master/client.js) for a list of exported symbols. Note that there is no default export, so `import topojson from "topojson"` won’t work!

<a name="feature" href="#feature">#</a> topojson.<b>feature</b>(<i>topology</i>, <i>object</i>)

Returns the GeoJSON Feature or FeatureCollection for the specified *object* in the given *topology*. If the specified object is a GeometryCollection, a FeatureCollection is returned, and each geometry in the collection is mapped to a Feature. Otherwise, a Feature is returned.

Some examples:

* A point is mapped to a feature with a geometry object of type “Point”.
* Likewise for line strings, polygons, and other simple geometries.
* A null geometry object (of type null in TopoJSON) is mapped to a feature with a null geometry object.
* A geometry collection of points is mapped to a feature collection of features, each with a point geometry.
* A geometry collection of geometry collections is mapped to a feature collection of features, each with a geometry collection.

See [the tests](https://github.com/mbostock/topojson/blob/master/test/feature-test.js) for more examples.

<a name="merge" href="#merge">#</a> topojson.<b>merge</b>(<i>topology</i>, <i>objects</i>)

Returns the GeoJSON MultiPolygon geometry object representing the union for the specified array of Polygon and MultiPolygon *objects* in the given *topology*. Interior borders shared by adjacent polygons are removed. See [Merging States](http://bl.ocks.org/mbostock/5416405) for an example.

<a name="mergeArcs" href="#mergeArcs">#</a> topojson.<b>mergeArcs</b>(<i>topology</i>, <i>objects</i>)

Equivalent to [topojson.merge](#merge), but returns a TopoJSON MultiPolygon object rather than GeoJSON.

<a name="mesh" href="#mesh">#</a> topojson.<b>mesh</b>(<i>topology</i>, <i>object</i>, [<i>filter</i>])

Returns the GeoJSON MultiLineString geometry object representing the mesh for the specified *object* in the given *topology*. This is useful for rendering strokes in complicated objects efficiently, as edges that are shared by multiple features are only stroked once.

An optional *filter* function may be specified to prune arcs from the returned mesh using the topology. The filter function is called once for each candidate arc and takes two arguments, *a* and *b*, two geometry objects that share that arc. Each arc is only included in the resulting mesh if the filter function returns true. For typical map topologies the geometries *a* and *b* are adjacent polygons and the candidate arc is their boundary. If an arc is only used by a single geometry then *a* and *b* are identical. This property is useful for separating interior and exterior boundaries; an easy way to produce a mesh of interior boundaries is:

```js
var interiors = topojson.mesh(topology, object, function(a, b) { return a !== b; });
```

See [this county choropleth](http://bl.ocks.org/mbostock/4060606) for example. Note: the *a* and *b* objects are TopoJSON objects (pulled directly from the topology), and not automatically converted to GeoJSON features as by [topojson.feature](#feature).

<a name="meshArcs" href="#meshArcs">#</a> topojson.<b>meshArcs</b>(<i>topology</i>[, <i>objects</i>[, <i>filter</i>]])

Equivalent to [topojson.mesh](#mesh), but returns a TopoJSON MultiLineString object rather than GeoJSON.

<a name="neighbors" href="#neighbors">#</a> topojson.<b>neighbors</b>(<i>objects</i>)

Returns an array representing the set of neighboring objects for each object in the specified *objects* array. The returned array has the same number of elements as the input array; each element *i* in the returned array is the array of indexes for neighbors of object *i* in the input array. For example, if the specified objects array contains the features *foo* and *bar*, and these features are neighbors, the returned array will be [​[1], [0]​], indicating that *foo* is a neighbor of *bar* and *vice versa*. Each array of neighbor indexes for each object is guaranteed to be sorted in ascending order.

For a practical example, see the [world map](http://bl.ocks.org/mbostock/4180634) with topological coloring.

<a name="presimplify" href="#presimplify">#</a> topojson.<b>presimplify</b>(<i>topojson</i>[, <i>triangleArea</i>])

…

## Server API

TopoJSON provides an extended API for use within Node.js that supports conversion from GeoJSON to TopoJSON. (You can also convert ESRI Shapefiles to TopoJSON by converting them to GeoJSON first, say by using [mbostock/shapefile](https://github.com/mbostock/shapefile) or ogr2ogr.) This is the same functionality that is supported by the [topojson command-line tool](Command-Line-Reference), but exposed programmatically for greater control. You can install the TopoJSON server API by running:

```
npm install topojson
```

Then, to require:

```js
var topojson = require("topojson");
```

<a name="topology" href="#topology">#</a> topojson.<b>topology</b>(<i>objects</i>[, <i>options</i>])

Returns the TopoJSON topology representing the merger of the specified map of GeoJSON *objects*. The returned topology’s `objects` will contain an object for each feature, feature collection or geometry in the specified map. For example, if you want to convert a single GeoJSON feature collection to TopoJSON:

```js
var collection = {type: "FeatureCollection", features: […]}; // GeoJSON
var topology = topojson.topology({collection: collection}); // convert to TopoJSON
console.log(topology.objects.collection); // inspect TopoJSON
```

The following *options* may be specified:

* verbose - if truthy, informational messages will be output to stderr.
* coordinate-system - either "cartesian", "spherical" or null to infer the coordinate system automatically.
* stitch-poles - if truthy and using spherical coordinates, polar antimeridian cuts will be stitched.
* quantization - quantization precision; the maximum number of differentiable points per dimension.
* id - a function for computing the id of each input feature.
* property-transform - a function for remapping properties.

If the coordinate system is not specified explicitly, then spherical coordinates will be used if the bounding box of the input objects are approximately within [±180°, ±90°]; otherwise Cartesian coordinates are assumed. Choice of coordinate system has little effect on computing the topology, but note that polar stitching is only enabled in spherical coordinates. If *options* are specified, but the coordinate system is not specified, this method will set the `coordinate-system` option to the inferred coordinate system (either "cartesian" or "spherical").

The *id* function defaults to the following accessor:

```js
function id(d) {
  return d.id;
}
```

Thus, by default, the returned topology will preserve the id of the input features. Specifying a custom *id* function allows properties to be promoted to ids. For example, to promote the `iso_a2` property to the feature id, say:

```js
function id(d) {
  return d.properties.iso_a2;
}
```

The *property transform* function can likewise be used to remap properties on input features to the output topology. The property transform function has the following interface:

```js
function propertyTransform(feature) {
  return {
    "foo": 42,
    "bar": "hello!"
  };
}
```

For example, to preserve all input properties:

```js
function propertyTransform(feature) {
  return feature.properties;
}
```

If the property transform function returns null or an empty object, then the output feature will have no defined properties. The default property transform function drops all properties; use `-p` to preserve properties.

Note that the input to `topojson.topology` is modified in-place, so if you intend to use the GeoJSON input after creating a TopoJSON representation, clone the input first.

<a href="#simplify" name="simplify">#</a> topojson.<b>simplify</b>(<i>topology</i>[, <i>options</i>])

…

<a href="#prune" name="prune">#</a> topojson.<b>prune</b>(<i>topology</i>[, <i>options</i>])

Removes any unused arcs from the specified topology. The following *options* may be specified:

* verbose - if truthy, informational messages will be output to stderr.

<a href="#filter" name="filter">#</a> topojson.<b>filter</b>(<i>topology</i>[, <i>options</i>])

…

<a href="#bind" name="bind">#</a> topojson.<b>bind</b>(<i>topology</i>, <i>propertiesById</i>)

…
