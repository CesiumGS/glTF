<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# EXT\_geospatial\_crs\_source

## Contributors

- Ronald Poirrier, Esri
- Jean-Philippe Pons, Esri
- Tamrat Belayneh, Esri
- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [EXT_geospatial_crs](../EXT_geospatial_crs/README.md).

## Optional

This extension is optional, meaning it should be placed in the glTF root's `extensionsUsed` list, but not in the `extensionsRequired` list.

## Overview

This extension allows an asset to specify its source coordinate reference system (CRS) and supply additional transformation matrices to convert the content from its local model space to world space coordinate within the specified coordinate system.

The content remains compatible with the coordinate reference system specified by [EXT_geospatial_crs](../EXT_geospatial_crs/README.md). For example, an asset may specify a projected coordinate system with `EXT_geospatial_crs_source` and a fallback geocentric coordinate system with `EXT_geospatial_crs`, as shown in the example below:

``` jsonc
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "EXT_geospatial_crs": {
      "format": "wkid",
      "extensions": {
        "EXT_geospatial_crs_wkid": {
          "authority": "EPSG",
          "wkid": 4978 // WGS-84 Earth-centered, Earth-fixed
        }
      }
    },
    "EXT_geospatial_crs_source": {
      "format": "wkid",
      "extensions": {
        "EXT_geospatial_crs_wkid": {
          "authority": "EPSG",
          "wkid": 31255,   // PCS MGI/Austria GK central
          "vcsWkid": 5778  // VCS GHA height Austria (Gebrauchshohen ADRIA)
        }
      }
    }
  }
}
```


```jsonc
{ // a node
  "boundingVolume": {...}, // ECEF bounding box (ignored in PCS mode)
  "matrix": [...], // ECEF transformation matrix (ignored in PCS mode)
  "mesh": 0,
  "extensions": {
    "EXT_geospatial_crs_source": {
      "boundingVolume": {...}, // PCS bounding box
      "matrix": {...}, // PCS transformation matrix
    }
  }
}
```

During traversal, the `EXT_geospatial_crs_source` transform may be used instead of the node transform to convert the content to projected coordinates. When the extension is not supported, the node transform is used and the asset falls back to geocentric coordinates.

## Properties

### Document-level extension

| Property | Type | Description |
| --- | --- | --- |
|format| string| The coordinate system format.<br><br> There are several standard formats used to define coordinate reference systems. This extension doesn't adhere to a specific format, instead defines a `"format"` property whose value **MUST** be defined by additional extensions:<br><br><ul><li>[EXT_geospatial_crs_wkid](../EXT_geospatial_crs_wkid/README.md) - Well-Known ID, commonly used to represent EPSG codes. Defines format `"wkid"`</li><li>[EXT_geospatial_crs_wkt2](../EXT_geospatial_crs_wkt2/README.md) - Well-Known Text version 2. Defines format `"wkt2"`</li></ul><br>Additional extensions may define additional formats.|

### Node extension

| Property | Type | Description |
| --- | --- | --- |       
|boundingVolume| object| Is identical to `node.boundingVolume`. |
|matrix|number[16]|Is identical to `node.matrix`.|
|translation|number[3]|Is identical to `node.translation`.|
|rotation|number[4]|Is identical to `node.rotation`.|
|scale|number[3]|Is identical to `node.scale`.|
