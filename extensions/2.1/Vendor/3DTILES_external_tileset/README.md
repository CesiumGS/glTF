# 3DTILES\_external\_tileset

## Contributors

- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [3DTILES_tileset](../3DTILES_tileset/README.md).

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension allows a tileset to reference external tilesets (other assets with the `3DTILES_tileset` extension). This enables, for example, storing each city in a tileset and then having a global tileset of tilesets. It is also helpful for partitioning large tilesets into multiple smaller tilesets to improve initial load time.

## TODO

- How to indicate that `externalAsset` is an external tileset before loading it?
