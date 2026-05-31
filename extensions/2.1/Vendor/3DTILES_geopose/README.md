# 3DTILES\_geopose

## Contributors

- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

GeoPose 1.0 is an OGC Implementation Standard for exchanging the location and orientation of real or virtual geometric objects (_Poses_) within reference frames anchored to the earth’s surface (_Geo_) or within other astronomical coordinate systems.

This extension implements [Standardization Target 1: Basic-YPR](https://docs.ogc.org/is/21-056r11/21-056r11.html#toc39).

```json
{
  "extensions": {
    "3DTILES_crs_geopose": {
      "position": {
        "lat": 47.7,
        "lon": -122.3,
        "h": 11.5
      },
      "angles": {
        "yaw": 5.514456741060452,
        "pitch": -0.43610515937237904,
        "roll": 0.0
      }
    }
  }
}
```

This extension uses WGS84 (EPSG:4979) as the coordinate reference system for specifying the position with longitude and latitude specified in degrees. Height above (or below) the ellipsoid must be specified in meters. The yaw-pitch-roll is provided as a rotation-only transform from a WGS84 referenced local tangent plane East-North-Up coordinate system.

## TODO

- How does this relate to `3DTILES_crs`? Is z-up assumed?
- For implementation guidance, link to `Cesium.Transforms.eastNorthUpToFixedFrame`.