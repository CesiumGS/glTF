<!--
SPDX-FileCopyrightText: 2026 The Khronos Group Inc.

SPDX-License-Identifier: CC-BY-4.0
-->

# EXT\_crs\_enu

## Contributors

- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

## Optional vs. Required

This extension is optional, meaning it should be placed in the glTF root's `extensionsUsed` list, but not in the `extensionsRequired` list.

## Overview

This extension declares that the glTF is in a local right-handed Cartesian coordinate system in meters where `+X` faces east, `+Y` faces north, and `+Z` faces up.

This differs from the default glTF [coordinate system](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units) where `+X` faces left, `+Y` faces up, and `+Z` faces forward.

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "EXT_crs_enu": {}
  }
}
```
