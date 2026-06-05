# 3DTILES\_crs

## Contributors

- Sean Lilley, Cesium
- Don McCurdy, Cesium
- Björn Blissing, Vantor

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
      "type": "geocentric"
    },
    "3DTILES_crs_epsg": {
      "code": "EPSG:4978"
    }
  }
}
```

Assets annotated with the `3DTILES_crs` extension are declared to have been authored for geospecific usage, with a particular CRS. Without this property, glTF assets are understood to have been authored using the coordinate system of the base glTF specification.

## CRS Types

This extension defines a single property `type` which can be one of the following values:

`type`|Description|Diagram
--|--|--
`"local"`|A local coordinate system. By default, a right-handed Cartesian coordinate system in meters where `+x` axis faces east, `+y` axis faces north, and `+z` axis faces up. The default CRS **MAY** be overriden by a [sub extension](#crs-identification-mechanisms).|![](./figures/local.png)
`"geocentric"`|A geocentric (planetocentric) coordinate system. By default [`EPSG:4978`](https://epsg.org/crs_4978/WGS-84.html) which defines `0,0,0` as the center of mass on Earth, where `+Z` extends through true north (i.e. the geodetic North Pole) and `+X` intersects the sphere of the earth at 0° latitude (the equator) and 0° longitude (the prime meridian which passes through Greenwich). See notes on [precision](#precision). The default CRS **MAY** be overriden by a [sub extension](#crs-identification-mechanisms).|![](./figures/ecef.png)
`"geographic"`|A geographic (planetographic) coordinate system. By default [`EPSG:4979`](https://epsg.org/crs_4979/WGS-84.html) which defines coordinates as _(latitude, longitude, height)_ on to the WGS84 ellipsoid. The default CRS **MAY** be overriden by a [sub extension](#crs-identification-mechanisms).|
`"projected"`|A projected coordinate system. There is no default; the CRS **MUST** be provided by a [sub extension](#crs-identification-mechanisms).|

Additional types may be defined by extensions.

> **Note:** Implementations are only required to support `"geocentric"` and `"local"`. Other types may be used for application-specific purposes, but are discouraged as they often require dedicated coordinate transformation libraries and ancillary data, such as grid shift files, in order to be rendered in 3D globe engines.

## CRS Identification Mechanisms

Coordinate reference systems can represented in multiple ways such as [EPSG](https://epsg.org/home.html) codes, `IAU` codes, [WKT2](https://www.ogc.org/standards/wkt-crs/) strings, or other mechanisms. Aiming for maximum conformance and interoperability these are split into separate sub-extensions:

- [3DTILES_crs_epsg](../3DTILES_crs_epsg/README.md)
- [3DTILES_crs_iau](../3DTILES_crs_iau/README.md)
- [3DTILES_crs_wkt2](../3DTILES_crs_wkt2/README.md)

(Additional extensions may be added in the future)


`3DTILES_crs` is usually paired with a sub-extension that provides the specific CRS, for example:

### Example 1: EPSG code (ECEF)

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "3DTILES_crs": {
      "type": "geocentric"
    },
    "3DTILES_crs_epsg": {
      "code": "EPSG:4978",
      "epoch": "2019.81"
    }
  }
}
```

### Example 2: IAU code (Moon)

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "3DTILES_crs": {
      "type": "geocentric"
    },
    "3DTILES_crs_iau": {
      "code": "IAU_2015:30100"
    }
  }
}
```

### Example 3: Projected CRS (UTM zone 11N)
```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "3DTILES_crs": {
      "type": "projected"
    },
    "3DTILES_wkt2": {
      "string": "PROJCRS[\"WGS 84 / UTM zone 11N\",BASEGEOGCRS[\"WGS 84\",ENSEMBLE[\"World Geodetic System 1984 ensemble\",MEMBER[\"World Geodetic System 1984 (Transit)\"],MEMBER[\"World Geodetic System 1984 (G730)\"],MEMBER[\"World Geodetic System 1984 (G873)\"],MEMBER[\"World Geodetic System 1984 (G1150)\"],MEMBER[\"World Geodetic System 1984 (G1674)\"],MEMBER[\"World Geodetic System 1984 (G1762)\"],MEMBER[\"World Geodetic System 1984 (G2139)\"],MEMBER[\"World Geodetic System 1984 (G2296)\"],ELLIPSOID[\"WGS 84\",6378137,298.257223563,LENGTHUNIT[\"metre\",1]],ENSEMBLEACCURACY[2.0]],PRIMEM[\"Greenwich\",0,ANGLEUNIT[\"degree\",0.0174532925199433]],ID[\"EPSG\",4326]],CONVERSION[\"UTM zone 11N\",METHOD[\"Transverse Mercator\",ID[\"EPSG\",9807]],PARAMETER[\"Latitude of natural origin\",0,ANGLEUNIT[\"degree\",0.0174532925199433],ID[\"EPSG\",8801]],PARAMETER[\"Longitude of natural origin\",-117,ANGLEUNIT[\"degree\",0.0174532925199433],ID[\"EPSG\",8802]],PARAMETER[\"Scale factor at natural origin\",0.9996,SCALEUNIT[\"unity\",1],ID[\"EPSG\",8805]],PARAMETER[\"False easting\",500000,LENGTHUNIT[\"metre\",1],ID[\"EPSG\",8806]],PARAMETER[\"False northing\",0,LENGTHUNIT[\"metre\",1],ID[\"EPSG\",8807]]],CS[Cartesian,2],AXIS[\"(E)\",east,ORDER[1],LENGTHUNIT[\"metre\",1]],AXIS[\"(N)\",north,ORDER[2],LENGTHUNIT[\"metre\",1]],USAGE[SCOPE[\"Navigation and medium accuracy spatial referencing.\"],AREA[\"Between 120°W and 114°W, northern hemisphere between equator and 84°N, onshore and offshore. Canada - Alberta; British Columbia (BC); Northwest Territories (NWT); Nunavut. Mexico. United States (USA).\"],BBOX[0,-120,84,-114]],ID[\"EPSG\",32611]]"
    }
  }
}
```

## Notes

_This section is non-normative._

This extension and sub-extensions are the successor to the 3D Tiles 1.1 semantics [`TILESET_CRS_GEOCENTRIC`](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/Semantics/README.adoc#tileset-semantics) and [`TILESET_CRS_COORDINATE_EPOCH`](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/Semantics/README.adoc#tileset-semantics) and the 3D Tiles 1.1 extension [3DTILES_ellipsoid](https://github.com/CesiumGS/3d-tiles/tree/main/extensions/3DTILES_ellipsoid).


## Appendix

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

To mitigate floating-point error, coordinates may be transformed to a local tangent plane such that 32-bit floating-point precision is adequate to describe the distance between each position and the origin of the tangent plane. These relative positions are stored in the glTF vertex data. The transformation from the local tangent plane to the ellipsoid's fixed reference frame is stored in the node's `matrix` property in full precision in JSON, or may be expressed with [`3DTILES_georeference`](../3DTILES_georeference/README.md).

<p align="center">
  <img src="./figures/enu-xyz.png"/><br/>
</p>

For more details on this approach see [Precisions, Precisions](https://help.agi.com/STKComponents/html/BlogPrecisionsPrecisions.htm).




## TODO

- What happens if an external asset defines `3DTILES_crs` with a different `crs`?
