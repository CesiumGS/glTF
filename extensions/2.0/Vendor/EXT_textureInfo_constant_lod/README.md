<!--
Copyright 2025 Bentley Systems, Incorporated
SPDX-License-Identifier: CC-BY-4.0
-->

# EXT_textureInfo_constant_lod

## Contributors

* Paul Connelly, Bentley Systems, [@pmconne](https://github.com/pmconne)
* Erin Ingram, Bentley Systems, [@eringram](https://github.com/eringram)
* Mark Schlosser, Bentley Systems, [@markschlosseratbentley](https://github.com/markschlosseratbentley)
* Daniel Zhong, Bentley Systems, [@danielzhong](https://github.com/danielzhong)

## Status

Complete

## Dependencies

Written against the glTF 2.0 spec.

## Overview

Constant level-of-detail ("LOD") is a technique of texture coordinate generation which dynamically calculates texture coordinates to maintain a consistent texel-to-pixel ratio on screen, regardless of camera distance. As the camera zooms in or out, the texture coordinates are recalculated so that the texture pattern remains at approximately the same visual scale. The transition between scale levels is smoothly blended to avoid abrupt changes.

The `EXT_textureInfo_constant_lod` extension defines properties needed to calculate these dynamic texture coordinates: the number of times the texture is repeated per meter, an offset to shift the texture, and the minimum and maximum distance for which to clamp the texture. The minimum and maximum clamp distance control at which point the texture sizes shall be blended. These images illustrate the textures blending to create the constant LOD effect:

![Constant LOD gif](./figures/constantlod.gif "Constant LOD gif")

![Constant LOD image](./figures/constantlod.jpg "Constant LOD image")

The extension specifies an alternative way of computing the texture coordinates, so if it is supported by the client then the `textureInfo`'s `texCoord` is not used. For maximum compatibility however, encoders still must provide `texCoord` UV coordinates so the texture renders even if the extension is not supported.

## Specifying Constant LOD Texture Mapping

The `EXT_textureInfo_constant_lod` extension is defined on `textureInfo` structures. When that `textureInfo` is used by a material, this extension applies the constant LOD technique to the specified texture.

Constant LOD uses the following properties:

* `repetitions` - specifies the number of times the texture is repeated per meter in both the X and Y dimensions. Increasing this will make the texture pattern appear smaller; decreasing it will make it appear larger.
* `offset` - used to shift the texture, specified as a pair of numbers in meters in the format [X, Y].
* `minClampDistance` - specifies the minimum distance in meters from the camera to the surface at which to clamp the texture. This value must not be greater than `maxClampDistance`.
* `maxClampDistance` - specifies the maximum distance in meters from the camera to the surface at which to clamp the texture. This value must not be less than `minClampDistance`.

For example, the following JSON defines a material with a texture at index 0 that is modified by the `EXT_textureInfo_constant_lod` extension. The extension has a `repetitions` value of 2 causing it to be repeated twice per meter, making its pattern appear half the size. It is shifted by 1 meter in the X direction and 0 meters in the Y direction. It also has a minimum clamp distance of 0.5 meters, meaning the constant LOD effect stops being present when the camera is 0.5m away from the surface or closer. This is a closer distance than the default value of 1m; therefore, this material requires a closer zoom before the texture stops adapting its level of detail (and for it to appear magnified). The absence of the `maxClampDistance` property means it has a default value of $2^{32}$, so the constant LOD effect will occur until the camera is $2^{32}$ away from the surface.

```json
"materials": [
  {
    "pbrMetallicRoughness": {
      "baseColorTexture": {
        "index": 0,
        "texCoord": 0,
        "extensions": {
          "EXT_textureInfo_constant_lod": {
            "repetitions": 2,
            "offset": [1, 0],
            "minClampDistance": 0.5
          }
        }
      },
      "baseColorFactor": [1.0, 1.0, 1.0, 1.0], 
      "metallicFactor": 0.0,
      "roughnessFactor": 1.0
    }
  }
],
```

## Implementation Notes

The `EXT_textureInfo_constant_lod` extension may be present on any `textureInfo` object, including those that extend `textureInfo` such as `normalTextureInfo`. Encoders and clients may decide to implement the extension such that the normal texture has identical constant LOD properties to the base color texture to produce a synchronized blending of levels-of-detail, although this is not required.

### Formulas

This section outlines the formulas that shall be used by implementations to calculate the dynamic texture coordinates for constant LOD.

In the vertex shader:

```math
\begin{aligned}
customUvCoords.xy &= worldPosition.xy + offset\\
customUvCoords.z &= \begin{cases}
-eyeSpace & \text{if } isPerspectiveProjection \\
frustrumWidth & \text{if } isOrthographicProjection \\
\end{cases}
\end{aligned}
```

Where $worldPosition$ is the vertex position in world coordinates, $eyeSpace$ is the vertex position in camera coordinates, and $frustumWidth$ is the frustum width in meters.

The resulting $customUvCoords.xy$ contain the vertex's X and Y position in world coordinates. $customUvCoords.z$ is the negative eye-space depth (z-coordinate) from the camera to the fragment when using perspective projection. Since all points are equidistant to the camera when using orthographic projection, this value shall be set to the frustum width as a proxy for zoom level. $customUvCoords$ should then be passed into the fragment shader as a `varying`.

In the fragment shader:

```math
\begin{aligned}
(x, y, z) &= customUvCoords\\
minDistance &= min(clampDistance) \\
maxDistance &= max(clampDistance) \\
logDepth &= \log_{2}{z}\\
textureCoordinates_1 &= \frac{xy}{clamp(2^{\lfloor logDepth \rfloor}, minDistance, maxDistance)} \cdot repetitions\\
textureCoordinates_2 &= \frac{xy}{clamp(2^{\lfloor logDepth \rfloor + 1}, minDistance, maxDistance)} \cdot repetitions\\
result &= mix(textureCoordinates_1, textureCoordinates_2, fract(logDepth))
\end{aligned}
```

*Non-normative note: the addition of $`\lfloor logDepth \rfloor + 1`$ when calculating $`textureCoordinates_2`$ allows the result of the $`mix`$ function to blend between two adjacent mipmap levels.*

Where $clamp(x, minVal, maxVal)$ constrains the value of $x$ to the range of $minVal$ to $maxVal$, defined by the [GLSL definition](https://registry.khronos.org/OpenGL-Refpages/gl4/html/clamp.xhtml) of:

```math
\min(\max(x, minVal), maxVal)
```

$fract(x)$ returns the fractional part of $x$, defined by the [GLSL definition](https://registry.khronos.org/OpenGL-Refpages/gl4/html/fract.xhtml) of:

```math
x - \lfloor x \rfloor
```

and $mix(x, y, a)$ performs a linear interpolation between $x$ and $y$ using $a$ to weight between them, using the [GLSL definition](https://registry.khronos.org/OpenGL-Refpages/gl4/html/mix.xhtml) of:

```math
x \cdot (1 − a) + y \cdot a
```

## JSON Schema

* [textureInfo.EXT_textureInfo_constant_lod.schema.json](schema/textureInfo.EXT_textureInfo_constant_lod.schema.json)

## Known Implementations

* [iTwin.js](https://github.com/iTwin/itwinjs-core/pull/8882)
* [CesiumJS](https://github.com/CesiumGS/cesium/pull/13121)
