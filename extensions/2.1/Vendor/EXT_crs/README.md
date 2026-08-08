<!--
SPDX-FileCopyrightText: 2026 The Khronos Group Inc.

SPDX-License-Identifier: CC-BY-4.0
-->

# EXT\_crs

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

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension declares the Coordinate Reference System (CRS) in which a glTF 2.1 asset was authored. 



Assets with this extension are declared to have been authored for geospecific usage, with a particular CRS that overrides the coordinate system of the base glTF specification — right-handed, +Y up, +Z forward, and -X right — as defined in the [Coordinate System and Units](https://www.khronos.org/registry/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units) section.

For example, the CRS may be a projected CRS that is Z-up and U.S. Survey Foot, or a geocentric CRS where there is no single "up" direction. In either case, the provided CRS overrides the default glTF coordinate system.

There are several standard formats used to define coordinate reference systems. This extension doesn't adhere to a specific format,  instead defines a `"type"` property whose value **MUST** be defined by additional extensions.

The list of additional extensions includes (but is not limited to):

- [EXT_crs_wkid](../EXT_crs_wkid/README.md)
- [EXT_crs_wkt2](../EXT_crs_wkt2/README.md)

This flexibility allows implementations to chose with CRS formats they would like to support.

The following example shows an asset annotated to indicate a [WGS 84](https://epsg.org/ellipsoid_7030/WGS-84.html) Earth-centered, Earth-fixed (ECEF) geocentric coordinate reference system ([EPSG 4978](https://epsg.org/crs_4978/WGS-84.html)) with `EXT_crs` and `EXT_crs_wkid`.

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "EXT_crs": {
      "type": "wkid",
      "extensions": {
        "EXT_crs_wkid": {
          "authority": "EPSG",
          "wkid": 4978
        }
      }
    }
  }
}
```

The following example shows an asset defined in UTM Zone 11N coordinates using `EXT_crs` and `EXT_crs_wkt2`.

```json
{
  "asset": {
    "version": "2.1"
  },
  "extensions": {
    "EXT_crs": {
      "type": "wkt2",
      "extensions": {
        "EXT_crs_wkt2": {
          "wkt2": "PROJCRS[\"WGS 84 / UTM zone 11N\",BASEGEOGCRS[\"WGS 84\",ENSEMBLE[\"World Geodetic System 1984 ensemble\",MEMBER[\"World Geodetic System 1984 (Transit)\"],MEMBER[\"World Geodetic System 1984 (G730)\"],MEMBER[\"World Geodetic System 1984 (G873)\"],MEMBER[\"World Geodetic System 1984 (G1150)\"],MEMBER[\"World Geodetic System 1984 (G1674)\"],MEMBER[\"World Geodetic System 1984 (G1762)\"],MEMBER[\"World Geodetic System 1984 (G2139)\"],MEMBER[\"World Geodetic System 1984 (G2296)\"],ELLIPSOID[\"WGS 84\",6378137,298.257223563,LENGTHUNIT[\"metre\",1]],ENSEMBLEACCURACY[2.0]],PRIMEM[\"Greenwich\",0,ANGLEUNIT[\"degree\",0.0174532925199433]],ID[\"EPSG\",4326]],CONVERSION[\"UTM zone 11N\",METHOD[\"Transverse Mercator\",ID[\"EPSG\",9807]],PARAMETER[\"Latitude of natural origin\",0,ANGLEUNIT[\"degree\",0.0174532925199433],ID[\"EPSG\",8801]],PARAMETER[\"Longitude of natural origin\",-117,ANGLEUNIT[\"degree\",0.0174532925199433],ID[\"EPSG\",8802]],PARAMETER[\"Scale factor at natural origin\",0.9996,SCALEUNIT[\"unity\",1],ID[\"EPSG\",8805]],PARAMETER[\"False easting\",500000,LENGTHUNIT[\"metre\",1],ID[\"EPSG\",8806]],PARAMETER[\"False northing\",0,LENGTHUNIT[\"metre\",1],ID[\"EPSG\",8807]]],CS[Cartesian,2],AXIS[\"(E)\",east,ORDER[1],LENGTHUNIT[\"metre\",1]],AXIS[\"(N)\",north,ORDER[2],LENGTHUNIT[\"metre\",1]],USAGE[SCOPE[\"Navigation and medium accuracy spatial referencing.\"],AREA[\"Between 120°W and 114°W, northern hemisphere between equator and 84°N, onshore and offshore. Canada - Alberta; British Columbia (BC); Northwest Territories (NWT); Nunavut. Mexico. United States (USA).\"],BBOX[0,-120,84,-114]],ID[\"EPSG\",32611]]"
        }
      }
    }
  }
}
```
