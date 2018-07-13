# geojson-spec
This repository contains a website that presents reformatted HTML version of the official GeoJSON spec (RFC 7946).

The text below is a work in progress, simplified, extra-readable version of the spec. It will be somehow 
integrated into the rest of the repo at some point, to offer two versions of the spec: a human-readable TLDR version (below) 
and the more formal version, still formatted for easier reading than the official RFC website.

## Introduction

GeoJSON is an easy-to-work with format for geospatial data. This document is an unofficial summary of the [complete RFC 7946 GeoJSON specification](http://geojson.win/), made by [Steve Bennett](https://hire.stevebennett.me).

### GeoJSON file

A GeoJSON file is a JSON file which contains exactly one GeoJSON object, which is most commonly (but need not be) a FeatureCollection.

All capitalisation MUST match the forms given here.

### GeoJSON objects

There are four classes of GeoJSON object: six geometry types (`Point`, `LineString`, `Polygon`, `MultiPoint`, `MultiLineString`, `MultiPolygon`), `Feature`, `GeometryCollection`, `FeatureCollection`.

There is one other kind of object: bbox.

#### bbox

A "bbox" defines a bounding box for a GeoJSON object, defined as: `[west, south, east, north]` or `[west, south, low, east, north, high]`. 

If the bounding box spans the 180 line of longitude, "west" will be numerically greater than "east".

#### Geometry

Represents a location.

```js
{
    // REQUIRED, one of: Point, LineString, Polygon, MultiPoint, MultiLineString, MultiPolygon.
    "type": "Point",

    // REQUIRED: array of coordinates whose structure depends on the "type":
    "coordinates": [144.9, -37.8],

    // OPTIONAL: bbox object
    "bbox": [144.9, -37.8, 144.9, -37.8],

    // OPTIONAL: additional attributes, which MUST NOT include "features" or "properties"
}
```

The `coordinates` array is composed ultimately of Position arrays, nested to varying depths.

A Position is either `[longitude, latitude]` OR `[longitude, latitude, elevation]`, where:
 
* `longitude` and `latitude` are numbers, in degrees, defining a location on the WGS84 reference ellipsoid.
* `elevation` is a number of metres above (or below) the WGS84 reference ellipsoid. (If absent, assumed to be 0.)

The structure of the `coordinates` array depends on the geometry type:

* `Point` (depth 1, length 2 or 3): `Position`
* `MultiPoint` (depth 2, length 0+): `[Point, Point, ...]`
* `LineString` (depth 2, length 2+): `[Point, Point, ...]`
* `MultiLineString` (depth 3, length 0+): `[LineString, LineString, ...]`
* `Polygon` (depth 3, length 1+): `[LineString, LineString, ...]`.
  * Each `LineString` array MUST have length 4+, and its last element MUST be identical to its first. 
  * The first LineString represents the exterior boundary, traced counterclockwise. The rest are holes inside the polygon, traced clockwise.
* `MultiPolygon` (depth 4, length 0+): `[Polygon, Polygon, ...]`

A LineString or Polygon SHOULD NOT cross the 180 line of longitude. Instead, it should be split into two or more pieces and represented with a MultiLineString or MultiPolygon instead.

#### GeometryCollection

Represents a location composed of heterogeneous geometry types, such as a line combined with a point. Contains an ordered array of Geometry objects. 

A GeometryCollection SHOULD NOT contain other GeometryCollections. 

A GeometryCollection SHOULD NOT be used if a different Geometry object (such as a MultiPolygon) could be used instead.

```js
{
    // REQUIRED: must contain "GeometryCollection"
    "type": "GeometryCollection",

    // REQUIRED: array of 0+ Geometry objects.
    "geometries": [...],

   // OPTIONAL: bbox object
   "bbox": [...],

   // OPTIONAL: additional attributes.
}
```


#### Feature

Represents a thing that has a location. Contains a Geometry or GeometryCollection, and additional properties.

```js
{
   // REQUIRED: must contain "Feature"
   "type": "Feature",

   // REQUIRED: Geometry object OR GeometryCollection object OR null
   "geometry": { ... }

   // REQUIRED: properties object. May contain anything, with no special meaning to any properties.
   "properties": { ... },

   // OPTIONAL: bbox object
   "bbox": [...],

   // OPTIONAL: identifier (string or number). If the feature has a commonly used identifier, it SHOULD be included here
   "id": 10,

   // OPTIONAL: additional attributes, which MUST NOT include "features" or "geometries"
}
```

#### FeatureCollection

Represents a collection of things that have locations. Contains an ordered array of Feature objects.

```js
{
    // REQUIRED: must contain "FeatureCollection"
    "type": "FeatureCollection",

    // REQUIRED: array of 0+ Feature objects.
    "features": [...],

   // OPTIONAL: bbox object
   "bbox": [...],

   // OPTIONAL: additional attributes, which MUST NOT include "coordinates" or "geometry".
}
```

