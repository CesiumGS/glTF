<!--
Copyright 2026 Bentley Systems, Incorporated
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
The extension is also applied to line-type primitives to indicate the presence of a per-vertex cumulative distance attribute semantic used for stable dash alignment.

This extension introduces two primary properties controlling line appearance:

* `width`: the pixel width of the rendered line
* `pattern`: an unsigned integer whose bits represent a repeating on/off pattern along the line

## Specifying Line Styles

The `BENTLEY_materials_line_style` extension is applied to a material. When that material is used by any line-type primitive, or by the edges described by the [`EXT_mesh_primitive_edge_visibility`](https://github.com/KhronosGroup/glTF/pull/2479) extension, it defines the **width** and **pattern** with which those lines are rendered.

When applied to a line-type primitive, the extension indicates that the primitive supplies the per-vertex cumulative distance attribute semantic defined below.

### Width

The `width` property specifies the line's thickness **in screen pixels**. The value of `width` must be greater than `0`.

For each line segment, implementations should extrude geometry by half this width on each side of the segment's centerline, perpendicular to its direction. Implementations should also insert triangles at the joints between line segments comprising a continuous line string or line loop, when appropriate to visually round out the joints.

### Pattern

The `pattern` property specifies a **repeating on/off bit pattern** applied along the length of the line, encoded as a 16-bit unsigned integer. Each bit corresponds to one screen pixel:

* Bit value `1`: lit (on)
* Bit value `0`: unlit (off)

The least significant bit (bit 0) corresponds to the start of the pattern. The pattern repeats cyclically after all 16 bits have been used. If a pattern requires fewer than 16 bits, its unused higher-order bits must be `0`.

The pattern must be applied continuously along each continuous line string or line loop so that connected segments remain visually consistent. Implementations must not restart the pattern at intermediate vertices within the same line string.

Example patterns:

| Bit Pattern (hex) | Binary | Description     |
|--------------------|---------|----------------|
| `0xFFFF`          | 1111111111111111 | solid line |
| `0xAAAA`          | 1010101010101010 | dotted line |
| `0xF0F0`          | 1111000011110000 | dashed line |
| `0xC3C3`          | 1100001111000011 | custom pattern |

### Cumulative Distance (Primitive Attribute)

When the extension is applied to a line-type primitive, the primitive must supply a vertex attribute with the semantic:

* `BENTLEY_materials_line_style:CUMULATIVE_DISTANCE`

This attribute provides a per-vertex **cumulative distance** from the start of the line string, expressed in model/world units. Implementations use this distance to align the dash pattern consistently along the full line string.

The accessor for this attribute:

* **MUST** be `SCALAR`.
* **MUST** have a count matching the primitive's vertex count.
* **MUST** use component type `FLOAT` **OR** a normalized integer type.

Values are expected to be non-decreasing along each line string.

## Implementation Notes

Because many graphics APIs do not support line primitives with a width larger than 1, tessellation is generally required to draw wide lines.

## JSON Schema

* [material.BENTLEY_materials_line_style.schema.json](schema/material.BENTLEY_materials_line_style.schema.json)
* [mesh.primitive.BENTLEY_materials_line_style.schema.json](schema/mesh.primitive.BENTLEY_materials_line_style.schema.json)

## Known implementations

* [iTwin.js](https://github.com/iTwin/itwinjs-core/pull/8856)
* [Cesium.js](https://github.com/CesiumGS/cesium/pull/13110)