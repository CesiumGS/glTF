# 3DTILES\_georeference

## Contributors

- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension georeferences a node to the provided geographic coordinates.

```json
{
  "extensions": {
    "3DTILES_georeference": {
      "longitude": -75.15836368768382,
      "latitude": 39.95090650840344,
      "height": -21.668226434267066
    }
  },
  "mesh": 0
}
```

`longitude` and `latitude` are specified in degrees. `height` above (or below) the ellipsoid is specified in meters.

This extension uses WGS84 ([EPSG:4979](https://epsg.org/crs_4979/WGS-84.html)) as the default coordinate reference system. A different coordinate reference system may be specified with [`3DTILES_crs`](../3DTILES_crs/README.md) in which case the longitude, latitude, and height values are now geographic coordinates on the provided ellipsoid instead of the WGS84 ellipsoid.

The glTF axis mapping is as follows:

- The `+z` axis (forward) faces east 
- The `+x` axis (left) faces north 
- The `+y` axis (up) faces up 

When [`3DTILES_crs`](../3DTILES_crs/README.md) is used:

- The `+x` axis faces east
- The `+y` axis faces north
- The `+z` axis faces up

<p align="center">
  <img src="./figures/enu-xyz.png"/><br/>
</p>

## Transformation Order

The georeference transform is applied **after** the node transform.

In this example, the georeference transform is applied after the local 45° heading.

<table>
  <tr>
    <td><pre><code>{
  "extensions": {
    "3DTILES_georeference": {
      "longitude": -75.15836368768382,
      "latitude": 39.95090650840344,
      "height": -21.668226434267066
    }
  },
  "rotation": [0, -0.38268, 0, 0.923879],
  "mesh": 0,
}</code></pre></td>
    <td><img src="./figures/plane.jpg"/></td>
  </tr>
</table>

## Appendix

For computing the transformation matrix from local ENU coordinates to geocentric coordinates see [`Cesium.Transforms.eastNorthUpToFixedFrame`](Cesium.Transforms.eastNorthUpToFixedFrame).
