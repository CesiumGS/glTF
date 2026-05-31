# 3DTILES\_crs

## Contributors

- Sean Lilley, Cesium
- Don McCurdy, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension declares the Coordinate Reference System (CRS) in which a glTF 2.0 asset was authored, which may differ from the default — right-handed, +Y up, +Z forward, and -X right — as defined in the [Coordinate System and Units](https://www.khronos.org/registry/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units) section of the glTF specification.

glTF assets used in certain contexts, and notably as 3D Tiles content, are georeferenced and aligned with a larger geospatial dataset. "+Y up" or "+Z up" are meaningless conventions in this context – there is no single "Up" vector on the surface of a globe.

Rather than requiring client implementations to transform generic glTF assets into a geospecific orientation at runtime, this extension allows content pipelines producing 3D Tiles with glTF content to prepare data as needed (e.g. orienting and perhaps splitting an asset) to be delivered in a 3D Tiles tileset. In order to identify that such processing has been applied and the default coordinate system no longer applies, assets are annotated with a CRS.

The following example shows an asset annotated to indicate a `EPSG:4978` CRS.

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "3DTILES_crs": {
      "crs": "EPSG:4978"
    }
  }
}
```

The `crs` property **SHOULD** be an EPSG code, WKT2 string, or other common CRS identifier.


Implementations of this extensions are only required to support [geocentric coordinate reference systems](#geocentric-crs). Other coordinate reference systems, such as projected or geographic coordinate reference systems, may be provided for application-specific purposes, but are discouraged as they require dedicated coordinate transformation libraries that are not always runtime efficient.

## Geocentric Coordinate Reference Systems

_This section is non-normative._

A geocentric CRS is typically a right-handed Cartesian coordinate system (x, y, z) where:

- The origin (0, 0, 0) is located at the center of mass of the central body.
- The X-axis points through the intersection of the equator and the prime meridian.
- The Y-axis points through the intersection of the equator and 90° longitude.
- The Z-axis points to the north pole.

The image below shows a [WGS 84](https://epsg.org/ellipsoid_7030/WGS-84.html) Earth-centered, Earth-fixed (ECEF) coordinate reference system ([EPSG 4978](https://epsg.org/crs_4978/WGS-84.html)).

<p align="center">
  <img src="./figures/ecef.png"/><br/>
</p>

Common geocentric `crs` values include:

`crs` | Description
--|--
`"EPSG:4978"` | WGS 84
`"EPSG:7656"` | WGS 84 (G730)
`"EPSG:7658"` | WGS 84 (G873)
`"EPSG:7660"` | WGS 84 (G1150)
`"EPSG:7662"` | WGS 84 (G1674)
`"EPSG:7664"` | WGS 84 (G1762)
`"EPSG:9753"` | WGS 84 (G2139)
`"EPSG:7842"` | GDA2020

### Precision Considerations

Geocentric coordinates are often quite large and can't be adequately represented in 32-bit floating point, which is the highest precision allowed by `POSITION` attribute accessors.

For example, given the geocentric coordinates:

* `x: 1254151.3944734565`
* `y: -4732843.845023793`
* `z: 4073794.407620059`

The closest representable values in 32-bit floating point would be

* `x: 1254151.375`
* `y: -4732844`
* `z: 4073794.5`

The results in an error of about 0.25 meters.

To mitigate floating point error, geocentric coordinates may be transformed to a local coordinate system and the inverse transformation — the local to global transform — may be set on the root node's `matrix` property in full precision in JSON. For more details about this approach see [Precisions, Precisions](https://help.agi.com/STKComponents/html/BlogPrecisionsPrecisions.htm).

Alternatively, the asset may be defined a local coordinate systems and georeferenced to a specific longitude and latitude  with [3DTILES_geopose](../3DTILES_geopose/README.md). This approach is fundamentally similar but provides clearer georeferencing semantics.

## Notes

_This section is non-normative._

This extension is a successor to the 3D Tiles 1.1 semantic [`TILESET_CRS_GEOCENTRIC`](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/Semantics/README.adoc#tileset-semantics) and the 3D Tiles 1.1 extension [3DTILES_ellipsoid](https://github.com/CesiumGS/3d-tiles/tree/main/extensions/3DTILES_ellipsoid).
