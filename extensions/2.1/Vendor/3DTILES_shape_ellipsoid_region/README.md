<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# 3DTILES\_shape\_ellipsoid\_region

## Contributors

- Janine Liu, Cesium
- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension defines an ellipsoid-conforming region as an additional shape type for glTF 2.1 shapes. These regions are commonly used in geospatial applications to describe volumes that conform to the curvature of the Earth, or other bodies.

`3DTILES_shape_ellipsoid_region` extends the `shape` object in glTF 2.1. The `shape.type` **MUST** be set to `"ellipsoid region"`. The properties define the region following the surface of the ellipsoid between two different height values.

The volume does not necessarily contain the full ellipsoid—and for many geospatial use cases, it will not. Rather, the ellipsoid is used as a reference from which the actual region is extruded. However, a region may extend beneath the surface of the ellipsoid. Given the right height values, the region could contain the entire ellipsoid if desired.

## Ellipsoid Region Shape

An ellipsoid region shape is defined by adding the `3DTILES_shape_ellipsoid_region` extension to a `shape` object of type `"ellipsoid region"`.

### Properties

| Property | Type | Description | Required |
|---|---|---|---|
| **semiMajorAxisRadius** | `number` | The radius along the semi-major axis of the reference ellipsoid in meters. Corresponds to the radii along the X and Z axes. | Yes, minimum: `0.0` |
| **semiMinorAxisRadius** | `number` | The radius along the semi-minor axis of the reference ellipsoid in meters. Corresponds to the radius along the Y-axis. | Yes, minimum: `0.0` |
| **minimumHeight** | `number` | The minimum height of the region relative to the ellipsoid's surface, in meters. May be negative. | Yes |
| **maximumHeight** | `number` | The maximum height of the region relative to the ellipsoid's surface, in meters. May be negative. | Yes |
| **minimumLatitude** | `number` | The minimum latitude of the region, in radians. Must be in the range `[-pi/2, pi/2]`. | No, default: `-1.57079632679` |
| **maximumLatitude** | `number` | The maximum latitude of the region, in radians. Must be in the range `[-pi/2, pi/2]`. | No, default: `1.57079632679` |
| **minimumLongitude** | `number` | The minimum longitude of the region, in radians. See [Longitude](#longitude) for evaluation details. | No, default: `-3.14159265359` |
| **maximumLongitude** | `number` | The maximum longitude of the region, in radians. See [Longitude](#longitude) for evaluation details. | No, default: `3.14159265359` |

#### Axis Radius

The semi-minor and semi-major axes of the reference ellipsoid are defined as floats using the `semiMinorAxisRadius` and `semiMajorAxisRadius` properties. Both the `semiMinorAxisRadius` and `semiMajorAxisRadius` are required and **MUST** be defined.

The property value of `semiMajorAxisRadius` corresponds to the radii along the X and Z axes. The property value of `semiMinorAxisRadius` corresponds to the radius along the Y-axis.

> [!IMPORTANT]
> For prolate spheroids, the geometrical semi-major axis (polar radius along Y) is larger than the equatorial radius (X and Z). However, for schema consistency across all shape types, this extension always uses `semiMajorAxisRadius` for the X and Z axes (equatorial radii) and `semiMinorAxisRadius` for the Y-axis (polar radius), following the convention of oblate reference bodies like Earth.

#### Height

The minimum and maximum height of the region relative to the ellipsoid's surface is defined by the `minimumHeight` and `maximumHeight` properties. These represent the height in meters referenced to the surface. Negative values are below the surface, and positive values are above the service.

The height values must satisfy the condition $height_{min} \leqslant height_{max}$.

#### Latitude

The latitudinal span of the region is defined using the `minimumLatitude` and `maximumLatitude` properties. The values are the geodetic latitude represented in radians.

Both `minimumLatitude` and `maximumLatitude` **MUST** be in the range $-\frac{\pi}{2} \leqslant latitude \leqslant \frac{\pi}{2}$ and satisfy the condition $latitude_{min} \leqslant latitude_{max}$.

#### Longitude

The longitudinal span of the region is defined using the `minimumLongitude` and `maximumLongitude` properties. These values are stored as radians.

When evaluated, the `minimumLongitude` and `maximumLongitude` values **MUST** satisfy this requirement:

$$0 \leqslant | longitude_{max} - longitude_{min} | \leqslant 2 \cdot \pi$$

This helps to preserve sampling at the antemeridian.

### Details

The reference ellipsoid is centered at the origin. The `semiMajorAxisRadius` indicates the radius of the ellipsoid in meters along the `x` and `z` axes. The `semiMinorAxisRadius` indicates the radius of the ellipsoid in meters along the `y` axis.

> [!NOTE]
> The `x` and `z` radii are made equal to simplify the math required to render implicit regions along the ellipsoid.

The `minimumHeight` and `maximumHeight` properties indicate the heights of the region from the ellipsoid's surface in meters. A height of `0` sits right at the surface. Negative heights are also valid—they simply extend underneath the ellipsoid's surface.

This example corresponds to the image below it:

```json
"shapes": [
  {
    "name": "Global Ellipsoidal Height Region",
    "type": "ellipsoid region",
    "extensions": {
      "3DTILES_shape_ellipsoid_region": {
        "semiMajorAxisRadius": 3.5,
        "semiMinorAxisRadius": 2.0,
        "minimumHeight": 0.0,
        "maximumHeight": 0.5
      }
    }
  }
]
```

![](figures/hollow-ellipsoid.png)

An ellipsoid region may also be confined to a specific latitude and/or longitude range. The `minimumLatitude` and `maximumLatitude` properties represent the latitude values at which the region starts and stops, defined in the range `[-pi/2, pi/2]`. Similarly, the `minimumLongitude` and `maximumLongitude` properties represent the longitude bounds within the range `[-pi, pi]`.

```json
"shapes": [
  {
    "name": "Geographic Bounding Region (Ellipsoid)",
    "type": "ellipsoid region",
    "extensions": {
      "3DTILES_shape_ellipsoid_region": {
        "semiMajorAxisRadius": 3.5,
        "semiMinorAxisRadius": 2.0,
        "minimumHeight": 0.0,
        "maximumHeight": 0.5,
        "minimumLongitude": 0.0,
        "maximumLongitude": 1.57079632679,
        "minimumLatitude": -0.78539816339,
        "maximumLatitude": 0.78539816339
      }
    }
  }
]
```

![](figures/half-ellipsoid.png)

It is valid for the `maximumLongitude` property to be less than `minimumLongitude`. This would define a region that crosses over the line at `-pi` or `pi`, equivalent to the International Date Line on Earth.

## Implicit Subdivision

When used with [Implicit Tiling](../../3DTILES_implicit_tiling/README.md), the implicit tile coordinates are interpreted as `(longitude, height, latitude)` for the ellipsoid region.

A `QUADTREE` subdivision will subdivide along the longitude and latitude axes. An `OCTREE` subdivision will subdivide along the longitude, height, and latitude axes.

| Root Region  | Quadtree | Octree |
|---|---|---|
| ![Parent Cell](figures/root.png)  | ![Quadtree Cells](figures/quadtree.png)  | ![Octree Cells](figures/octree.png)  |

Coordinate|Positive Direction
--|--
x| From west to east (increasing longitude)
y| From bottom to top (increasing height)
z| From south to north (increasing latitude)
