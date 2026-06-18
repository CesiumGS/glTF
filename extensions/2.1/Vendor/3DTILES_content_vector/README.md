<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated
SPDX-FileCopyrightText: 2026, 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# 3DTILES\_content\_vector

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

Applied to geospatial domains, the term “vector data” refers to geometric topologies: points in 2D or 3D coordinate systems; polylines connecting a series of points; or polygons having a closed, exterior loop of points, optionally with additional interior loops defining holes in the polygon. The term "vector data" exists in contrast to “raster data”, which stores pixel grids in image-like formats, rather than discrete geometric types.

Extending the concept of vector data into the domain of graphics APIs, and of 3D scenes already composed of graphics primitives — such as points, lines, triangles — this extension proposes and defines an additional distinction: “vector data” comprises point, polyline, and polygon geometries with intrinsic semantic meaning, but without intrinsic rendering intent.

> [!NOTE]
> Unlike typical glTF geometry, vector data geometry does not directly convey rendering intent. Vector data is a vehicle for topology or for associated properties. Points may be aggregated, clustered, or used as anchors for labels. Lines may be widened, dashed, or used as an invisible track for animation. Polygons may be outlined, extruded, subtracted from existing scene geometry, or used to define an abstract area of analysis.

Working from the definition above, this extension proposes a mechanism for encoding vector point, line, and polygon data in 3D Tiles using glTF content, and for distinguishing such vector data from general glTF content. The extension further refines tile and content bounding volumes as needed to support seamless tiled vector rendering in a variety of visual styles.

## TODO

To be updated from https://github.com/CesiumGS/3d-tiles/pull/838