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

This extension declares the Coordinate Reference System (CRS) in which a glTF 2.0 asset was authored, which may differ from the default — meters, right-handed, +Y up, +Z forward, and -X right — as defined in the [Coordinate System and Units](https://www.khronos.org/registry/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units) section of the glTF specification.


The following example shows an asset annotated to indicate a [WGS 84](https://epsg.org/ellipsoid_7030/WGS-84.html) Earth-centered, Earth-fixed (ECEF) geocentric coordinate reference system ([EPSG 4978](https://epsg.org/crs_4978/WGS-84.html)).

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

Assets annotated with the `crs` property are declared to have been authored for geospecific usage, with a particular CRS. Without this property, glTF assets are understood to have been authored using the coordinate system of the base glTF specification.

The `crs` property **SHOULD** be an [EPSG](https://epsg.org/home.html) code, [WKT2](https://www.ogc.org/standards/wkt-crs/) string, or other common CRS identifier.

Implementations are only required to support [geocentric CRSs](#geocentric-crs). Other coordinate reference systems, such as projected or geographic coordinate reference systems, may be used for application-specific purposes, but are discouraged as they often require dedicated coordinate transformation libraries and ancillary data, such as grid shift files, in order to be rendered in 3D globe engines.

## Geocentric CRS

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

Common geocentric `crs` values include, but are not limited to:

`crs` | Description
--|--
`"EPSG:4978"` | WGS 84
`"EPSG:7656"` | WGS 84 (G730)
`"EPSG:7658"` | WGS 84 (G873)
`"EPSG:7660"` | WGS 84 (G1150)
`"EPSG:7662"` | WGS 84 (G1674)
`"EPSG:7664"` | WGS 84 (G1762)
`"EPSG:9753"` | WGS 84 (G2139)
`"EPSG:10604"` | WGS 84 (G2296)
`"EPSG:7842"` | GDA2020

### Precision

Geocentric coordinates often can't be adequately represented in 32-bit floating-point, which is the highest precision allowed by `POSITION` attribute accessors.

For example, given the geocentric coordinates:

- `x: 1254151.3944734565`
- `y: -4732843.845023793`
- `z: 4073794.407620059`

The closest representable values in 32-bit floating-point would be

- `x: 1254151.375`
- `y: -4732844`
- `z: 4073794.5`

The results in an error of about 0.25 meters.

To mitigate floating-point error, geocentric coordinates may be transformed to a local tangent plane such that 32-bit floating-point precision is adequate to describe the distance between each position and the origin of the tangent plane. These relative positions are stored in the glTF vertex data. The transformation from the local tangent plane to the ellipsoid's fixed reference frame is stored in the node's `matrix` property in full precision in JSON, or may be expressed with [`3DTILES_georeference`](../3DTILES_georeference/README.md).

<p align="center">
  <img src="./figures/enu-xyz.png"/><br/>
</p>

For more details on this approach see [Precisions, Precisions](https://help.agi.com/STKComponents/html/BlogPrecisionsPrecisions.htm).

## Notes

_This section is non-normative._

This extension is a successor to the 3D Tiles 1.1 semantic [`TILESET_CRS_GEOCENTRIC`](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/Semantics/README.adoc#tileset-semantics) and the 3D Tiles 1.1 extension [3DTILES_ellipsoid](https://github.com/CesiumGS/3d-tiles/tree/main/extensions/3DTILES_ellipsoid).

## TODO

- Decide on scope of `crs` property. Should it be flexible or limited to a single CRS identification mechanism?
- Does an external asset inherit the CRS of this asset or does it need to be explicitly provided?
- What happens if an external asset defines `3DTILES_crs` with a different `crs`?
- May need an `epoch` property like `TILESET_CRS_COORDINATE_EPOCH`
- How to specify coordinate systems for other planetary bodies like Moon and Mars?
- Is it possible to specify a local z-up coordinate system?
