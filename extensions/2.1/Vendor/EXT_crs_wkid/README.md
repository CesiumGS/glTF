# 3DTILES\_crs\_wkid

## Contributors

- Sean Lilley, Cesium
- Don McCurdy, Cesium
- Björn Blissing, Vantor
- Tamrat Belayneh, ESRI

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

## Optional vs. Required

This extension is optional, meaning it should be placed in the glTF root's `extensionsUsed` list, but not in the `extensionsRequired` list.

## Overview

This extension declares the Coordinate Reference System (CRS) in which a glTF 2.1 asset was authored. The CRS is provided as a Well-Known ID (WKID), commonly an [EPSG](https://epsg.org/home.html) code.

Assets with this extension are declared to have been authored for geospecific usage, with a particular CRS. Without this extension, glTF assets are understood to have been authored using the coordinate system of the base glTF specification — right-handed, +Y up, +Z forward, and -X right — as defined in the [Coordinate System and Units](https://www.khronos.org/registry/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units) section.

The following example shows an asset annotated to indicate a [WGS 84](https://epsg.org/ellipsoid_7030/WGS-84.html) Earth-centered, Earth-fixed (ECEF) geocentric coordinate reference system ([EPSG 4978](https://epsg.org/crs_4978/WGS-84.html)).

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "EXT_crs_wkid": {
      "authority": "EPSG",
      "wkid": 4978
    }
  }
}
```

The extension has the following properties:

- `authority` is the organization that defines and manages a specific CRS identifier.
- `wkid` is an integer that identifies the CRS.
- `vcsWkid` (optional) is an integer that identifies the vertical CRS if `wkid` identifies a horizontal CRS.
- `epoch` (optional) is an epoch for coordinates that are referenced to a dynamic CRS such as WGS 84. Expressed as a decimal year (e.g. `"2019.81"`). See [WKT representation of coordinate epoch and coordinate metadata](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html#128) for more details.

## Examples

An asset defined in [WGS 84](https://epsg.org/ellipsoid_7030/WGS-84.html) geocentric coordinates with a defined epoch:

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "EXT_crs_wkid": {
      "authority": "EPSG",
      "wkid": 4978,
      "epoch": "2019.81"
    }
  }
}
```

An asset defined in Moon planetocentric coordinates:

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "EXT_crs_wkid": {
      "authority": "IAU_2015",
      "wkid": 30100
    }
  }
}
```

An asset defined with a horizontal CRS (UTM Zone 11N) and vertical CRS (NAVD 88):

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "EXT_crs_wkid": {
      "authority": "EPSG",
      "wkid": 32611,
      "vcsWkid": 5703
    }
  }
}
```
