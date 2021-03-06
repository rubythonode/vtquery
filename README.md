# vtquery

[![Build Status](https://travis-ci.org/mapbox/vtquery.svg?branch=master)](https://travis-ci.org/mapbox/vtquery)
[![node-cpp-skel](https://mapbox.s3.amazonaws.com/cpp-assets/node-cpp-skel-badge_blue.svg)](https://github.com/mapbox/node-cpp-skel)

    npm install @mapbox/vtquery

Get the closest features from a longitude/latitude in a set of vector tile buffers. Made possible with node-cpp-skel, vtzero, geometry.hpp, spatial-algorithms, and cheap-ruler-cpp.

The two major use cases for this library are:

1.  To get a list of the features closest to a query point
2.  Point in polygon checks

# Response object

The response object is a GeoJSON FeatureCollection with Point features containing the following in formation:

-   Geometry - always a `Point`. This library does not return full geometries of features. It only returns the closet point (longitude/latitude) of a feature. If the original feature is a point, the result is the actual location of that point. If the original feature is a linestring or polygon, the result is the interpolated closest latitude/longitude point of that feature. This could be _along_ a line of a polygon and not an actual node of a polygon. If the query point is _within_ a polygon, the result will be the query point location.
-   Properties of the feature
-   Extra properties including:
    -   `tilequery.geometry_type` - either "Point", "Linestring", or "Polygon"
    -   `tilequery.distance` in meters - if distance is `0.0`, the query point is _within_ the geometry (point in polygon)
    -   `tilequery.layer` which layer the feature was a part of in the vecto tile buffer
-   An `id` if it existed in the vector tile feature

Here's an example response

```JSON
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          -122.42901,
          37.80633
        ]
      },
      "properties": {
        "property_one": "hello",
        "property_two": 15,
        "tilequery": {
          "distance": 11.3,
          "geometry": "Point",
          "layer": "parks"
        }
      }
    },
    { ... },
    { ... },
  ]
}
```

# API

## vtquery

**Parameters**

-   `tiles` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array).&lt;[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)>** an array of tile objects with `buffer`, `z`, `x`, and `y` values
-   `LngLat` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array).&lt;[Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)>** a query point of longitude and latitude to query, `[lng, lat]`
-   `options` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)=**
    -   `options.radius` **[Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)=** the radius to query for features. If your radius is larger than
        the extent of an individual tile, include multiple nearby buffers to collect a realstic list of features (optional, default `0`)
    -   `options.results` **[Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)=** the number of results/features returned from the query. (optional, default `5`)
    -   `options.layers` **[Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array).&lt;[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)>=** an array of layer string names to query from. Default is all layers.
    -   `options.geometry` **[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)=** only return features of a particular geometry type. Can be `point`, `linestring`, or `polygon`.
        Defaults to all geometry types.

**Examples**

```javascript
const vtquery = require('@mapbox/vtquery');

vtquery(tiles, lnglat, options, callback);
```

# Develop

```bash
git clone git@github.com:mapbox/vtquery.git
cd vtquery

# Build binaries. This looks to see if there were changes in the C++ code. This does not reinstall deps.
make

# Run tests
make test

# Cleans your current builds and removes potential cache
make clean

# Cleans everything, including the things you download from the network in order to compile (ex: npm packages).
# This is useful if you want to nuke everything and start from scratch.
# For example, it's super useful for making sure everything works for Travis, production, someone else's machine, etc
make distclean

# Generate API docs from index.js
# Requires documentation.js to be installed globally
npm install -g documentation
npm run docs
```

# Benchmarks

Benchmarks can be run with the bench/vtquery.bench.js script to test vtquery against common real-world fixtures (provided by mvt-fixtures). When making changes in a pull request, please provide the benchmarks from the master branch and the HEAD of your current branch.  You can control the `concurrency` and `iterations` of the benchmarks with the following command:

```
node bench/vtquery.bench.js --iterations 1000 --concurrency 5
```

And the output will show how many times the library was able to execute per second, per fixture:

```
1: 9 tiles, chicago, radius 1000 ... 191 runs/s (5240ms)
2: 9 tiles, chicago, only points ... 3257 runs/s (307ms)
3: mbx streets no radius ... 4049 runs/s (247ms)
4: mbx streets 2000 radius ... 240 runs/s (4159ms)
5: mbx streets only points ... 4673 runs/s (214ms)
6: mbx streets only linestrings ... 286 runs/s (3499ms)
7: mbx streets only polys ... 2801 runs/s (357ms)
8: complex multipolygon ... 1253 runs/s (798ms)
```

# Viz

The viz/ directory contains a small node application that is helpful for visual QA of vtquery results. It requests Mapbox Streets tiles and adds results as points to the map. In order to request tiles, you'll need a `MapboxAccessToken` environment variable.

```shell
cd viz
npm install
MapboxAccessToken={token} node app.js
# localhost:5000
```
