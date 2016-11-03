# Command Line Reference

**The [TopoJSON 2.0 API Reference](https://github.com/topojson/topojson/blob/master/README.md) has moved. This page describes the TopoJSON 1.x API.**

In this document, “TopoJSON” refers to the [serialization format](Specification), whereas `topojson` refers to the command-line application you can [install](Installation) via [NPM](http://npmjs.org). `topojson` is a reference implementation for converting data to the TopoJSON format, but not the only implementation; for example, see Sean Gillies’ [topojson.py](https://sgillies.net/2012/11/21/topojson-with-python.html).

## Usage

```text
topojson [options] -- [file …]
```

Input files can be one or more of:

* <i>.json</i> <font color="#777">GeoJSON or TopoJSON</font>
* <i>.shp</i> <font color="#777">ESRI shapefile</font>
* <i>.csv</i> <font color="#777">comma-separated values (CSV)</font>
* <i>.tsv</i> <font color="#777">tab-separated values (TSV)</font>

Options:

* <i>-o, --out</i> <font color="#777">output TopoJSON file name</font>
* <i>-q, --quantization, --no-quantization</i> <font color="#777">maximum number of differentiable points along either dimension</font>
* <i>-s, --simplify</i> <font color="#777">precision threshold for Visvalingam simplification</font>
* <i>--simplify-proportion</i> <font color="#777">proportion of points to retain for Visvalingam simplification</font>
* <i>--cartesian</i> <font color="#777">assume Cartesian input coordinates</font>
* <i>--spherical</i> <font color="#777">assume spherical input coordinates</font>
* <i>--no-force-clockwise</i> <font color="#777">do not force clockwise exterior rings and counterclockwise interior rings</font>
* <i>--no-stitch-poles</i> <font color="#777">do not splice antimeridian cuts for polygons that encompass a pole</font>
* <i>--filter</i> <font color="#777">preserve null or empty features (e.g., zero-area polygons)</font>
* <i>--bbox</i> <font color="#777">include bbox property in generated topology</font>
* <i>--id-property</i> <font color="#777">name of feature property to promote to geometry id</font>
* <i>-p, --properties</i> <font color="#777">feature properties to preserve; no name preserves all properties</font>
* <i>-e, --external-properties</i> <font color="#777">CSV or TSV file to join properties (by id) to output features</font>
* <i>--shapefile-encoding</i> <font color="#777">character encoding for reading shapefile properties</font>
* <i>--ignore-shapefile-properties</i> <font color="#777">skip reading shapefile properties (.dbf) for faster performance</font>
* <i>-x, --longitude</i> <font color="#777">name of the longitude property for CSV or TSV geometry input</font>
* <i>-y, --latitude</i> <font color="#777">name of the latitude property for CSV or TSV geometry input</font>
* <i>--projection</i> <font color="#777">project spherical input geometry using a D3 geographic projection</font>
* <i>--width</i> <font color="#777">scale and translate to fit a viewport of the specified width</font>
* <i>--height</i> <font color="#777">scale and translate to fit a viewport of the specified height</font>
* <i>--margin</i> <font color="#777">pixels of margin to reserve when scaling to fit a viewport</font>

## Input and Output

* <i>-o, --out</i> <font color="#777">output TopoJSON file name</font>

The simplest usage of `topojson` is:

```bash
topojson -o output.json input.json
```

This converts the GeoJSON file, input.json, to a TopoJSON file, output.json. Equivalently, you can output to stdout and then redirect it to a file:

```bash
topojson input.json > output.json
```

You can specify [ESRI shapefiles](http://www.esri.com/library/whitepapers/pdfs/shapefile.pdf) as input:

```bash
topojson -o output.json input.shp
```

You can even specify [CSV](http://en.wikipedia.org/wiki/Comma-separated_values) or TSV files as input, where each row defines the properties "longitude" and "latitude", representing a point:

```bash
topojson -o output.json input.csv
```

Use `--` to separate the input files from the output files and options, for clarity and safety. If you list multiple input files, they will be combined into a single output TopoJSON file:

```
topojson -o foobar.json -- foo.json bar.json
```

The generated output TopoJSON file, here foobar.json, contains a *topology* as described in the [specification](Specification). The topology contains a map of named objects; the names of the objects are inferred from the file name, minus any enclosing directories and file extensions. (Be careful not to pass multiple input files with the same name in different directories!) The above foobar.json looks something like this:

```json
{
  "type": "Topology",
  "transform": …,
  "objects": {
    "foo": …,
    "bar": …
  },
  "arcs": …
}
```

Thus, topology.objects.foo is the converted contents of the input foo.json file, and topology.objects.bar is the converted contents of the input bar.json file. If you want to assign explicit names to the objects in the generated topology, rather than using the file name, you can prefix the input file with `name=`. For example, the following command will use the name “states” for us-states.json and “counties” for us-counties.json:

```
topojson -o us.json -- states=us-states.json counties=us-counties.json
```

All objects in a generated topology share the same set of arcs, even if they come from different input files!

### Supported Input Formats

`topojson` supports [GeoJSON](http://geojson.org/), [ESRI Shapefiles](http://www.esri.com/library/whitepapers/pdfs/shapefile.pdf) (via a JavaScript [shapefile parser](https://github.com/mbostock/shapefile)), CSV, TSV and TopoJSON as input. The type of input is inferred from the input file extensions:

* <i>.json</i> <font color="#777">GeoJSON or TopoJSON</font>
* <i>.shp</i> <font color="#777">ESRI shapefile</font>
* <i>.csv</i> <font color="#777">comma-separated values (CSV)</font>
* <i>.tsv</i> <font color="#777">tab-separated values (TSV)</font>

If your input geometry is not in one of these common formats, use [ogr2ogr](http://www.gdal.org/ogr2ogr.html) to convert it first. See the [Let’s Make a Map](http://bost.ocks.org/mike/map/) tutorial for tips on installing ogr2ogr and GDAL. ogr2ogr has many other useful features not supported by `topojson`, including clipping, filtering and reprojection. For example, to reproject a shapefiles to the [World Geodetic System](http://en.wikipedia.org/wiki/World_Geodetic_System), [EPSG:4326](http://spatialreference.org/ref/epsg/4326/):

```bash
ogr2ogr -f 'ESRI Shapefile' -t_srs EPSG:4326 input-fixed.shp input.shp
topojson -o output.json -- input-fixed.shp
```

### CSV or TSV Points

* <i>-x, --longitude</i> <font color="#777">name of the longitude property for CSV or TSV geometry input</font>
* <i>-y, --latitude</i> <font color="#777">name of the latitude property for CSV or TSV geometry input</font>

For CSV or TSV geometry input, each row represents a point feature. The names of the longitude and latitude properties can be configured using the `-x` and `-y` (or equivalently `--longitude` and `--latitude`) options, respectively. Any remaining properties can be used as output properties via the `-p` argument, as described below.

## Quantization

* <i>-q, --quantization, --no-quantization</i> <font color="#777">maximum number of differentiable points along either dimension</font>

Unlike GeoJSON, ESRI shapefiles, and other popular formats, TopoJSON uses quantized (fixed-point) coordinates by default, rather than floating-point coordinates. This means that you need to tell `topojson` how much precision to retain in the input geometry. The more precision you retain, the more accurate the encoded geometry, and the larger the file size. (See [Josh Livni’s post](http://www.porcupinealley.com/2014/08/topojson.html) for an example of error due to over-quantization.)

To disable quantization, specify `--no-quantization`. This will result in significantly larger files, and may also produce messier topologies due to inaccuracies in non-topological inputs.

If you are generating a small static map in the browser, the default quantization factor of 10,000 (`-q 1e4`) is probably sufficient. This means that there are a maximum of ten thousand differentiable values along each dimension. If you are generating a large high-resolution map, or if you want to allow the user to zoom in and see greater detail, you should increase the quantization factor by at least one or two orders of magnitude: `-q 1e5` or `-q 1e6`.

When you run `topojson` on the command line, it will output the quantization precision in meters along *x* and *y*. For example:

```text
quantization: 400m (0.00360°) 188m (0.00169°)
```

The precision is the interval between quantized fixed-point output coordinates. Above, 0.00360° (12.96[″](http://en.wikipedia.org/wiki/Minute_of_arc)) corresponds to 400 meters, assuming a spherical Earth of radius 6,371 kilometers. If you expect to display this map at a resolution higher than 400 meters per pixel, then you should increase the quantization accordingly!

Quantization causes coincident points. You may want to increase the quantization factor to produce a cleaner topology, as overly-quantized inputs may result in excessive arc intersection due to these coincident points. Coincident points along an arc will be removed, and thus the output geometry may have fewer coordinates than the input geometry. In the extreme case, such as highly-quantized small islands, the geometry objects can collapse down to a single point; see the `--filter` argument below to control whether these degenerate features are included in the output.

## Simplification

* <i>-s, --simplify</i> <font color="#777">precision threshold for Visvalingam simplification</font>
* <i>--simplify-proportion</i> <font color="#777">proportion of points to retain for Visvalingam simplification</font>

Geometry simplification is different from quantization, but is another method of reducing the complexity of the geometry and the output file size substantially. See my tutorial on [Visvalingam simplification](http://bost.ocks.org/mike/simplify/) for an explanation of the algorithm and a demonstration.

Simplification is controlled using either the `-s` (`--simplify`) argument, which specifies an area threshold. The area is measured in [steradians](http://en.wikipedia.org/wiki/Steradian) for spherical coordinates, or local square units when used with `--cartesian`. You can instead use the `--simplify-proportion` argument to specify the fraction of coordinates (roughly corresponding to output file size) to retain. Use the latter if it is convenient, but specifying an explicit area threshold using `-s` may be safer assuming you know the output resolution of the map you are going to display. The `-s` threshold should be less than the per-pixel area of your map to avoid visual artifacts. Note that in the case of projections that are not equal-area, artifacts caused by over-simplification will be significantly more visible in areas of larger scale, such as near the poles when using the [Mercator projection](http://en.wikipedia.org/wiki/Mercator_projection). When in doubt, try it and see!

When using Cartesian coordinates, the meaning of the simplification threshold changes to be in projected coordinates. In conjunction with the `--width` and `--height` arguments (described below), you can specify a simplification threshold in pixels, which is convenient!

Simplification preserves topology; rather than running on each geometry object independently, only the topology’s arcs are simplified. Thus, shared borders remain shared after simplification. Note that because topology is inferred from coincident points after quantization, the behavior of simplification is dependent on the quantization factor (`-q`) as described above. If simplification behaves unexpectedly, try increasing quantization precision by a factor of ten (e.g., `-q 1e5`).

## Geometry

* <i>--cartesian</i> <font color="#777">assume Cartesian coordinates</font>
* <i>--spherical</i> <font color="#777">assume spherical coordinates</font>
* <i>--no-force-clockwise</i> <font color="#777">do not force clockwise exterior rings and counterclockwise interior rings</font>
* <i>--no-stitch-poles</i> <font color="#777">do not splice antimeridian cuts for polygons that encompass a pole</font>
* <i>--filter</i> <font color="#777">preserve null or empty features (e.g., zero-area polygons)</font>
* <i>--projection</i> <font color="#777">project spherical input geometry using a D3 geographic projection</font>
* <i>--width</i> <font color="#777">scale and translate to fit a viewport of the specified width</font>
* <i>--height</i> <font color="#777">scale and translate to fit a viewport of the specified height</font>
* <i>--margin</i> <font color="#777">pixels of margin to reserve when scaling to fit a viewport</font>
* <i>--bbox</i> <font color="#777">include bbox property in generated topology</font>

Input files must be topologically-consistent, in order to generate valid topological output. This means that for a shared arc between two features, the coordinates of the arc must be in exactly the same position and in the same order (or the reversed order). Spurious intermediate coordinates, or non-aligned coordinates, can prevent `topojson` from detecting shared arcs and producing detached geometry objects.

Some input formats do not require a winding order for polygons; TopoJSON, in contrast, uses the right hand rule. Thus, by default, `topojson` forces all polygons to be clockwise. Without this step, small input polygons with opposite winding order would cover more than one hemisphere. See [Jason Davies’ clipping demonstration](https://www.jasondavies.com/maps/clip/) for an example of such large polygons. If you wish to retain these polygons of unusual size, use the `--no-force-clockwise` argument to disable this behavior.

By default, `topojson` assumes spherical coordinates if all input coordinates are within the range [±180°, ±90°]. If any coordinate is outside this range, `topojson` instead assumes Cartesian coordinates. You can make this behavior explicit by specifying either the `--spherical` or `--cartesian` option. When using spherical coordinates, simplification is performed using spherical triangle area (per [L’Huiler’s theorem](http://mathworld.wolfram.com/LHuiliersTheorem.html)); in Cartesian coordinates, [planar triangle area](http://mathworld.wolfram.com/TriangleArea.html) is used instead. It is an error to use `--spherical` if any coordinates are outside the range [±180°, ±90°]. Also note that using `--cartesian` implicitly changes the meaning of the `--simplify` argument to take square units rather than steradians.

When using spherical coordinates, `topojson` will remove extraneous antimeridian cuts for polygons that encompass a pole, such as Antarctica. Use the `--no-stitch-poles` argument to disable this behavior (or use Cartesian coordinates).

Normally `topojson` does not include null and empty objects in the output topology. These can arise either because they were present in the input, or as a result of quantization or simplification. Use the `--filter=none` argument to retain empty and null features. (Null geometry objects in TopoJSON have an undefined type, as described in the [specification](Specification).) Use `--filter=small` if you want small rings that are attached to other rings to be removed; the default filter behavior, `--filter=small-detached`, only removes small rings that are not attached to other polygons, such as islands.

To include a `bbox` property in the generated topology that represents the bounding box of all contained features, use the `--bbox` argument. The bounding box array follows the same format as GeoJSON: [x0, y0, x1, y1].

You can use `--projection` to convert spherical input geometry to Cartesian coordinates via a [D3 geographic projection](https://github.com/d3/d3-geo-projection). For example, `--projection 'd3.geo.albersUsa()'` will project geometry using a composite Albers equal-area conic projection suitable for the contiguous United States, Alaska and Hawaii.

When using Cartesian coordinates, use the `--width` and `--height` arguments to scale and translate the geometry to fit a viewport of the specified size. If `--width` is specified but not `--height`, or if `--height` is specified but not `--width`, the geometry is scaled to fit just the width or the height, respectively. An additional `--margin` argument can be used to reserve some pixels of padding from the edge of the viewport. See the [projected TopoJSON example](http://bl.ocks.org/mbostock/5557726).

## Properties

* <i>-p, --properties</i> <font color="#777">feature properties to preserve; no name preserves all properties</font>
* <i>--shapefile-encoding</i> <font color="#777">character encoding for reading shapefile properties</font>
* <i>--ignore-shapefile-properties</i> <font color="#777">skip reading shapefile properties (.dbf) for faster performance</font>

By default, `topojson` removes all properties. Use the `-p` (`--properties`) argument to retain some or all properties.

If you only want to retain *some* properties, pass an input property name to the `-p` argument. For example, `-p foo` will retain the property "foo". You can specify multiple properties to retain either by using multiple `-p` arguments, as in `-p foo -p bar`, or by using a comma to separate the property names, as in `-p foo,bar`.

If you want to retain *all* properties, pass the `-p` argument as the last option on the command line, with no value. Alternately use `--` to terminate command line flags. Ie: `topojson -o output.json -p -- input.json`.

The `-p` argument also allows you to rename properties, should you wish to convert an awkward input property name to a more usable output property name. Use the form `-p target=source` to rename an input property named "source" to the output property named "target". Renaming properties also works with multiple properties. For example, to downcase the property names "FOO" and "BAR", use `-p foo=FOO,bar=BAR`.

If the properties referenced by `-p` are null or undefined, they are omitted from the output geometry object. If the output geometry object has no defined properties (that is, all properties are null or undefined), the geometry object will *not* have a properties object. Be careful dereferencing properties from geometry objects in the client, as dereferencing a property will throw an error if the geometry object has no properties object!

You can use `-p` to map multiple input properties to the same output property name. For example, if you want the output property "name" to be defined from either the input property "STATE" or "COUNTY", use `-p name=STATE,name=COUNTY`. However, if an input feature has both properties ("STATE" and "COUNTY") defined, the value of the last one ("COUNTY") wins.

The `-p` argument can coerce input property values to numbers, if desired. Prepend a `+` in front of the input property name to coerce its value to a number:

```bash
topojson -o output.json -p +iso_n3 -- input.shp
```

(Note: it is not currently possible to preserve properties whose name contains a comma, leading plus or equals sign.)

When using shapefiles as input, the default encoding is [ISO 8859-1](http://en.wikipedia.org/wiki/ISO/IEC_8859-1) for legacy reasons. You can change the encoding to the more expressive [UTF-8](http://en.wikipedia.org/wiki/UTF-8) encoding by saying `--shapefile-encoding utf8`.

When reading an input shapefile from a file, TopoJSON attempts to load a matching .dbf file with the same filename prefix to obtain shapefile properties. If the .dbf file does not exist, an error will be thrown. To prevent the file from being loaded, use `--ignore-shapefile-properties`. This also improves performance if the properties are not needed.

### IDs

* <i>--id-property</i> <font color="#777">name of feature property to promote to geometry id</font>

By default, `topojson` retains the `id` attribute of input features. If you wish to promote a property to the feature ID as part of the conversion to TopoJSON, use the `--id-property` argument. For example, to promote the "iso_3166_2" property to the ID:

```bash
topojson -o output.json --id-property iso_3166_2 -- input.shp
```

The `--id-property` argument can also be used to coerce input property values to numeric IDs, if desired. Prepend a `+` in front of the input property name to coerce its value to a number:

```bash
topojson -o output.json --id-property +iso_n3 -- input.shp
```

If the properties referenced by `--id-property` are null or undefined, they are omitted from the output geometry object. Thus, the generated objects may not have a defined ID if the input features did not have a property with the specified name.

The `--id-property` argument can take multiple input property names to compute the ID. For example, if you want the ID be defined from either the input property "STATE" or "COUNTY", use `--id-property STATE,COUNTY`. If an input feature has both properties ("STATE" and "COUNTY") defined, the value of the first one ("STATE") wins. Note that this behavior is different from the `-p` option.

(Note: it is not currently possible to generate an ID from properties whose name contains a comma, leading plus or equals sign.)

### External Properties

* <i>-e, --external-properties</i> <font color="#777">CSV or TSV file to join properties (by id) to output features</font>

For convenience, you can specify a CSV or TSV file to `topojson` that specifies additional properties that you want to bind to output geometry objects. For example, say you had a TSV file like this:

```text
FIPS	rate
1001	.097
1003	.091
1005	.134
1007	.121
1009	.099
1011	.164
1013	.167
1015	.108
1017	.186
1019	.118
1021	.099
…
```

Using `-e`, you can map these to a numeric output property named "unemployment":

```bash
topojson \
  -o output.json \
  -e unemployment.tsv \
  --id-property=+FIPS \
  -p unemployment=+rate \
  -- input.shp
```

Here, the input property "FIPS" is coerced to a number and used as the feature identifier; likewise, the column named "FIPS" is used as the identifier in the CSV file. (If your CSV file uses a different column name for the feature identifier, you can specify multiple id properties, such as `--id-property=+FIPS,+id`.) Then, the output property "unemployment" is generated from the external data file, unemployment.tsv, which defines the input property "rate". This property is also coerced to a number, which is necessary because CSV and TSV files are untyped.
