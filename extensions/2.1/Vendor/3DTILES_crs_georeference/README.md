# 3DTILES\_crs\_georeference

## Contributors

- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [3DTILES_crs_geocentric](../3DTILES_crs_geocentric/README.md).

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension transforms the root node of a tileset from an East-North-Up (ENU) coordinate system to a geocentric coordinate system based on the provided geographic coordinates.

```json
{
    "extensions": {
        "EXT_crs_georeference": {
            "longitude": -75.15836368768382,
            "latitude": 39.95090650840344,
            "height": -21.668226434267066
        }
    }
}
```
