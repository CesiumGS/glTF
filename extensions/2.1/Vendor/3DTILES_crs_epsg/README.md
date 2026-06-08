# 3DTILES\_crs\_epsg

## Contributors

- Sean Lilley, Cesium
- Björn Blissing, Vantor

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [3DTILES_crs](../3DTILES_crs/README.md).

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension declares the Coordinate Reference System (CRS) in which a glTF 2.1 asset was authored as an [EPSG](https://epsg.org/home.html) identifier.

This extension **MUST** be used with [3DTILES_crs](../3DTILES_crs/README.md).

The following example shows an asset annotated to indicate a [WGS 84](https://epsg.org/ellipsoid_7030/WGS-84.html) Earth-centered, Earth-fixed (ECEF) geocentric coordinate reference system ([EPSG 4978](https://epsg.org/crs_4978/WGS-84.html)).

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "3DTILES_crs": {
      "type": "geocentric"
    },
    "3DTILES_crs_epsg": {
      "id": "EPSG:4978",
      "epoch": "2019.81"
    }
  }
}
```

The extension has the following properties:

- `id` is an EPSG identifier in form `EPSG:XXXX` or `EPSG:XXXX+YYYY` (in the case of a compound CRS)
- `epoch` - the coordinate epoch for coordinates that are referenced to a dynamic CRS such as WGS 84. Expressed as a decimal year (e.g. `"2019.81"`). See [WKT representation of coordinate epoch and coordinate metadata](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html#128) for more details.