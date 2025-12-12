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

The `EXT_textureInfo_constant_lod` extension is defined on `textureInfo` structures. When that `textureInfo` is used by a material, this extension applies the constant LOD technique to the specified texture.

### Repetitions

The `repetitions` property specifies the number of times the texture is repeated. Increasing this will make the texture pattern appear smaller, decreasing it will make it larger.

### Offset

The `offset` property is used to shift the texture, specified as a pair of numbers in meters in the format [X, Y].

### Minimum Clamp Distance

The `minClampDistance` property specifies the minimum distance in meters from the eye to the surface at which to clamp the texture.

### Maximum Clamp Distance

The `maxClampDistance` property specifies the maximum distance in meters from the eye to the surface at which to clamp the texture.

## Implementation Notes

This is a general formula that can be used to calculate the two different texture coordinates and how to blend them.

In the vertex shader, where $worldPosition$ is the vertex position in world coordinates and $eyeSpace$ is the vertex position in camera coordinates:

$$customUvCoords = worldPosition + offset$$
$$\text{if }isPerspectiveProjection \text{:}$$
$$customUvCoords.z = -eyeSpace.z$$
$$\text{if }isOrthographicProjection \text{:}$$
$$customUvCoords.z = frustumWidth$$

The resulting $customUvCoords.xy$ should contain the vertex's X and Y position in world coordinates. $customUvCoords.z$ is the negative eye-space depth (z-coordinate) from the camera to the fragment when using perspective projection. Since all points are equidistant to the camera when using orthographic projection, this value should be set to the frustum width as a proxy for zoom level.

In the fragment shader:

$$logDepth = \log_{2}{customUvCoords.z}$$
$$textureCoordinates_1 = customUvCoords.xy \div clamp(2^{\lfloor logDepth \rfloor}, minClampDistance, maxClampDistance) \times repetitions$$
$$textureCoordinates_2 = customUvCoords.xy \div clamp(2^{\lfloor logDepth \rfloor + 1}, minClampDistance, maxClampDistance) \times repetitions$$
$$result = mix(textureCoordinates_1, textureCoordinates_2, fract(logDepth))$$

>Note: the addition of $\lfloor logDepth \rfloor + 1$ when calculating $textureCoordinates_2$ allows the result of the $mix$ function to blend between two adjacent mipmap levels.

Where $clamp(x, minVal, maxVal)$ constrains the value of $x$ to the range of $minVal$ to $maxVal$, defined by the [GLSL definition](https://registry.khronos.org/OpenGL-Refpages/gl4/html/clamp.xhtml) of:

$$\min(\max(x, minVal), maxVal)$$

$fract(x)$ returns the fractional part of $x$, defined by the [GLSL definition](https://registry.khronos.org/OpenGL-Refpages/gl4/html/fract.xhtml) of:

$$x - \lfloor x \rfloor$$

and $mix(x, y, a)$ performs a linear interpolation between $x$ and $y$ using $a$ to weight between them, using the [GLSL definition](https://registry.khronos.org/OpenGL-Refpages/gl4/html/mix.xhtml) of:

$$x \times (1 − a) + y \times a$$

## JSON Schema

* [textureInfo.EXT_textureInfo_constant_lod.schema.json](schema/textureInfo.EXT_textureInfo_constant_lod.schema.json)
