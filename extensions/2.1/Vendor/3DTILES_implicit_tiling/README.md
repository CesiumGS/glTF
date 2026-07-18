<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# 3DTILES\_implicit\_tiling

## Contributors

- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [3DTILES_tileset](../3DTILES_tileset/README.md).

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

Implicit tiling defines a concise representation of quadtrees and octrees in 3D Tiles. This regular pattern allows for random access of tiles based on their tile coordinates which enables accelerated spatial queries, new traversal algorithms, and efficient updates of tile content, among other use cases.

Implicit tiling also allows for better interoperability with existing GIS data formats with implicitly defined tiling schemes. Some examples are [TMS](https://wiki.osgeo.org/wiki/Tile_Map_Service_Specification), [WMTS](https://www.ogc.org/standards/wmts), [S2](http://s2geometry.io/), and [CDB](https://docs.opengeospatial.org/is/15-113r5/15-113r5.html).

In order to support sparse datasets, *availability* data determines which tiles exist. To support massive datasets, availability is partitioned into fixed-size *subtrees*. Subtrees may store *properties* for available tiles and contents.

The `3DTILES_implicit_tiling` extension may be added to any tile in the tileset. The extension defines how the tile is subdivided and where to locate content resources. It may be added to multiple tiles to create more complex subdivision schemes.

<p align="center">
  <img src="./figures/sparse-octree.png"/><br/>
  <em>A point cloud organized into a sparse octree. Data source: Trimble.</em>
</p>

## Implicit Root Tile

The `3DTILES_implicit_tiling` extension may be added to any tile in the tileset. Such a tile is called an *implicit root tile*, to distinguish it from the root tile of the tileset. The implicit root tile is [unconditionally refinable](../3DTILES_tileset/README.md#unconditional-refinement).

```json
{
  "extensions": {
    "3DTILES_tileset": {
      "geometricError": 5000.0,
      "refine": "REPLACE"
    },
    "3DTILES_implicit_tiling": {
      "contentUri": "content/{level}/{x}/{y}.glb",
      "subtreeUri": "subtrees/{level}/{x}/{y}.subtree.glb",
      "subdivisionScheme": "QUADTREE",
      "availableLevels": 21,
      "subtreeLevels": 7
    }
  },
  "boundingVolume": {
    "shape": 0
  }
}
```

The `3DTILES_implicit_tiling` extension has the following properties:

Property | Description
--|--
`contentUri`|[Template URI]((#template-uris)) for locating content files.
`subtreeUri`|[Template URI]((#template-uris)) for locating subtree files.
`subdivisionScheme`|Either `QUADTREE` or `OCTREE`. See [Subdivision scheme](#subdivision-scheme).
`availableLevels`|How many levels there are in the tree with available tiles.
`subtreeLevels`|How many levels there are in each subtree.

The following constraints apply to implicit root tiles:

- The `children` property **MUST NOT** be defined
- The `externalAsset` property **MUST NOT** be defined
- The contents referenced by `contentUri` **MUST NOT** be [external tilesets](../3DTILES_tileset/README.md#external-tilesets)

## Subdivision Scheme

A *subdivision scheme* is a recursive pattern for dividing a bounding volume of a tile into smaller children tiles that take up the same space.

A *quadtree* divides space only on the `x` and `y` dimensions. It divides each tile into 4 smaller tiles where the `x` and `y` dimensions are halved. The quadtree `z` minimum and maximum remain unchanged. The resulting tree has 4 children per tile.

<p align="center">
  <img src="./figures/quadtree.png"/><br/>
</p>

An *octree* divides space along all 3 dimensions. It divides each tile into 8 smaller tiles where each dimension is halved. The resulting tree has 8 children per tile.

<p align="center">
  <img src="./figures/octree.png"/><br/>
</p>

Sphere bounding volumes are disallowed, as these cannot be divided into a quadtree or octree.

- For subdivision of ellipsoid region bounding volumes refer to  [3DTILES_shape_ellipsoid_region](../3DTILES_shape_ellipsoid_region/README.md#implicit-subdivision)
- For subdivision of cylinder region bounding volumes refer to [3DTILES_shape_cylinder_region](../3DTILES_shape_cylinder_region/README.md#implicit-subdivision)
- For subdivision of S2 bounding volumes refer to [3DTILES_bounding_volume_S2](../3DTILES_bounding_volume_S2/README.md#implicit-subdivision)

## Subdivision Rules

Implicit tiling only requires defining the subdivision scheme, refinement strategy, bounding volume, and geometric error at the implicit root tile. For descendant tiles, these properties are computed automatically, based on the following rules:

Subdivision rules for implicit tiling:

Property|Subdivision Rule
--|--
`subdivisionScheme`|Constant for all descendant tiles.
`refine`|Constant for all descendant tiles.
`boundingVolume`|Divided into four or eight parts depending on the `subdivisionScheme`.
`geometricError`|Each child's `geometricError` is half of its parent's `geometricError`.

> [!NOTE]
> In order to maintain numerical stability during this subdivision process, the actual bounding volumes should not be computed progressively by subdividing a non-root tile volume. Instead, the exact bounding volumes should be computed directly for a given level.
>
> Let the extent of the root bounding volume along one dimension *d* be *(min<sub>d</sub>, max<sub>d</sub>)*. The number of bounding volumes along that dimension for a given level  is *2<sup>level</sup>*. The size of each bounding volume at this level, along dimension *d*, is *size<sub>d</sub> = (max<sub>d</sub> - min<sub>d</sub>) / 2<sup>level</sup>*. The extent of the bounding volume of a child can then be computed directly as *(min<sub>d</sub> + size<sub>d</sub> * i, min<sub>d</sub> + size<sub>d</sub> * (i + 1))*, where *i* is the index of the child in dimension *d*.

The computed tile `boundingVolume` and `geometricError` can be overridden with [tile semantics](#tile-semantics), if desired. Content bounding volumes are not computed automatically but they may be provided by [content semantics](#content-semantics). Tile and content bounding volumes **SHOULD** maintain [spatial coherence](../3DTILES_tileset/README.md#spatial-coherence).

## Tile Coordinates

*Tile coordinates* are a tuple of integers that uniquely identify a tile. Tile coordinates are either `(level, x, y)` for quadtrees or `(level, x, y, z)` for octrees. All tile coordinates are 0-indexed.

`level` is 0 for the implicit root tile. This tile's children are at level 1, and so on.

`x`, `y`, and `z` coordinates define the location of the tile within the level.

For `box` bounding volumes:

Coordinate|Positive Direction
--|--
`x`|Along the `+x` axis of the bounding box
`y`|Along the `+y` axis of the bounding box
`z`|Along the `+z` axis of the bounding box

![](./figures/box-coordinates.png)

## Template URIs

A *Template URI* is a URI pattern used to refer to tiles by their tile coordinates.

Template URIs **MAY** include the variables `{level}`, `{x}`, `{y}`. Template URIs for octrees **MAY** also include `{z}`. When referring to a specific tile, the tile's coordinates are substituted for these variables.

Template URIs, when given as relative paths, are resolved relative to the tileset JSON file.

![](./figures/template-uri.png)

## Subtrees

In order to support sparse datasets, additional information is needed to indicate which tiles or contents exist. This is called *availability*.

*Subtrees* are fixed size sections of the tileset tree used for storing availability. The tileset is partitioned into subtrees to bound the size of each availability buffer for optimal network transfer and caching. The `subtreeLevels` property defines the number of levels in each subtree. The subdivision scheme determines the number of children per tile.

![](./figures/subtree-anatomy.png)

After partitioning a tileset into subtrees, the result is a tree of subtrees.

![](./figures/subtree-tree.png)

### Availability

Each subtree contains tile availability, content availability, and child subtree availability.

- *Tile availability* indicates which tiles exist within the subtree
- *Content availability* indicates which tiles have associated content resources
- *Child subtree availability* indicates what subtrees are reachable from this subtree

Each type of availability is represented as a separate bitstream. Each bitstream is a 1D array where each element represents a node in the quadtree or octree. A 1 bit indicates that the element is available, while a 0 bit indicates that the element is unavailable. Alternatively, if all the bits in a bitstream are the same, a single constant value can be used instead.

To form the 1D bitstream, the tiles are ordered with the following rules:

- Within each level of the subtree, the tiles are ordered using the [Morton Z-order curve](https://en.wikipedia.org/wiki/Z-order_curve)
- The bits for each level are concatenated into a single bitstream

![](./figures/availability-ordering.png)


In the diagram above, colored cells represent 1 bits, grey cells represent 0 bits.

Storing tiles in Morton order provides these benefits:

- Efficient indexing - The Morton index for a tile is computed in constant time by interleaving bits.
- Efficient traversal - The Morton index for a parent or child tile are computed in constant time by removing or adding bits, respectively.
- Locality of reference - Consecutive tiles are near to each other in 3D space.
- Better Compression - Locality of reference leads to better compression of availability bitstreams.

For more detailed information about working with Morton indices and availability bitstreams, see [Availability Indexing](./AVAILABILITY.md).

#### Tile Availability

Tile availability determines which tiles exist in a subtree.

Tile availability has the following restrictions:

- If a non-root tile's availability is 1, its parent tile's availability **MUST** also be 1.
- A subtree **MUST** have at least one available tile.

![](./figures/tile-availability.png)

#### Content Availability

Content availability determines which tiles have a content resource. The content resource is located using the template URI of the tile content. If there are no tiles with a content resource, then `contentUri` **MUST** be omitted.

Content availability has the following restrictions:

- If content availability is 1 its corresponding tile availability **MUST** also be 1. Otherwise, it would be possible to specify content files that are not reachable by the tiles of the tileset.
- If content availability is 0 and its corresponding tile availability is 1 then the tile is considered to be an empty tile.

![](./figures/content-availability.png)

#### Child Subtree Availability

Child subtree availability determines which subtrees are reachable from the deepest level of this subtree. This links subtrees together to form a tree.

Unlike tile and content availability, which store bits for every level in the subtree, child subtree availability stores bits for nodes one level deeper than the deepest level of the subtree, and represent the root nodes of child subtrees. This is used to determine which other subtrees are reachable before requesting tiles. If availability is 0 for all child subtrees, then the tileset does not subdivide further.

![](./figures/child-subtree-availability.png)

### Semantics

Tile properties such as bounding volume, refinement type, and geometric error that are computed automatically based on [Subdivision Rules](#subdivision-rules) but may be overridden by values stored in the subtree.

In particular, `tileBoundingBox`, `tileBoundingSphere`, `tileBoundingRegion`, `tileMinimumHeight`, and `tileMaximumHeight` semantics may define a tighter bounding volume for a tile than is implicitly calculated.

> [!NOTE]
>
> The following diagram shows how tile height semantics may be used to define tighter bounding regions for an implicit tileset: The overall height of the bounding region of the whole tileset is 320. The bounding regions for the child tiles will be computed by splitting the bounding regions of the respective parent tile at its center. By default, the height will remain constant. By storing the _actual_ height of the contents in the respective region, and providing it as the `tileMaximumHeight` for each available tile, it is possible to define the tightest-fitting bounding region for each level.
>
> ![](figures/tile-height-semantics.png)

### Properties

Subtrees may store application-specific properties at multiple granularities.

- *Tile properties* - properties for available tiles in the subtree
- *Content properties* - properties for available content in the subtree

#### Tile Properties

When tiles are listed explicitly within a tileset, each tile's properties are also embedded explicitly within the tile definition. When the tile hierarchy is _implicit_, as enabled by implicit tiling, tiles are not listed exhaustively and properties cannot be directly embedded in tile definitions. To support properties for tiles within implicit tiling schemes, property values for all available tiles in a subtree are encoded in a [property table](https://github.com/CesiumGS/glTF/tree/3d-tiles-next/extensions/2.0/Vendor/EXT_structural_metadata#property-tables) with [EXT_structural_metadata](https://github.com/CesiumGS/glTF/tree/3d-tiles-next/extensions/2.0/Vendor/EXT_structural_metadata). The binary representation is particularly efficient for larger datasets with many tiles.

Property values exist only for available tiles and are tightly packed by an increasing tile index according to the [Availability Ordering](#availability). Each available tile **MUST** have a value — representation of missing values within a tile is possible only with the `noData` indicator defined by a [EXT_structural_metadata](https://github.com/CesiumGS/glTF/tree/3d-tiles-next/extensions/2.0/Vendor/EXT_structural_metadata) schema.

> [!NOTE]
>
> To determine the index into a property value array for a particular tile, count the number of available tiles occurring before that index, according to the tile Availability Ordering. If `i` available tiles occur before a particular tile, that tile's property values are stored at index `i` of each property value array. These indices may be precomputed for all available tiles, as a single pass over the subtree availability buffer.

#### Content Properties

Subtrees may also store properties for tile content. Property values exist only for available content and are tightly packed by increasing tile index.

## TODO

- Template URI bypasses glTF 2.1 `files` mechanism, which would prevent [packaging external assets](https://github.com/KhronosGroup/glTF/issues/2589).
- Incorporate [3DTILES_implicit_tiling_custom_template_variables](https://github.com/CesiumGS/3d-tiles/pull/815)