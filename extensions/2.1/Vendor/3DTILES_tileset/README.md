# 3DTILES\_tileset

## Contributors

- Sean Lilley, Cesium
- Adam Morris, Cesium
- Tamrat Belayneh, ESRI

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

## Overview

This extension specifies a well defined subset of glTF 2.1 for representing a tileset in [3D Tiles](https://github.com/CesiumGS/3d-tiles/tree/main/specification).

It uses the glTF node hierarchy and glTF 2.1 features like external assets and bounding volumes to define a Hierarchical Level of Detail (HLOD) structure for streaming massive 3D scenes.

## File Extensions

Assets using this extension SHOULD use the `.tileset.gltf` or `.tileset.glb` file extensions. This helps differentiate tileset files from content files.

When using [External Tilesets](#external-tilesets), the root tileset SHOULD be named `root.tileset.gltf`.

## Constraints

When this extension is used the following constraints apply to the glTF:

- The glTF must have a single root tile as specified by [GODOT_single_root]().

The following constraints apply to all nodes:

- The `3DTILES_tileset` extension MUST be defined
- The `"mesh"` property MUST NOT be defined.
- The `"boundingVolume"` property MUST be defined.
- The bounding volume shape type MUST be `"box"` or `"sphere"`
  - Additional shapes types may be supported through extensions

## Concepts

### Overview

In 3D Tiles, a _tileset_ is a set of _tiles_ organized in a spatial data structure, the _tree_. Each tile may reference renderable _content_.

The content references a set of _features_, such as 3D models representing buildings or trees, or points in a point cloud. Each feature has position and appearance properties and additional application-specific properties. A client may choose to select features at runtime and retrieve their properties for visualization or analysis.

Tiles are organized in a tree which incorporates the concept of Hierarchical Level of Detail (HLOD) for optimal rendering of spatial data. Each tile has a _bounding volume_, an object defining a spatial extent completely enclosing its content. The tree has [spatial coherence](https://github.com/CesiumGS/3d-tiles/tree/main/specification#core-bounding-volume-spatial-coherence); the content for child tiles are completely inside the parent’s bounding volume.

A tileset may use a 2D spatial tiling scheme similar to raster and vector tiling schemes (like a Web Map Tile Service (WMTS) or XYZ scheme) that serve predefined tiles at several levels of detail (or zoom levels). However since the content of a tileset is often non-uniform or may not easily be organized in only two dimensions, the tree can be any spatial data structure with spatial coherence, including k-d trees, quadtrees, octrees, and grids. [Implicit tiling](#implicit-tiling) defines a concise representation of quadtrees and octrees.

Application-specific _metadata_ may be provided at multiple granularities within a tileset. Properties may be associated with high-level entities like tilesets, tiles, contents, or features, or with individual vertices and texels. Metadata conforms to a well-defined type system described by the [3D Metadata Specification](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/README.adoc#metadata-3d-metadata-specification), which may be extended with application- or domain-specific semantics.

Optionally a [3D Tiles Style](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Styling/README.adoc#styling-styling), or _style_, may be applied to a tileset. A style defines expressions to be evaluated which modify how each feature is displayed.

### Tileset

A tileset is a set of tiles organized in a spatial data structure, the tree. The tree has a single root tile. Tiles are represented as nodes in the glTF node hierarchy, or defined implicitly with [Implicit Tiling](#implicit-tiling).

3D Tiles uses one main tileset file as the entry point to define a tileset. To create a tree of trees, a tileset may also reference [external tilesets](#external-tilesets).


The following example shows a tree with a root tile and a child tile.

```json
{
  "extensionsUsed": ["3DTILES_tileset"],
  "asset": {
    "version": "2.1"
  },
  "scenes": [
    {
      "nodes": [0]
    }
  ],
  "scene": 0,
  "shapes": [
    {
      "type": "box",
      "box": {
        "size": [1.0, 1.0, 1.0]
      }
    },
    {
      "type": "box",
      "box": {
        "size": [0.9, 0.3, 1.0]
      }
    }
  ],
  "assets": [
    {
      "uri": "root.glb"
    },
    {
      "uri": "child.glb"
    }
  ],
  "extensions": {
    "3DTILES_tileset": {
      "geometricError": 240
    }
  },
  "nodes": [
    {
      "boundingVolume": {
        "shape": 0
      },
      "extensions": {
        "3DTILES_tileset": {
          "geometricError": 70.0,
          "refine": "ADD"
        }
      },
      "asset": 0,
      "children": [1],
    },
    {
      "boundingVolume": {
        "shape": 1
      },
      "extensions": {
        "3DTILES_tileset": {
          "geometricError": 0.0,
        }
      },
      "asset": 1
    },
  ]
}
```

The top-level `3DTILES_tileset` extension has the following properties:

`geometricError` is a nonnegative number that defines the error, in meters, that determines if the tileset is rendered. At runtime, the geometric error is used to compute _Screen-Space Error_ (SSE), the error measured in pixels. If the SSE does not exceed a required minimum, the tileset should not be rendered, and none of its tiles should be considered for rendering, see [Geometric error](#geometric-error).

### Tile

Tiles consist of metadata used to determine if a tile is rendered, a reference to the renderable content, and an array of any children tiles.

The following example shows the root tile above.

```json
{
  "boundingVolume": {
    "shape": 0
  },
  "extensions": {
    "3DTILES_tileset": {
      "geometricError": 70.0,
      "refine": "ADD"
    }
  },
  "asset": 0,
  "children": [1],
}
```

The `boundingVolume` defines a volume enclosing the tile, and is used to determine which tiles to render at runtime. The bounding volume may define a local space transform of the shape by supplying any of `translation`, `rotation`, and `scale` properties (not shown above).

The `geometricError` property is a nonnegative number that defines the error, in meters, introduced if this tile is rendered and its children are not. At runtime, the geometric error is used to compute _Screen-Space Error_ (SSE), the error measured in pixels. The SSE determines if a tile is sufficiently detailed for the current view or if its children should be considered, see [Geometric error]().

The `refine` property is a string that is either `"REPLACE"` for replacement refinement or `"ADD"` for additive refinement. It is required for the root tile of a tileset; it is optional for all other tiles. A tileset can use any combination of additive and replacement refinement. When the `refine` property is omitted, it is inherited from the parent tile.

The optional `asset` property provides a reference to the tile's content. When `asset` is not defined the tile is considered to be an _empty tile_.

The asset object may have an optional [bounding volume]() similar to the tile `boundingVolume`. But unlike the tile bounding volume, the asset bounding volume is a tightly fitting bounding volume enclosing just the tile's content.

The tile may defined a local space transform by supplying a `matrix` property, or any of `translation`, `rotation`, and `scale` properties. These properties MUST either be not defined or have default values for the root tile.

The `children` property is an array of node indices to child tiles. Each child tile's content is fully enclosed by its parent tile's `boundingVolume`. For _leaf tiles_, there are no children, and `children` MAY not be defined.

### Geometric Error

Tiles are structured into a tree incorporating _Hierarchical Level of Detail_ (HLOD) so that at runtime a client implementation will need to determine if a tile is sufficiently detailed for rendering and if the content of tiles should be successively refined by children tiles of higher resolution. An implementation will consider a maximum allowed _Screen-Space Error_ (SSE), the error measured in pixels.

A tile’s geometric error defines the selection metric for that tile. Its value is a nonnegative number that defines the error, in meters, introduced if this tile is rendered and its children are not.

Generally, the root tile will have the largest geometric error, and each successive level of children will have a smaller geometric error than its parent, with leaf tiles having a geometric error of or close to 0. If the child tile's geometric error is greater than or equal to its parent's geometric error, the tile is considered _unconditionally refinable_.

In a client implementation, geometric error is used with other screen space metrics—​e.g., distance from the tile to the camera, screen size, and resolution-- to calculate the SSE introduced if this tile is rendered and its children are not. If the introduced SSE exceeds the maximum allowed, then the tile is refined and its children are considered for rendering.

The geometric error is formulated based on a metric like point density, mesh or texture decimation, or another factor specific to that tileset. In general, a higher geometric error means a tile will be refined more aggressively, and children tiles will be loaded and rendered sooner.

### Refinement

Refinement determines the process by which a lower resolution parent tile renders when its higher resolution children are selected to be rendered. Permitted refinement types are replacement (`"REPLACE"`) and additive (`"ADD"`). If the tile has replacement refinement, the children tiles are rendered in place of the parent, that is, the parent tile is no longer rendered. If the tile has additive refinement, the children are rendered in addition to the parent tile.

A tileset can use replacement refinement exclusively, additive refinement exclusively, or any combination of additive and replacement refinement.

A refinement type is required for the root tile of a tileset; it is optional for all other tiles. When omitted, a tile inherits the refinement type of its parent.

##### Replacement

If a tile uses replacement refinement, when refined it renders its children in place of itself.

| Parent Tile | Refined |
|:---:|:--:|
| ![](figures/replacement_1.jpg) | ![](figures/replacement_2.jpg) |

##### Additive

If a tile uses additive refinement, when refined it renders itself and its children simultaneously.

| Parent Tile | Refined |
|:---:|:--:|
| ![](figures/additive_1.jpg) | ![](figures/additive_2.jpg) |

### Bounding Volumes

A bounding volume defines the spatial extent enclosing a tile or a tile’s content. The bounding volume shape MUST be `"box"` or `"sphere"`.

Additional shapes may be supported through extensions such as

- [3DTILES_shape_ellipsoid_region]()
- [3DTILES_shape_cylinder_region]()
- [3DTILES_shape_S2]()


#### Bounding Box

The following example shows an oriented bounding box.

```json
{
  "shapes": [
    {
      "type": "box",
      "box": {
        "size": [1.0, 1.0, 1.0]
      }
    }
  ],
  "nodes": [
    {
    "boundingVolume": {
      "shape": 0,
      "translation": [-3923021.73, -931070.70, 4925458.17],
      "rotation": [0.26262, -0.20758, -0.58433, 0.73925],
      "scale": [100.0, 100.0, 100.0]
    },
    ...
    }
  ]
}
```

#### Bounding Sphere

The following example shows a bounding sphere encompassing the Earth.

```json
{
  "shapes": [
    {
      "type": "sphere",
      "sphere": {
        "radius": 6378137.0
      }
    }
  ],
  "nodes": [
    {
    "boundingVolume": {
      "shape": 0
    },
    ...
    }
  ]
}
```

### Spatial Coherence

### Coordinate reference systems

- `EXT_geospatial_crs` for geospatial coordinate reference system definitions
- Forward axis, how does this affect bv's 
- Geopose on root node instead of supply arbitrary 4x4 transform

### Implicit Tiling

- `3DTILES_implicit_tiling`
- `3DTILES_subtree` for concise representation of quadtrees and octrees.
- Implicit tiling derived transforms

### External Tilesets

- `3DTILES_external_tileset`
- How to indicate that `asset` is an external tileset?

### Metadata

- `EXT_structural_metadata` for assigning properties to tileset, tile, content, etc
- "Metadata" vs. "properties"
- Fold 3D Metadata Specification into EXT_structural_metadata (or KHR_structural_metadata)?

### Styling

## TODO

- Inner vs. outer bounding volume
- Unconditional refinement
- viewerRequestVolume - related to behaviors and interactivity?
- Better picture for replacement refinement
- Geometric error image from reference card
