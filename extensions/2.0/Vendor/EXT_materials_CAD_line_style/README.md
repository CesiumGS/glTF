<!--
Copyright 2015-2025 The Khronos Group Inc.
SPDX-License-Identifier: CC-BY-4.0
-->

# EXT_materials_CAD_line_style

## Contributors

* Paul Connelly, Bentley Systems, [@pmconne](https://github.com/pmconne)

## Status

Complete

## Dependencies

Written against the glTF 2.0 spec.

## Overview

3D modeling and computer-aided drafting (CAD) environments like SketchUp, MicroStation, and Revit use lines extensively to annotate both two- and three-dimensional visualizations. Different lines are rendered with different styles to convey particular semantics and/or differing levels of emphasis. A CAD line style specifies the width of the line and a dash pattern. The EXT_materials_CAD_line_style augments a material with a description of a CAD line style.

## Example


[open question: device-pixel-ratio for width and pixel patterns?]

## glTF Schema Updates

The `EXT_materials_CAD_line_style` extension is applied to a material. If that material is used by any line-type primitive, or by the edges described by the `EXT_mesh_primitive_edge_visibility` extension, then it dictates the width and pattern with which the lines are to be drawn.

The `width` property specifies the length of 


## Implementation Notes

[tesselation generally required for width > 1 because many graphics APIs like WebGL no longer support wide line primitives]
[implementation may smooth out the tesselated lines by, e.g., inserting triangles at the joints]
[pattern should be continuous along the length of a line string]
[patterns could be aggregated into a texture]

## JSON Schema

- [EXT_mesh_primitive_restart.schema.json](schema/EXT_mesh_primitive_restart.schema.json)
- [primitiveGroup.schema.json](schema/primitiveGroup.schema.json)

