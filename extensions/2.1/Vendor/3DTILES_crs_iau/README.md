# 3DTILES\_crs\_iau

## Contributors

- Sean Lilley, Cesium
- Mark Dane, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [3DTILES_crs](../3DTILES_crs/README.md).

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension declares the Coordinate Reference System (CRS) in which a glTF 2.1 asset was authored as an [International Astronomical Union (IAU) ](https://www.iau.org/) identifier. This extension is most commonly used for declaring CRSs for non-Earth planetary bodies such as the Moon and Mars.

This extension **MUST** be used with [3DTILES_crs](../3DTILES_crs/README.md).

The following example shows an asset declared as `IAU_2015:40100`, a geocentric coordinate system for the Moon.

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
      "id": "IAU_2015:30100",
      "body": "Moon"
    }
  }
}
```

The extension has the following properties:

- `id` is an IAU identifier. Examples include `"IAU_2015:30100"` for Moon and `"IAU_2015:49902"` for Mars.
- `body` is the name of the central body as recognized by the IAU . For example, the [IAU names](https://www.iau.org/public/themes/naming/#majorplanetsandmoon) of the major planet’s and Earth’s moon are: Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune, and Moon.


