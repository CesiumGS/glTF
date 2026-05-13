# 3DTILES\_tileset

## Contributors

- Sean Lilley, Cesium
- Adam Morris, Cesium
- Tamrat Belayneh, ESRI

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on:

- `EXT_structural_metadata` for assigning properties to entities
- `EXT_geometric_error` for geometric error based node refinement
- `EXT_bounding_volume_region` for region bounding volumes
- `EXT_geospatial_crs` for geospatial coordinate reference system definitions
- `3DTILES_subtree` for concise representation of quadtrees and octrees.

## Overview

This extension specifies a well defined subset of glTF 2.1 for representing a tileset in [3D Tiles](https://github.com/CesiumGS/3d-tiles/tree/main/specification).

## 3D Tiles

In 3D Tiles, a _tileset_ is a set of _tiles_ organized in a spatial data structure, the _tree_. Each tile may reference renderable _content_.

The content references a set of _features_, such as 3D models representing buildings or trees, or points in a point cloud. Each feature has position and appearance properties and additional application-specific properties. A client may choose to select features at runtime and retrieve their properties for visualization or analysis.

Tiles are organized in a tree which incorporates the concept of Hierarchical Level of Detail (HLOD) for optimal rendering of spatial data. Each tile has a _bounding volume_, an object defining a spatial extent completely enclosing its content. The tree has [spatial coherence](https://github.com/CesiumGS/3d-tiles/tree/main/specification#core-bounding-volume-spatial-coherence); the content for child tiles are completely inside the parent’s bounding volume.

A tileset may use a 2D spatial tiling scheme similar to raster and vector tiling schemes (like a Web Map Tile Service (WMTS) or XYZ scheme) that serve predefined tiles at several levels of detail (or zoom levels). However since the content of a tileset is often non-uniform or may not easily be organized in only two dimensions, the tree can be any spatial data structure with spatial coherence, including k-d trees, quadtrees, octrees, and grids. [Implicit tiling](#implicit-tiling) defines a concise representation of quadtrees and octrees.

Application-specific _metadata_ may be provided at multiple granularities within a tileset. Properties may be associated with high-level entities like tilesets, tiles, contents, or features, or with individual vertices and texels. Metadata conforms to a well-defined type system described by the [3D Metadata Specification](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Metadata/README.adoc#metadata-3d-metadata-specification), which may be extended with application- or domain-specific semantics.

Optionally a [3D Tiles Style](https://github.com/CesiumGS/3d-tiles/blob/main/specification/Styling/README.adoc#styling-styling), or _style_, may be applied to a tileset. A style defines expressions to be evaluated which modify how each feature is displayed.


## glTF constraints

This extension imposes certain constraints on glTF assets as described below.

### Scenes

- The document `"scenes"` array MUST have exactly one scene.
- The document `"scene"` index MUST be set to 0, the index of the only scene in the `"scenes"` array.
- The scene MUST have exactly one node, the single root node.

### Nodes

- A node MUST NOT have a `mesh`
- A node MUST have a `boundingVolume`
- A node MUST use the `EXT_geometric_error` extension





## Concepts

### Overview

This section describes the concepts of 3D Tiles and how they are represented in glTF.

### Tileset

A tileset is a set of tiles organized in a spatial data structure, the tree.






A glTF with the `3DTILES_tileset` extension is a tileset and its node hierarchy is the tree.

3D Tiles uses one main tileset file as the entry point to define a tileset. To create a tree of trees, a tileset may also reference external tilesets.

> [!NOTE]  
> Tileset files are not required to follow a specific naming convention, but generally tileset files (those with the `3DTILES_tileset` extension) use the naming scheme `*.tileset.gltf` to distinguish them from content resources `*.gltf` or `*.glb`

Here is a subset of the tileset used for Canary Wharf:













### Tile

Tiles consist of metadata used to determine if a tile is rendered, a reference to the renderable content, and an array of any children tiles.

A tile is represented by a glTF node.

```json
{
  "reference": 0,
  "boundingVolume": {
    "shape": 0
  },
  "extensions": {
    "EXT_geometric_error": {
      "geometricError": 70.0,
    }
  },
  "children": [1, 2, 3, 4],
}
```

#### Tile Content

A tile can be associated with renderable content. 



Tile content is represented by an external reference `reference`

The content.uri of each content object refers to the tile’s content in one of the tile formats that are defined in the Tile format specifications), or another tileset JSON to create a tileset of tilesets (see External tilesets).

The content.group property assigns the content to a group. Contents of different tiles or the contents of a single tile can be assigned to groups in order to categorize the content. Additionally, each group can be associated with Metadata.

Each content can be associated with a bounding volume. While tile.boundingVolume is a bounding volume that encloses all contents of the tile, each individual content.boundingVolume is a tightly fit bounding volume enclosing just the respective content. More details about the role of tile- and content bounding volumes are given in the bounding volume section.



#### Geometric error

#### Refinement

##### Replacement

##### Additive

#### Feature

### Bounding volumes

### Metadata

### Style

## Bounding volumes

The <<core-region,region>> bounding volume specifies bounds using a geographic coordinate system (latitude, longitude, height). Specifically, https://epsg.org/crs_4979/WGS-84.html[EPSG 4979], but with the latitude and longitude given in _radians_ instead of _degrees_. The reference ellipsoid is assumed to be the same as the reference ellipsoid of the tileset.

## Coordinate reference system (CRS)

3D Tiles uses a right-handed Cartesian coordinate system; that is, the cross product of x and y yields z. 3D Tiles defines the z axis as up for local Cartesian coordinate systems. A tileset’s global coordinate system will often be in a WGS 84 Earth-centered, Earth-fixed (ECEF) reference frame (EPSG 4978), but it doesn’t have to be, e.g., a power plant may be defined fully in its local coordinate system for use with a modeling tool without a geospatial context.



A tileset's global coordinate system will often be in a https://epsg.org/ellipsoid_7030/WGS-84.html[WGS 84] Earth-centered, Earth-fixed (ECEF) reference frame (https://epsg.org/crs_4978/WGS-84.html[EPSG 4978]), but it doesn't have to be, e.g., a power plant may be defined fully in its local coordinate system for use with a modeling tool without a geospatial context.

The CRS of a tileset may be defined explicitly, as part of the <<core-metadata,tileset metadata>>. The metadata for the tileset can contain a property that has the xref:{url-specification-metadata-semantics}README.adoc#metadata-semantics-tileset-semantics[`TILESET_CRS_GEOCENTRIC` semantic], which is a string that represents the EPSG Geodetic Parameter Dataset identifier.

An additional <<core-tile-transforms,tile transform>> may be applied to transform a tile's local coordinate system to the parent tile's coordinate system.


## Example

A tileset with a tileset node, a root node


```json
{
  "extensionsUsed": ["3DTILES_tileset", "EXT_geometric_error"],
  "extensionsRequired": ["3DTILES_tileset", "EXT_geometric_error"],
  "asset": {
    "version": "2.1"
  },
  "scenes": [
    {
      "nodes": [0]
    }
  ],
  "scene": 0,
  "references": [
    {
      "uri": "parent.glb"
    },
    {
      "uri": "ll.glb"
    },
    {
      "uri": "lr.glb"
    },
    {
      "uri": "ur.glb"
    },
    {
      "uri": "ul.glb"
    }
  ],
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
      "name": "tileset",
      "children": [1],
      "extensions": {
        "EXT_geometric_error": {
          "geometricError": 240.0,
          "refine": "ADD",
        }
      }
    },
    {
      "name": "root",
      "children": [2, 3, 4, 5],
      "reference": 0,
      "boundingVolume": {
        "shape": 0,
        "scale": [10.0, 2.0, 10.0]
      },
      "extensions": {
        "EXT_geometric_error": {
          "geometricError": 70.0,
        }
      }
    },
    {
      "name": "ll",
      "reference": 1,
      "boundingVolume": {
        "shape": 0,
        "scale": [5.0, 1.0, 5.0],
        "translation": [-5.0, 0.0, -5.0]
      },
      "extensions": {
        "EXT_geometric_error": {
          "geometricError": 0.0
        }
      }
    },
    {
      "name": "lr",
      "reference": 2,
      "boundingVolume": {
        "shape": 0,
        "scale": [5.0, 1.0, 5.0],
        "translation": [5.0, 0.0, -5.0]
      },
      "extensions": {
        "EXT_geometric_error": {
          "geometricError": 0.0
        }
      }
    },
    {
      "name": "ur",
      "reference": 3,
      "extensions": {
        "EXT_geometric_error": {
          "geometricError": 0.0
        }
      }
    },
    {
      "name": "ul",
      "reference": 4,
      "extensions": {
        "EXT_geometric_error": {
          "geometricError": 0.0
        }
      }
    }
  ]
}
```


## Recommended File

By convention, glTF*.tileset.gltf


## TODO

- "Metadata" vs. "properties"
- Where to put recommended file extension `*.tileset.gltf`, in this extension or in the 3D Tiles 2.0 spec
- Forward axis, how does this affect bv's and 
- Implicit tiling derived transforms