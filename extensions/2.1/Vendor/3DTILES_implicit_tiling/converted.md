# Implicit Tiling

## Overview

Implicit tiling defines a concise representation of quadtrees and octrees in 3D Tiles. This regular pattern allows for random access of tiles based on their tile coordinates which enables accelerated spatial queries, new traversal algorithms, and efficient updates of tile content, among other use cases.

Implicit tiling also allows for better interoperability with existing GIS data formats with implicitly defined tiling schemes. Some examples are [TMS](https://wiki.osgeo.org/wiki/Tile_Map_Service_Specification), [WMTS](https://www.ogc.org/standards/wmts), [S2](http://s2geometry.io/), and [CDB](https://docs.opengeospatial.org/is/15-113r5/15-113r5.html).

In order to support sparse datasets, *availability* data determines which tiles exist. To support massive datasets, availability is partitioned into fixed-size *subtrees*. Subtrees may store *metadata* for available tiles and contents.

An `implicitTiling` object may be added to any tile in the tileset JSON. The object defines how the tile is subdivided and where to locate content resources. It may be added to multiple tiles to create more complex subdivision schemes.

![](figures/sparse-octree.png)

_A point cloud organized into a sparse octree. Data source: Trimble_

## Implicit Root Tile

An `implicitTiling` object may be added to any tile in the tileset JSON. Such a tile is called an *implicit root tile*, to distinguish it from the root tile of the tileset JSON.

```json
{
  "root": {
    "boundingVolume": {
      "region": [-1.318, 0.697, -1.319, 0.698, 0, 20]
    },
    "refine": "REPLACE",
    "geometricError": 5000,
    "content": {
      "uri": "content/{level}/{x}/{y}.glb"
    },
    "implicitTiling": {
      "subdivisionScheme": "QUADTREE",
      "availableLevels": 21,
      "subtreeLevels": 7,
      "subtrees": {
        "uri": "subtrees/{level}/{x}/{y}.json"
      }
    }
  }
}
```

The `implicitTiling` object has the following properties:

Property | Description
--|--
`subdivisionScheme`|Either `QUADTREE` or `OCTREE`. See [Subdivision scheme](#subdivision-scheme).
`availableLevels`|How many levels there are in the tree with available tiles.
`subtreeLevels`|How many levels there are in each subtree.
`subtrees`|Template URI for subtree files. See [Subtrees](#subtrees).

[Template URIs](#template-uris) are used for locating subtree files as well as tile contents. For content, the template URI is specified in the tile's `content.uri` property.

The following constraints apply to implicit root tiles:

* The tile shall omit the `children` property
* The tile shall omit the `metadata` property
* The `content.uri` shall not point to an [external tileset](../3DTILES_external_tileset/README.md)
* The `content` shall omit the `boundingVolume` property

## Subdivision Scheme

A *subdivision scheme* is a recursive pattern for dividing a bounding volume of a tile into smaller children tiles that take up the same space.

A *quadtree* divides space only on the `x` and `y` dimensions. It divides each tile into 4 smaller tiles where the `x` and `y` dimensions are halved. The quadtree `z` minimum and maximum remain unchanged. The resulting tree has 4 children per tile.

![](figures/quadtree.png)

An *octree* divides space along all 3 dimensions. It divides each tile into 8 smaller tiles where each dimension is halved. The resulting tree has 8 children per tile.

![](figures/octree.png)

For a `region` bounding volume, `x`, `y`, and `z` refer to `longitude`, `latitude`, and `height` respectively.

Sphere bounding volumes are disallowed, as these cannot be divided into a quadtree or octree.

For subdivision of S2 bounding volumes refer to [3DTILES_bounding_volume_S2](https://github.com/CesiumGS/3d-tiles/tree/main/extensions/3DTILES_bounding_volume_S2/README.md#implicit-subdivision).

The following diagrams illustrate the subdivision in the bounding volume types supported by 3D Tiles:

Root Box | Quadtree | Octree
--|--|--
![](figures/box.png)|![](figures/box-quadtree.png)|![](figures/box-octree.png)

Root Region | Quadtree | Octree
--|--|--
![](figures/region.png)|![](figures/region-quadtree.png)|![](figures/region-octree.png)


### Subdivision Rules

Implicit tiling only requires defining the subdivision scheme, refinement strategy, bounding volume, and geometric error at the implicit root tile. For descendant tiles, these properties are computed automatically, based on the following rules:

Property | Subdivision Rule
--|--
`subdivisionScheme`|Constant for all descendant tiles
`refine`|Constant for all descendant tiles
`boundingVolume`|Divided into four or eight parts depending on the `subdivisionScheme`
`geometricError`|Each child's `geometricError` is half of its parent's `geometricError`

> **Informative**
>
>In order to maintain numerical stability during this subdivision process, the actual bounding volumes should not be computed progressively by subdividing a non-root tile volume. Instead, the exact bounding volumes should be computed directly for a given level.
>
> Let the extent of the root bounding volume along one dimension _d_ be _(min<sub>d</sub>, max<sub>d</sub>)_. The number of bounding volumes along that dimension for a given level  is _2<sup>level</sup>_. The size of each bounding volume at this level, along dimension _d_, is _size<sub>d</sub> = (max<sub>d</sub> - min<sub>d</sub>) / 2<sup>level</sup>_. The extent of the bounding volume of a child can then be computed directly as _(min<sub>d</sub> + size<sub>d</sub> * i, min<sub>d</sub> + size<sub>d</sub> * (i + 1))_, where _i_ is the index of the child in dimension _d_.

The computed tile `boundingVolume` and `geometricError` can be overridden with [tile metadata](#tile-metadata), if desired. Content bounding volumes are not computed automatically but they may be provided by [content metadata](#content-metadata). Tile and content bounding volumes shall maintain [spatial coherence](../3DTILES_tileset/README.md#spatial-coherence).

## Tile Coordinates

*Tile coordinates* are a tuple of integers that uniquely identify a tile. Tile coordinates are either `(level, x, y)` for quadtrees or `(level, x, y, z)` for octrees. All tile coordinates are 0-indexed.

`level` is 0 for the implicit root tile. This tile's children are at level 1, and so on.

`x`, `y`, and `z` coordinates define the location of the tile within the level.

For `box` bounding volumes:

Coordinate | Positive Direction
--|--
`x`|Along the `+x` axis of the bounding box
`y`|Along the `+y` axis of the bounding box
`z`|Along the `+z` axis of the bounding box

![](figures/box-coordinates.png)

For `region` bounding volumes:

Coordinate | Positive Direction
--|--
`x`|From west to east (increasing longitude)
`y`|From south to north (increasing latitude)
`z`|From bottom to top (increasing height)

![](figures/region-coordinates.png)

## Template URIs

A *Template URI* is a URI pattern used to refer to tiles by their tile coordinates.

Template URIs shall include the variables `+{level}+`, `+{x}+`, `+{y}+`. Template URIs for octrees shall also include `+{z}+`. When referring to a specific tile, the tile's coordinates are substituted for these variables.

Template URIs, when given as relative paths, are resolved relative to the tileset JSON file.

![](figures/template-uri.png)

## Subtrees

In order to support sparse datasets, additional information is needed to indicate which tiles or contents exist. This is called *availability*.

*Subtrees* are fixed size sections of the tileset tree used for storing availability. The tileset is partitioned into subtrees to bound the size of each availability buffer for optimal network transfer and caching. The `subtreeLevels` property defines the number of levels in each subtree. The subdivision scheme determines the number of children per tile.

![](figures/subtree-anatomy.png)

After partitioning a tileset into subtrees, the result is a tree of subtrees.

![](figures/subtree-tree.png)

### Availability

Each subtree contains tile availability, content availability, and child subtree availability.

* *Tile availability* indicates which tiles exist within the subtree
* *Content availability* indicates which tiles have associated content resources
* *Child subtree availability* indicates what subtrees are reachable from this subtree

Each type of availability is represented as a separate bitstream. Each bitstream is a 1D array where each element represents a node in the quadtree or octree. A 1 bit indicates that the element is available, while a 0 bit indicates that the element is unavailable. Alternatively, if all the bits in a bitstream are the same, a single constant value can be used instead.

To form the 1D bitstream, the tiles are ordered with the following rules:

* Within each level of the subtree, the tiles are ordered using the [Morton Z-order curve](https://en.wikipedia.org/wiki/Z-order_curve)
* The bits for each level are concatenated into a single bitstream

![](figures/availability-ordering.png)

In the diagram above, colored cells represent 1 bits, grey cells represent 0 bits.

Storing tiles in Morton order provides these benefits:

* Efficient indexing - The Morton index for a tile is computed in constant time by interleaving bits.
* Efficient traversal - The Morton index for a parent or child tile are computed in constant time by removing or adding bits, respectively.
* Locality of reference - Consecutive tiles are near to each other in 3D space.
* Better Compression - Locality of reference leads to better compression of availability bitstreams.

For more detailed information about working with Morton indices and availability bitstreams, see [Availability Indexing](./AVAILABILITY.md).

#### Tile Availability

Tile availability determines which tiles exist in a subtree.

Tile availability has the following restrictions:

* If a non-root tile's availability is 1, its parent tile's availability shall also be 1.
* A subtree shall have at least one available tile.

![](figures/tile-availability.png)

#### Content Availability

Content availability determines which tiles have a content resource. The content resource is located using the template URI of the tile content. When the tile has [multiple contents](https://github.com/CesiumGS/3d-tiles/blob/main/specification/README.adoc#core-tile-content), then there is one content availability bitstream for each content. If there are no tiles with a content resource, then `tile.content` and `tile.contents` shall be omitted.

Content availability has the following restrictions:

* If content availability is 1 its corresponding tile availability shall also be 1. Otherwise, it would be possible to specify content files that are not reachable by the tiles of the tileset.
* If content availability is 0 and its corresponding tile availability is 1 then the tile is considered to be an empty tile.

![](figures/content-availability.png)

#### Child Subtree Availability

Child subtree availability determines which subtrees are reachable from the deepest level of this subtree. This links subtrees together to form a tree.

Unlike tile and content availability, which store bits for every level in the subtree, child subtree availability stores bits for nodes one level deeper than the deepest level of the subtree, and represent the root nodes of child subtrees. This is used to determine which other subtrees are reachable before requesting tiles. If availability is 0 for all child subtrees, then the tileset does not subdivide further.

![](figures/child-subtree-availability.png)

### Metadata

Subtrees may store metadata at multiple granularities.

* *Tile metadata* - metadata for available tiles in the subtree
* *Content metadata* - metadata for available content in the subtree
* *Subtree metadata* - metadata about the subtree as a whole

#### Tile Metadata

When tiles are listed explicitly within a tileset, each tile's metadata is also embedded explicitly within the tile definition. When the tile hierarchy is _implicit_, as enabled by implicit tiling, tiles are not listed exhaustively and metadata cannot be directly embedded in tile definitions. To support metadata for tiles within implicit tiling schemes, property values for all available tiles in a subtree are encoded in a [property table](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/ReferenceImplementation/PropertyTable/README.adoc#metadata-referenceimplementation-propertytable-property-table-implementation). The binary representation is particularly efficient for larger datasets with many tiles.

Tile metadata exists only for available tiles and is tightly packed by an increasing tile index according to the [Availability Ordering](#availability). Each available tile shall have a value -- representation of missing values within a tile is possible only with the `noData` indicator defined by the [*Binary Table Format*](https://github.com/CesiumGS/3d-tiles/tree/main/specification/Metadata#metadata-binary-table-format) specification.

> **Informative**
>
>To determine the index into a property value array for a particular tile, count the number of available tiles occurring before that index, according to the tile Availability Ordering. If `i` available tiles occur before a particular tile, that tile's property values are stored at index `i` of each property value array. These indices may be precomputed for all available tiles, as a single pass over the subtree availability buffer.

Tile properties can have [Semantics](https://github.com/CesiumGS/3d-tiles/tree/main/specification/Metadata/Semantics) which define how property values should be interpreted. In particular, `TILE_BOUNDING_BOX`, `TILE_BOUNDING_REGION`, `TILE_BOUNDING_SPHERE`, `TILE_MINIMUM_HEIGHT`, and `TILE_MAXIMUM_HEIGHT` semantics each define a more specific bounding volume for a tile than is implicitly calculated from implicit tiling. If more than one of these semantics are available for a tile, clients may select the most appropriate option based on use case and performance requirements.

> **Example**
>
>The following diagram shows how tile height semantics may be used to define tighter bounding regions for an implicit tileset: The overall height of the bounding region of the whole tileset is 320. The bounding regions for the child tiles will be computed by splitting the bounding regions of the respective parent tile at its center. By default, the height will remain constant. By storing the _actual_ height of the contents in the respective region, and providing it as the `TILE_MAXIMUM_HEIGHT` for each available tile, it is possible to define the tightest-fitting bounding region for each level.
>
>![](figures/tile-height-semantics.png)

The `TILE_GEOMETRIC_ERROR` semantic allows tiles to provide a geometric error that overrides the implicitly computed geometric error.

#### Content Metadata

Subtrees may also store metadata for tile content. Content metadata exists only for available content and is tightly packed by increasing tile index. Binary property values are encoded in a compact [*Binary Table Format*](https://github.com/CesiumGS/3d-tiles/tree/main/specification/Metadata#metadata-binary-table-format) defined by the 3D Metadata Specification and are stored in a [property table](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/ReferenceImplementation/PropertyTable/README.adoc#metadata-referenceimplementation-propertytable-property-table-implementation). If the implicit root tile has multiple contents then content metadata is stored in multiple property tables.

Content bounding volumes are not computed automatically by implicit tiling but may be provided by properties with semantics `CONTENT_BOUNDING_BOX`, `CONTENT_BOUNDING_REGION`, `CONTENT_BOUNDING_SPHERE`, `CONTENT_MINIMUM_HEIGHT`, and `CONTENT_MAXIMUM_HEIGHT`.

If the tile content is assigned to a [`group`](https://github.com/CesiumGS/3d-tiles/tree/main/specification#tile-content) then all contents in the implicit tree are assigned to that group.

#### Subtree Metadata

Properties assigned to subtrees provide metadata about the subtree as a whole. Subtree metadata is encoded in JSON according to the [JSON Format](https://github.com/CesiumGS/3d-tiles/tree/main/specification/Metadata#json-format) specification.

## Subtree JSON Format

*Defined in [`subtree.schema.json`](https://github.com/CesiumGS/3d-tiles/tree/main/specification/schema/Subtree/subtree.schema.json)*

A *subtree file* is a JSON file that contains availability and metadata information for a single subtree. A subtree may reference external files containing binary data. An alternative [Binary Format]() allows the JSON and binary data to be embedded into a single binary file.

### Buffers and Buffer Views

The [property table](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/ReferenceImplementation/PropertyTable/README.adoc#metadata-referenceimplementation-propertytable-property-table-implementation) defines the storage of metadata in a binary form based on _buffer views_ that are parts of a _buffer_.

A *buffer* is a binary blob. Each buffer has a `uri` that refers to an external file containing buffer data and a `byteLength` describing the buffer size in bytes. Relative paths are relative to the subtree file. Data URIs are not allowed.

In the [Binary Format]() the first buffer may instead refer to the binary chunk of the subtree file, in which case the `uri` property shall be undefined. This buffer is referred to as the _internal buffer_.

A *buffer view* is a contiguous subset of a buffer. A buffer view's `buffer` property is an integer index to identify the buffer. A buffer view has a `byteOffset` and a `byteLength` to describe the range of bytes within the buffer. The `byteLength` does not include any padding. There may be multiple buffer views referencing a single buffer.

For efficient memory access, the `byteOffset` of a buffer view shall be aligned to a multiple of 8 bytes.

### Availability

Tile availability (`tileAvailability`) and child subtree availability (`childSubtreeAvailability`) shall always be provided for a subtree.

Content availability (`contentAvailability`) is an array of content availability objects. If the implicit root tile has a single `content`, then this array will have one element; if the tile has multiple `contents`, then this array will have multiple elements. If the implicit root tile does not have a `content` or `contents` property, and therefore does not define a content template URI, then this means that none of the implicit tiles has any content, and the `contentAvailability` in the subtrees shall be omitted.

Availability may be represented either as a bitstream or a constant value. `bitstream` is an integer index that identifies the buffer view containing the availability bitstream. `constant` is an integer indicating whether all of the elements are available (`1`) or all are unavailable (`0`). `availableCount` is an integer indicating how many `1` bits exist in the availability bitstream.

Availability bitstreams are packed in binary using the format described in the [Booleans](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/README.adoc#metadata-booleans) section of the 3D Metadata Specification. Trailing bits of availability bitstreams that result from the number of nodes not being divisible by 8 shall be set to 0.

> **Example**
> >
> The JSON description of a subtree where each tile is available, but not all tiles have content, and not all child subtrees are available:
>
> ```json
> {
>   "buffers": [
>     {
>       "name": "Internal Buffer",
>       "byteLength": 16
>     },
>     {
>       "name": "External Buffer",
>       "uri": "external.bin",
>       "byteLength": 32
>     }
>   ],
>   "bufferViews": [
>     {
>       "buffer": 0,
>       "byteOffset": 0,
>       "byteLength": 11
>     },
>     {
>       "buffer": 1,
>       "byteOffset": 0,
>       "byteLength": 32
>     }
>   ],
>   "tileAvailability": {
>     "constant": 1,
>   },
>   "contentAvailability": [{
>     "bitstream": 0,
>     "availableCount": 60
>   }],
>   "childSubtreeAvailability": {
>     "bitstream": 1
>   }
> }
> ```
>
> The tile availability can be encoded by setting `tileAvailability.constant` to `1`, without needing an explicit bitstream, because all tiles in the subtree are available.
>
> Only some tiles have content, and `contentAvailability.bitstream` is the index of the buffer view where the bitstream for the content availability is stored: The `bufferView` with index 0 refers to the `buffer` with index 0. This buffer does not have a `uri` property, and therefore refers to the _internal_ buffer that is stored directly in the binary chunk of the subtree binary file. The `byteOffset` and `byteLength` indicate that the content availability bitstream is stored in the bytes `+[0...11)+` of the internal buffer.
>
> Some child subtrees exist, so `childSubtreeAvailability.bitstream` is the index of a buffer view that contains another bitstream. The `bufferView` with index 1 refers to the buffer with index `1`. This buffer has a `uri` property, indicating that this second bitstream is stored in an external binary file.

### Metadata

Subtrees may store metadata at multiple granularities. `tileMetadata` is a property table containing metadata for available tiles. `contentMetadata` is an array of property tables containing metadata for available content. If the implicit root tile has a single `content`, then this array will have one element; if the tile has multiple `contents`, then this array will have multiple elements. If the implicit root tile does not have a `content` or `contents` property, then none of the implicit tiles has any content, and the `contentMetadata` shall be omitted.

Subtree metadata (`subtreeMetadata`) is encoded in JSON according to the [JSON Format](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/README.adoc#metadata-json-format) specification.

> **Example**
>
> The same JSON description of a subtree extended with tile, content, and subtree metadata. The subtree JSON refers to a class ID in the root tileset schema. Tile and content metadata is stored in [property table](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/ReferenceImplementation/PropertyTable/README.adoc#metadata-referenceimplementation-propertytable-property-table-implementation); subtree metadata is encoded directly in JSON.
>
>_Schema in the root tileset JSON_
>
>```json
> {
>   "schema": {
>     "classes": {
>       "tile": {
>         "properties": {
>           "horizonOcclusionPoint": {
>             "semantic": "TILE_HORIZON_OCCLUSION_POINT",
>             "type": "VEC3",
>             "componentType": "FLOAT64",
>           },
>           "countries": {
>             "description": "Countries a tile intersects",
>             "type": "STRING",
>             "array": true
>           }
>         }
>       },
>       "content": {
>         "properties": {
>           "attributionIds": {
>             "semantic": "ATTRIBUTION_IDS",
>             "type": "SCALAR",
>             "componentType": "UINT16",
>             "array": true
>           },
>           "minimumHeight": {
>             "semantic": "CONTENT_MINIMUM_HEIGHT",
>             "type": "SCALAR",
>             "componentType": "FLOAT64"
>           },
>           "maximumHeight": {
>             "semantic": "CONTENT_MAXIMUM_HEIGHT",
>             "type": "SCALAR",
>             "componentType": "FLOAT64"
>           },
>           "triangleCount": {
>             "type": "SCALAR",
>             "componentType": "UINT32"
>           }
>         }
>       },
>       "subtree": {
>         "properties": {
>           "attributionStrings": {
>             "semantic": "ATTRIBUTION_STRINGS",
>             "type": "STRING",
>             "array": true
>           }
>         }
>       }
>     }
>   }
> }
> ```
> 
> _Subtree JSON_
> 
> ```json
> {
>   "buffers": [
>     {
>       "name": "Availability Buffer",
>       "uri": "availability.bin",
>       "byteLength": 48
>     },
>     {
>       "name": "Metadata Buffer",
>       "uri": "metadata.bin",
>       "byteLength": 6512
>     }
>   ],
>   "bufferViews": [
>     { "buffer": 0, "byteOffset": 0, "byteLength": 11 },
>     { "buffer": 0, "byteOffset": 16, "byteLength": 32 },
>     { "buffer": 1, "byteOffset": 0, "byteLength": 2040 },
>     { "buffer": 1, "byteOffset": 2040, "byteLength": 1530 },
>     { "buffer": 1, "byteOffset": 3576, "byteLength": 344 },
>     { "buffer": 1, "byteOffset": 3920, "byteLength": 1024 },
>     { "buffer": 1, "byteOffset": 4944, "byteLength": 240 },
>     { "buffer": 1, "byteOffset": 5184, "byteLength": 122 },
>     { "buffer": 1, "byteOffset": 5312, "byteLength": 480 },
>     { "buffer": 1, "byteOffset": 5792, "byteLength": 480 },
>     { "buffer": 1, "byteOffset": 6272, "byteLength": 240 }
>   ],
>   "propertyTables": [
>     {
>       "class": "tile",
>       "count": 85,
>       "properties": {
>         "horizonOcclusionPoint": {
>           "values": 2
>         },
>         "countries": {
>           "values": 3,
>           "arrayOffsets": 4,
>           "stringOffsets": 5,
>           "arrayOffsetType": "UINT32",
>           "stringOffsetType": "UINT32"
>         }
>       }
>     },
>     {
>       "class": "content",
>       "count": 60,
>       "properties": {
>         "attributionIds": {
>           "values": 6,
>           "arrayOffsets": 7,
>           "arrayOffsetType": "UINT16"
>         },
>         "minimumHeight": {
>           "values": 8
>         },
>         "maximumHeight": {
>           "values": 9
>         },
>         "triangleCount": {
>           "values": 10,
>           "min": 520,
>           "max": 31902
>         }
>       }
>     }
>   ],
>   "tileAvailability": {
>     "constant": 1
>   },
>   "contentAvailability": [{
>     "bitstream": 0,
>     "availableCount": 60
>   }],
>   "childSubtreeAvailability": {
>     "bitstream": 1
>   },
>   "tileMetadata": 0,
>   "contentMetadata": [1],
>   "subtreeMetadata": {
>     "class": "subtree",
>     "properties": {
>       "attributionStrings": [
>         "Source A",
>         "Source B",
>         "Source C",
>         "Source D"
>       ]
>     }
>   }
> }
> ```

### Multiple Contents

When the implicit root tile has multiple contents then `contentAvailability` and `contentMetadata` are provided for each content layer.

> **Example**
>
> JSON description of a subtree with multiple contents. In this example all tiles are available, all building contents are available, and only some tree contents are available.
>
> _Implicit root tile_
> 
> ```json
> {
>   "root": {
>     "boundingVolume": {
>       "region": [-1.318, 0.697, -1.319, 0.698, 0, 20]
>     },
>     "refine": "ADD",
>     "geometricError": 5000,
>     "contents": [
>       {
>         "uri": "buildings/{level}/{x}/{y}.glb",
>       },
>       {
>         "uri": "trees/{level}/{x}/{y}.glb",
>       }
>     ],
>     "implicitTiling": {
>       "subdivisionScheme": "QUADTREE",
>       "availableLevels": 21,
>       "subtreeLevels": 7,
>       "subtrees": {
>         "uri": "subtrees/{level}/{x}/{y}.json"
>       }
>     }
>   }
> }
> ```
> 
> _Subtree JSON_
> 
> ```json
> {
>   "propertyTables": [
>     {
>       "class": "building",
>       "count": 85,
>       "properties": {
>         "height": {
>           "values": 1
>         },
>         "owners": {
>           "values": 2,
>           "arrayOffsets": 3,
>           "stringOffsets": 4
>         }
>       }
>     },
>     {
>       "class": "tree",
>       "count": 52,
>       "properties": {
>         "height": {
>           "values": 5
>         },
>         "species": {
>           "values": 6
>         }
>       }
>     }
>   ],
>   "tileAvailability": {
>     "constant": 1
>   },
>   "contentAvailability": [
>     {
>       "constant": 1
>     },
>     {
>       "bitstream": 0,
>       "availableCount": 52
>     }
>   ],
>   "childSubtreeAvailability": {
>     "constant": 1
>   },
>   "contentMetadata": [0, 1]
> }
> ```
