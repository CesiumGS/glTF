<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# 3DTILES\_viewer\_request\_volume

## Contributors

- Sean Lilley, Cesium
- Adam Morris, Cesium
- Jeshurun Hembd, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

TODO: does this depend on `3DTILES_tileset`? That is the envisioned use case, but it could in theory be useful more broadly.

## Optional vs. Required

This extension is always optional. It should be placed in the tileset JSON `extensionsUsed` list, but not in the `extensionsRequired` list.

## Overview

This extension describes an alternate bounding volume for use in loading and rendering decisions. The alternate bounding volume
defines a volume of viewer positions around the node. If the viewer is within that volume, the node's content is assumed to be
relevant for rendering.

In datasets using the [3DTILES_tileset extension](../3DTILES_tileset/README.md), where
[geometric error](../3DTILES_tileset/README.md#geometric-error) is inadequate to define loading behavior
relative to the camera position and the other data in the scene, this extension can be used to trigger
loading and rendering of a node whenever the viewer is inside a given volume.

## Defining a viewer request volume for a node

The viewer request volume is specified similarly to bounding volumes. The volume shape type **MUST** be `"box"` or `"sphere"` unless additional shape types are enabled through extensions.

A list of extensions that enable additional shape types:

- [3DTILES_shape_ellipsoid_region](../3DTILES_shape_ellipsoid_region/README.md)
- [3DTILES_shape_cylinder_region](../3DTILES_shape_cylinder_region/README.md)
- [3DTILES_shape_s2](../3DTILES_shape_s2/README.md)

## Example

The following example shows a point cloud inside a building. The point cloud tile's `boundingVolume` is a sphere with a radius of `1.25`. It also has a larger sphere with a radius of `15` for the `viewerRequestVolume`. Since the `geometricError` is zero, the point cloud tile's content is always rendered (and initially requested) when the viewer is inside the large sphere defined in the `3DTILES_viewer_request_volume` extension.

```json
{
  "files": [
    {
      "mimeType": "model/gltf-binary",
      "uri": "building.glb"
    },
    {
      "mimeType": "model/gltf-binary",
      "uri": "points.glb"
    }
  ],
  "externalAssets": [
    {
      "file": 0
    },
    {
      "file": 1
    }
  ],
  "shapes": [
    {
      "type": "box",
      "box": {
        "size": [7.476, 7.44, 26.804]
      }
    },
    {
      "type": "sphere",
      "sphere": {
        "radius": 1.25
      }
    },
    {
      "type": "sphere",
      "sphere": {
        "radius": 15
      }
    }
  ],
  "nodes": [
    {
      "extensions": {
        "3DTILES_tileset": {
          "geometricError": 32.0,
          "refine": "ADD"
        }
      },
      "boundingVolume": {
        "shape": 0
      },
      "externalAsset": 0,
      "children": [1]
    },
    {
      "extensions": {
        "3DTILES_tileset": {
          "geometricError": 0.0
        },
        "3DTILES_viewer_request_volume": {
          "shape": 2
        }
      },
      "boundingVolume": {
        "shape": 1
      },
      "externalAsset": 1
    }
  ]
}
```

For more on request volumes, see the https://github.com/CesiumGS/3d-tiles-samples/tree/main/1.0/TilesetWithRequestVolume[sample tileset].
