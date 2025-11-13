<!--
Copyright 2015-2025 The Khronos Group Inc.
SPDX-License-Identifier: CC-BY-4.0
-->

# BENTLEY_material_line_style

## Contributors

* Paul Connelly, Bentley Systems, [@pmconne](https://github.com/pmconne)
* Daniel Zhong, Bentley Systems, [@danielzhong](https://github.com/danielzhong)

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec.

## Overview

Lines are fundamental elements in many 3D modeling and computer-aided design (CAD) environments. They are used to annotate two- and three-dimensional visualizations, with variations in width and dash patterns conveying semantic meaning or emphasis.  

The `BENTLEY_material_line_style` extension defines a method for describing the visual style of lines within glTF material. It enables authors to specify line thickness and a repeating dash pattern.  

This extension introduces two primary properties controlling line appearance:

- `width`: the pixel width of the rendered line  
- `pattern`: a string of arbitrary length representing a repeating sequence of lit (`-`) and unlit (` `) pixels

## Specifying Line Styles

The `BENTLEY_material_line_style` extension is applied to a material.  
When that material is used by any line-type primitive, or by the edges described by the `EXT_mesh_primitive_edge_visibility` extension, it defines the **width** and **pattern** with which those lines are rendered.

### Width

The `width` property specifies the line’s thickness **in screen pixels**.  
The value of `width` must be greater than `0`.  

For each line segment, implementations should extrude geometry by half this width on each side of the segment’s centerline, perpendicular to its direction. Implementations should also insert triangles at the joints between line segments comprising a continuous line string or line loop, when appropriate to visually round out the joints.


### Pattern

The `pattern` property specifies a **repeating on/off pixel sequence** to be applied along the length of the line.  
It is expressed as a string of arbitrary length, where each character corresponds to one pixel:

- `-` (hyphen): lit pixel (on)  
- ` ` (space): unlit pixel (off)

The pattern repeats cyclically once the end of the string is reached.  
The first character determines whether the first pixel drawn is lit or unlit.

Example patterns:

| Pattern | Description |
|----------|--------------|
| `---` | solid line |
| `- -` | dotted line |
| `--  ` | dashed line |
| `- - --  -` | custom pattern |

The pattern is applied continuously along each continuous line string or loop.

## Implementation Notes

Because many graphics APIs do not support line primitives with a width larger than 1, tessellation is generally required to draw wide lines. Implementations may smooth out the tesselated lines by, e.g., inserting additional triangles at the joints to round them.

The pattern should be continuous along the length of each continuous line string or line loop.

Implementations may choose to aggregate the set of unique dash patterns into a texture with one row per pattern to be indexed by the shader program when applying a line style's pattern.

## JSON Schema

* [material.BENTLEY_material_line_style.schema.json](schema/material.BENTLEY_material_line_style.schema.json)