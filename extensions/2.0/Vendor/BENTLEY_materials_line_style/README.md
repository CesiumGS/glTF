<!--
Copyright 2025 Bentley Systems, Incorporated
SPDX-License-Identifier: CC-BY-4.0
-->

# BENTLEY_materials_line_style

## Contributors

* Paul Connelly, Bentley Systems, [@pmconne](https://github.com/pmconne)
* Daniel Zhong, Bentley Systems, [@danielzhong](https://github.com/danielzhong)
* Mark Schlosser, Bentley Systems, [@markschlosseratbentley](https://github.com/markschlosseratbentley)

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec.

## Overview

Lines are fundamental elements in many 3D modeling and computer-aided design (CAD) environments. They are used to annotate two- and three-dimensional visualizations, with variations in width and dash patterns conveying semantic meaning or emphasis.  

The `BENTLEY_materials_line_style` extension defines a method for describing the visual style of lines within glTF material. It enables authors to specify line thickness and a repeating dash pattern.  

This extension introduces two primary properties controlling line appearance:

- `width`: the pixel width of the rendered line  
- `pattern`: an unsigned integer whose bits represent a repeating on/off pattern along the line

## Specifying Line Styles

The `BENTLEY_materials_line_style` extension is applied to a material. When that material is used by any line-type primitive, or by the edges described by the [`EXT_mesh_primitive_edge_visibility`](https://github.com/KhronosGroup/glTF/pull/2479) extension, it defines the **width** and **pattern** with which those lines are rendered.

### Width

The `width` property specifies the line’s thickness **in screen pixels**. The value of `width` must be greater than `0`.

For each line segment, implementations should extrude geometry by half this width on each side of the segment’s centerline, perpendicular to its direction. Implementations should also insert triangles at the joints between line segments comprising a continuous line string or line loop, when appropriate to visually round out the joints.

### Pattern

The `pattern` property specifies a **repeating on/off bit pattern** applied along the length of the line. Each bit represents one segment unit of equal length:

- Bit value `1`: lit (on)  
- Bit value `0`: unlit (off)

The least significant bit (bit 0) corresponds to the start of the pattern and represents the first drawn segment along the line. The pattern repeats cyclically once all bits have been consumed.  

Example patterns:

| Bit Pattern (hex) | Binary | Description     |
|--------------------|---------|----------------|
| `0xFFFF`          | 1111111111111111 | solid line |
| `0xAAAA`          | 1010101010101010 | dotted line |
| `0xF0F0`          | 1111000011110000 | dashed line |
| `0xC3C3`          | 1100001111000011 | custom pattern |

## Implementation Notes

Because many graphics APIs do not support line primitives with a width larger than 1, tessellation is generally required to draw wide lines. Implementations may smooth out the tessellated lines by, for example, inserting additional triangles at the joints to round them.

The pattern should be applied continuously along the length of each continuous line string or line loop, ensuring visual consistency across connected segments.

When rendering, the integer `pattern` value may be supplied to the shader as a uniform or read from a small lookup table shared among materials. Each bit of the integer defines whether a given unit segment along the line is drawn or skipped.

For now, implementations will be limited to 16-bit integers for pattern encoding, as this is sufficient for all current use cases. 
If a pattern uses fewer bits than the maximum supported width, unused higher bits must be set to zero.


## JSON Schema

* [materials.BENTLEY_material_line_style.schema.json](schema/materials.BENTLEY_material_line_style.schema.json)