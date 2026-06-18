<!--
SPDX-FileCopyrightText: 2026 2026 Bentley Systems, Incorporated

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

This extension specifies a well defined subset of glTF 2.1 for representing a subtree in [3DTILES_implicit_tiling](../3DTILES_implicit_tiling/README.md). A subtree stores tile, content, and child subtree availability, as well as metadata for available tiles and contents.

## File Extensions

Assets that use the `3DTILES_subtree` extension **SHOULD** use the `.subtree.gltf` or `.subtree.glb` file extensions. Though not required, this convention helps differentiate subtree files from tileset and content files.
