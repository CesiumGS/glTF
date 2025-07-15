<!--
Copyright 2015-2025 The Khronos Group Inc.
SPDX-License-Identifier: CC-BY-4.0
-->

# BENTLEY_materials_line_style

## Contributors

* Paul Connelly, Bentley Systems, [@pmconne](https://github.com/pmconne)

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec.

## Overview

3D modeling and computer-aided drafting (CAD) environments like SketchUp, MicroStation, and Revit use lines extensively to annotate both two- and three-dimensional visualizations. Different lines are rendered with different styles to convey particular semantics and/or differing levels of emphasis. A CAD line style specifies the width of the line and a dash pattern. The BENTLEY_materials_line_style augments a material with a description of a CAD line style.

This specification describes a minimal extension sufficient to meet Bentley Systems' requirements, with some suggestions for how it might be broadened and/or generalized while retaining a focus on CAD visualization.

## Specifying Line Styles

The `BENTLEY_materials_line_style` extension is applied to a material. If that material is used by any line-type primitive, or by the edges described by the `EXT_mesh_primitive_edge_visibility` extension, then it dictates the width and pattern with which the lines are to be drawn.

The `width` property specifies the width in pixels of the lines. For each line segment, extrude by half this width perpendicularly on both sides.

> Potential generalization: permit width to be specified in meters.
> 
> Potential generalization: permit pixel width to scale with distance between a minimum and maximum.
> 
> Potential generalization: we currently assume lines face the camera, which is sensible when using pixels as units of measure. But the glTF spec permits for lines to have normal and tangent attributes for purposes of lighting; this could also be used to specify the plane in which they should be extruded.

The `pattern` property specifies a pixel pattern to be repeated along the length of the line. It is an arbitrary length bitfield expressed as a string in which a `-` (hyphen) indicates a lit pixel and a ` ` (space) indicates an unlit pixel. The first character in the string determines whether the first pixel drawn by the line is lit or not, and so on until the last character, at which point the pattern repeats from the first character.

> Potential generalization: permit patterns to be specified in meters, such that each character in the bitfield correspond to some specified number of meters.

> TBD: do pixel widths and patterns refer to device pixels or screen pixels?

## glTF Schema Updates



## Implementation Notes

Because many graphics APIs do not support line primitives with a width larger than 1, tesselation is generally required to draw wide lines. Implementations may smooth out the tesselated lines by, e.g., inserting additional triangles at the joints to round them.

> Potential generalization: permit the join style (e.g., round, miter, bevel, diamond) and end cap style (butt, round, diamond, square) to be specified.

The pattern should be continuous along the length of each continuous line string or line loop.

Implementations may choose to aggregate the set of unique dash patterns into a texture with one row per pattern to be indexed by the shader program when applying a line style's pattern.

## JSON Schema


