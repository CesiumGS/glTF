<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# 3DTILES\_subtree

## Contributors

- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [3DTILES_implicit_tiling](../3DTILES_implicit_tiling/README.md).

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension specifies a well defined subset of glTF 2.1 for representing a subtree in [3DTILES_implicit_tiling](../3DTILES_implicit_tiling/README.md). A subtree stores tile, content, and child subtree availability, as well as metadata for available tiles and contents. A subtree is not intended for rendering and **SHOULD NOT** have any scenes.

## File Extensions

Assets that use the `3DTILES_subtree` extension **SHOULD** use the `.subtree.gltf` or `.subtree.glb` file extensions. Though not required, this convention helps differentiate subtree files from tileset and content files.

## Availability

Tile availability (`tileAvailability`) and child subtree availability (`childSubtreeAvailability`) **MUST** always be provided for a subtree.

Content availability (`contentAvailability`) **MUST** be provided if the implicit root tile has a `contentUri` property, otherwise it **MUST** be omitted.

Availability may be represented either as an array of values stored in an accessor or as a constant value.

* `values` is an integer index that identifies the accessor containing the availability values. Accessor values **MUST** be either `1` indicating that the element is available or `0` indicating that the element is unavailable. The accessor `type` **MUST** be `SCALAR` and `componentType` must be `5121` (UNSIGNED_BYTE). `availableCount` is an integer indicating how many elements in the accessor are available.
* `constant` is an integer indicating whether all of the elements are available (`1`) or all are unavailable (`0`).


> **Example:** The JSON description of a subtree where each tile is available, but not all tiles have content, and not all child subtrees are available:
>
> ```json
> {
>   "extensionsUsed": ["3DTILES_subtree"],
>   "extensionsRequired": ["3DTILES_subtree"],
>   "asset": {
>     "version": "2.1"
>   },
>   "buffers": [
>     {
>       "uri": "availability.bin",
>       "byteLength": 344
>     }
>   ],
>   "bufferViews": [
>     {
>       "buffer": 0,
>       "byteOffset": 0,
>       "byteLength": 85
>     },
>     {
>       "buffer": 0,
>       "byteOffset": 88,
>       "byteLength": 256
>     }
>   ],
>   "accessors": [
>     {
>       "type": "SCALAR",
>       "componentType": 5121,
>       "count": 85,
>       "bufferView": 0
>     },
>     {
>       "type": "SCALAR",
>       "componentType": 5121,
>       "count": 256,
>       "bufferView": 1
>     }
>   ],
>   "extensions": {
>     "3DTILES_subtree": {
>       "tileAvailability": {
>         "constant": 1,
>       },
>       "contentAvailability": [{
>         "values": 0,
>         "availableCount": 60
>       }],
>       "childSubtreeAvailability": {
>         "values": 1
>       }
>     }
>   }
> }
> ```

## Tile Semantics

Tile properties such as bounding volume, refinement type, and geometric error that are computed automatically based on [Subdivision Rules](../3DTILES_implicit_tiling/README.md#subdivision-rules) but may be overridden by values stored in the subtree.

In particular, `tileBoundingBox`, `tileBoundingSphere`, `tileBoundingRegion`, `tileMinimumHeight`, and `tileMaximumHeight` semantics may define a tighter bounding volume for a tile than is implicitly calculated from [Implicit Tiling](../3DTILES_implicit_tiling/README.md). If more than one of these semantics are available for a tile, clients may select the most appropriate option based on use case and performance requirements.

Property name|Type|Component Type|Description
--|--|--|--
`tileBoundingBox`|`MAT4`|`5130` (DOUBLE)|The bounding box of the tile, expressed as a column-major transformation matrix applied to a unit cube where the top-left 3x3 matrix defines the box's rotation and scale and the fourth column defines the box's translation.
`tileBoundingSphere`|`VEC4`|`5130` (DOUBLE)|The bounding sphere of the tile, where the first three elements define the x, y, and z values for the center of the sphere and the the last element defines the radius.
`tileBoundingRegion`|`VEC4`|`5130` (DOUBLE)|The bounding region of the tile, where the values are stored in west, south, east, north order in degrees. See [3DTILES_shape_ellipsoid_region](../3DTILES_shape_ellipsoid_region/README.md).
`tileMinimumHeight`|`SCALAR`|`5130` (DOUBLE)|The minimum height of the tile above or below the ellipsoid. Equivalent to the minimum height component of [3DTILES_shape_ellipsoid_region](../3DTILES_shape_ellipsoid_region/README.md) and [3DTILES_shape_s2](../3DTILES_shape_s2/README.md).
`tileMaximumHeight`|`SCALAR`|`5130` (DOUBLE)|The maximum height of the tile above or below the ellipsoid. Equivalent to the maximum height component of [3DTILES_shape_ellipsoid_region](../3DTILES_shape_ellipsoid_region/README.md) and [3DTILES_shape_s2](../3DTILES_shape_s2/README.md).
`tileHorizonOcclusionPoint`<sup>1</sup>|`VEC3`|`5130` (DOUBLE)|The horizon occlusion point of the tile expressed in an ellipsoid-scaled fixed frame. If this point is below the horizon, the entire tile is below the horizon. See [Horizon Culling](https://cesium.com/blog/2013/04/25/horizon-culling/) for more information.
`tileGeometricError`|`SCALAR`|`5130` (DOUBLE)|The geometric error of the tile.
`tileRefine`|`SCALAR`|`5121` (UNSIGNED_BYTE)|The tile refinement type. Valid values are `0` (`"ADD"`) and `1` (`"REPLACE"`).
`tileTransform`|`MAT4`|`5130` (DOUBLE)|The tile transform. Equivalent to `node.matrix`.

<sup>1</sup> `tileHorizonOcclusionPoint` should account for all content in a tile and its descendants, whereas `contentHorizonOcclusionPoint` should only account for content in a tile. When the two values are equivalent, only `tileHorizonOcclusionPoint` should be specified.

## Content Semantics

Property name|Type|Component Type|Description
--|--|--|--
`contentBoundingBox`|`MAT4`|`5130` (DOUBLE)|The bounding box of the content, expressed as a column-major transformation matrix applied to a unit cube where the top-left 3x3 matrix defines the box's rotation and scale and the fourth column defines the box's translation.
`contentBoundingSphere`|`VEC4`|`5130` (DOUBLE)|The bounding sphere of the content, where the first three elements define the x, y, and z values for the center of the sphere and the the last element defines the radius.
`contentBoundingRegion`|`VEC4`|`5130` (DOUBLE)|The bounding region of the content, where the values are stored in west, south, east, north order in degrees. See [3DTILES_shape_ellipsoid_region](../3DTILES_shape_ellipsoid_region/README.md).
`contentMinimumHeight`|`SCALAR`|`5130` (DOUBLE)|The minimum height of the content above or below the ellipsoid. Equivalent to the minimum height component of [3DTILES_shape_ellipsoid_region](../3DTILES_shape_ellipsoid_region/README.md) and [3DTILES_shape_s2](../3DTILES_shape_s2/README.md).
`contentMaximumHeight`|`SCALAR`|`5130` (DOUBLE)|The maximum height of the content above or below the ellipsoid. Equivalent to the maximum height component of [3DTILES_shape_ellipsoid_region](../3DTILES_shape_ellipsoid_region/README.md) and [3DTILES_shape_s2](../3DTILES_shape_s2/README.md).
`contentHorizonOcclusionPoint`<sup>1</sup>|`VEC3`|`5130` (DOUBLE)|The horizon occlusion point of the content expressed in an ellipsoid-scaled fixed frame. If this point is below the horizon, the entire content is below the horizon. See [Horizon Culling](https://cesium.com/blog/2013/04/25/horizon-culling/) for more information.
`contentLayer`|`SCALAR`|`5121` (UNSIGNED_BYTE), `5123` (UNSIGNED_SHORT), `5125` (UNSIGNED_INT)| The content layer. See [3DTILES_layers](../3DTILES_layers/README.md).

## Metadata

Subtrees may store metadata for available tiles and content.

* `tileMetadata` is an index of a property table in [`EXT_structural_metadata`](https://github.com/CesiumGS/glTF/tree/3d-tiles-next/extensions/2.0/Vendor/EXT_structural_metadata#property-tables) containing metadata for available tiles.
* `contentMetadata` is a property table in [`EXT_structural_metadata`](https://github.com/CesiumGS/glTF/tree/3d-tiles-next/extensions/2.0/Vendor/EXT_structural_metadata#property-tables) containing metadata for available content. If the implicit root tile does not have a `contentUri` property, then none of the implicit tiles has any content, and the `contentMetadata` property **MUST** be omitted.

> **Example:** The same JSON description of a subtree extended with tile and content metadata. The subtree JSON refers to a class ID in the root tileset schema. Tile and content metadata are stored in [property tables](https://github.com/CesiumGS/glTF/tree/3d-tiles-next/extensions/2.0/Vendor/EXT_structural_metadata#property-tables).
>
>
> <em>Schema in the root tileset</em>
> ```json
> {
>   "schema": {
>     "classes": {
>       "tile": {
>         "properties": {
>           "center": {
>             "type": "VEC3",
>             "componentType": "FLOAT64",
>           },
>           "countries": {
>             "description": "Countries a tile intersects",
>             "type": "STRING",
>             "array": true
>           }
>         }
>       },
>       "content": {
>         "properties": {
>           "credits": {
>             "type": "SCALAR",
>             "componentType": "UINT16",
>             "array": true
>           },
>           "minTemperature": {
>             "type": "SCALAR",
>             "componentType": "FLOAT64"
>           },
>           "maxTemperature": {
>             "type": "SCALAR",
>             "componentType": "FLOAT64"
>           },
>           "triangleCount": {
>             "type": "SCALAR",
>             "componentType": "UINT32"
>           }
>         }
>       }
>     }
>   }
> }
> ```
> 
> <em>Subtree JSON</em>
> 
> ```json
> {
>   "extensionsUsed": ["3DTILES_subtree", "EXT_structural_metadata"],
>   "extensionsRequired": ["3DTILES_subtree", "EXT_structural_metadata"],
>   "asset": {
>     "version": "2.1"
>   },
>   "buffers": [
>     {
>       "name": "Availability Buffer",
>       "uri": "availability.bin",
>       "byteLength": 344
>     },
>     {
>       "name": "Metadata Buffer",
>       "uri": "metadata.bin",
>       "byteLength": 6512
>     }
>   ],
>   "bufferViews": [
>     { "buffer": 0, "byteOffset": 0, "byteLength": 85 },
>     { "buffer": 0, "byteOffset": 88, "byteLength": 256 },
>     { "buffer": 1, "byteOffset": 0, "byteLength": 2040 },
>     { "buffer": 1, "byteOffset": 2040, "byteLength": 1530 },
>     { "buffer": 1, "byteOffset": 3576, "byteLength": 344 },
>     { "buffer": 1, "byteOffset": 3920, "byteLength": 1024 },
>     { "buffer": 1, "byteOffset": 4944, "byteLength": 240 },
>     { "buffer": 1, "byteOffset": 5184, "byteLength": 122 },
>     { "buffer": 1, "byteOffset": 5312, "byteLength": 480 },
>     { "buffer": 1, "byteOffset": 5792, "byteLength": 480 },
>     { "buffer": 1, "byteOffset": 6272, "byteLength": 240 }
>   ],
>   "accessors": [
>     {
>       "type": "SCALAR",
>       "componentType": 5121,
>       "count": 85,
>       "bufferView": 0
>     },
>     {
>       "type": "SCALAR",
>       "componentType": 5121,
>       "count": 256,
>       "bufferView": 1
>     }
>   ],
>   "extensions": {
>     "3DTILES_subtree": {
>       "tileAvailability": {
>         "constant": 1,
>       },
>       "contentAvailability": [{
>         "values": 0,
>         "availableCount": 60
>       }],
>       "childSubtreeAvailability": {
>         "values": 1
>       }
>     },
>     "EXT_structural_metadata": {
>       "propertyTables": [
>         {
>           "class": "tile",
>           "count": 85,
>           "properties": {
>             "horizonOcclusionPoint": {
>               "values": 2
>             },
>             "countries": {
>               "values": 3,
>               "arrayOffsets": 4,
>               "stringOffsets": 5,
>               "arrayOffsetType": "UINT32",
>               "stringOffsetType": "UINT32"
>             }
>           }
>         },
>         {
>           "class": "content",
>           "count": 60,
>           "properties": {
>             "attributionIds": {
>               "values": 6,
>               "arrayOffsets": 7,
>               "arrayOffsetType": "UINT16"
>             },
>             "minimumHeight": {
>               "values": 8
>             },
>             "maximumHeight": {
>               "values": 9
>             },
>             "triangleCount": {
>               "values": 10,
>               "min": 520,
>               "max": 31902
>             }
>           }
>         }
>       ]
>     }
>   }
> }
> ```
