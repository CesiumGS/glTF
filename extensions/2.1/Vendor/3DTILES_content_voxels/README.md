<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated
SPDX-FileCopyrightText: 2026, 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# 3DTILES\_content\_voxels

## Contributors

- Sean Lilley, Cesium
- Ian Lilley, Cesium
- Janine Liu, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [3DTILES_tileset](../3DTILES_tileset/README.md).

## Optional vs. Required

This extension is always optional. It should be placed in the tileset JSON `extensionsUsed` list, but not in the `extensionsRequired` list.

## Overview

This extension indicates the presence of voxel content and associates it with metadata definitions in the tileset's schema. Voxels are stored as glTFs with the [`EXT_primitive_voxels`](https://github.com/CesiumGS/glTF/tree/3d-tiles-next/extensions/2.0/Vendor/EXT_primitive_voxels) extension and are typically paired with [`EXT_structural_metadata`](https://github.com/CesiumGS/glTF/tree/3d-tiles-next/extensions/2.0/Vendor/EXT_structural_metadata) to unify the schema between a tileset and its tiles.

This extension is often paired with [Implicit Tiling](../3DTILES_implicit_tiling/README.md) for efficient representation of massive sparse voxel datasets. Although rendering implementations may vary, this extension can let runtimes detect voxel content in advance, such that they can allocate the necessary resources before any tiles load.

## TODO

To be updated from https://github.com/CesiumGS/3d-tiles/tree/voxels/extensions/3DTILES_content_voxels