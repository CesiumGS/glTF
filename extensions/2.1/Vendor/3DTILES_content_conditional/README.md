# 3DTILES\_content\_conditional

## Contributors

- Marco Hutter, Cesium
- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [3DTILES_tileset](../3DTILES_tileset/README.md).

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension adds support for conditional content in 3D Tiles, by defining the following elements:

- A new content *type* that can be referred to via the content URI in a 3D Tiles data set.
- An *extension object* in the top-level tileset that describes the structure of the conditional content

In the context of this extension, 'conditional content' is tile content where the actual content data that is loaded and rendered depends on user-selectable criteria. In the context of this specification, the content data item that should be loaded and rendered is referred to as the '_active_' content data.

## TODO

To be updated from https://github.com/CesiumGS/3d-tiles/pull/834/