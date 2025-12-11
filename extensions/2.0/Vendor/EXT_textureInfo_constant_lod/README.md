<!--
Copyright 2015-2025 The Khronos Group Inc.
SPDX-License-Identifier: CC-BY-4.0
-->

# EXT_textureInfo_constant_lod

## Contributors

* Paul Connelly, Bentley Systems, [@pmconne](https://github.com/pmconne)
* Erin Ingram, Bentley Systems, [@eringram](https://github.com/eringram)

## Status

Draft

## Dependencies

Written against the glTF 2.0 spec.

## Overview

Constant level-of-detail ("LOD") is a technique of texture coordinate generation which dynamically calculates cordinates to keep the texture near a certain size on the screen, thus preserving the level of detail no matter what the zoom level. It blends from one size of the texture to another as the view is zoomed in or out so that the change is smooth.

The `EXT_textureInfo_constant_lod` extension defines properties needed to calculate these texture coordinates: the number of times the texture is repeated, an offset to shift the texture, the minimum and maximum distance for which to clamp the texture. The minimum and maximum clamp distance control at which point the texture sizes should be blended. These images illustrate the textures blending to create the constant LOD effect:

![Constant LOD gif](./figures/constantlod.gif "Constant LOD gif")

![Constant LOD image](./figures/constantlod.jpg "Constant LOD image")

## Specifying Constant LOD Texture Mapping

### Repetitions

The `repetitions` property specifies the number of times the texture is repeated. Increasing this will make the texture pattern appear smaller, decreasing it will make it larger.

### Offset

The `offset` property is used to shift the texture, specified as a pair of numbers which are UV coordinates in the format [U, V].

### Minimum Clamp Distance

The `minClampDistance` property specifies the minimum distance in meters from the eye to the surface at which to clamp the texture.

### Maximum Clamp Distance

The `minClampDistance` property specifies the maximum distance in meters from the eye to the surface at which to clamp the texture.

## Implementation Notes


## JSON Schema

* [textureInfo.EXT_textureInfo_constant_lod.schema.json](schema/textureInfo.EXT_textureInfo_constant_lod.schema.json)
