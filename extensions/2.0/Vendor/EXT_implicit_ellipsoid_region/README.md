# EXT_implicit_ellipsoid_region

## Contributors
- Sean Lilley, Cesium
- Janine Liu, Cesium

## Status
Draft

## Dependencies
Written against the glTF 2.0 specification.

## Overview

This extension adds an ellipsoid-based region as an additional shape type to the [`KHR_implicit_shapes`](TODO) extension. Ellipsoid-based regions are commonly used in geospatial applications to describe volumes that conform to the curvature of the Earth, or other bodies.

`EXT_implicit_ellipsoid_region` is an extension on the `shape` object in `KHR_implicit_shapes`, where `type` should be set to `"ellipsoid region"`.

The extension's properties specify a region extruded from the surface of an ellipsoid. The ellipsoid itself is not necessarily part of the volume; it is merely used as reference from which the actual region is defined. However, a region may be extended beneath the surface of the ellipsoid, and thus can possibly contain the entire ellipsoid volume if desired.

### Details

The reference ellipsoid is centered at the origin. The `semiMajorAxisRadius` indicates the radii of the ellipsoid in meters along the `x` and `z` axes. These axes are made equal as a conscious decision to simplify the math implemented for rendering regions.

The `semiMinorAxisRadius` indicates the radius of the ellipsoid in meters along the `y` axis.

The `minHeight` and `maxHeight` properties indicates the heights of the region from the ellipsoid's surface, in meters. The `minimum` height should be a lower value, but not necessarily lower in magnitude. For example, `maxHeight` may be `10` while `minHeight` is `-100`.

<table>
  <tr>
    <th>
    Example
    </th>
  </tr>
  <tr>
    <td><pre>
    "extensions": [
      {
        "KHR_implicit_shapes": {
          "shapes": [
            {
              "type": "ellipsoid region",
              "extensions": {
                "EXT_implicit_ellipsoid_region": {
                  "semiMajorAxisRadius": 4,
                  "semiMinorAxisRadius": 2,
                  "minHeight": 0,
                  "maxHeight": 0.5
                }
              }
            }
          ]
        }
      }
    ]
    </pre></td>
    <td>
    **TODO** visual example
    </td>
  </tr>
</table>

An ellipsoid region may also be confined to a specific latitude and/or longitude range. The `minLatitude` and `maxLatitude` properties represent the latitude values at which the region starts and stops, defined in the range `[-pi/2, pi/2]`. Similarly, the `minLatitude` and `maxLatitude` properties represent the longitude bounds within the range `[-pi, pi]`.

<table>
  <tr>
    <th>
    Example
    </th>
  </tr>
  <tr>
    <td><pre>
    "extensions": [
      {
        "KHR_implicit_shapes": {
          "shapes": [
            {
              "type": "ellipsoid region",
              "extensions": {
                "EXT_implicit_ellipsoid_region": {
                  "semiMajorAxisRadius": 4,
                  "semiMinorAxisRadius": 2,
                  "minHeight": 0,
                  "maxHeight": 0.5,
                  // TODO
                }
              }
            }
          ]
        }
      }
    ]
    </pre></td>
    <td>
    **TODO** visual example
    </td>
  </tr>
</table>

## Optional vs. Required
This extension is required, meaning it should be placed in both the `extensionsUsed` list and `extensionsRequired` list.