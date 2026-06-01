# 3DTILES\_subtree

## Contributors

- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

A subtree file is a glTF file that contains availability and metadata information for a single subtree in [3DTILES_implicit_tiling](../3DTILES_implicit_tiling/README.md).

## File Extensions

Assets that use the `3DTILES_subtree` extension **SHOULD** use the `.subtree.gltf` or `.subtree.glb` file extensions. Though not required, this convention helps differentiate subtree files from tileset files and content files.

## Constraints

The following constraints apply when using the `3DTILES_subtree` extension:

- Top level properties other than `asset`, `accessors`, `bufferViews`, `buffers`, and `files` **MUST NOT** be defined.

## Schema

The JSON description of a subtree where each tile is available, but not all tiles have content, and not all child subtrees are available:

```json
{
  "buffers": [
    {
      "byteLength": 48
    }
  ],
  "bufferViews": [
    {
      "buffer": 0,
      "byteOffset": 0,
      "byteLength": 11
    },
    {
      "buffer": 1,
      "byteOffset": 0,
      "byteLength": 32
    }
  ],
  "accessors": [
    {
      "type": "SCALAR",
      "componentType": 5121,
      "count": 
    },
    {
      "type": "SCALAR",
      "componentType": 5121

    }
  ],
  "tileAvailability": {
    "constant": 1,
  },
  "contentAvailability": [{
    "bitstream": 0,
    "availableCount": 60
  }],
  "childSubtreeAvailability": {
    "bitstream": 1
  }
}
```

{
  "accessors": [
    {
    
    },
    {

    },
    {

    }
  ],
  "extensions": {
    "3DTILES_subtree": {
      "tileAvailability": {
        0,
      "contentAvailability": 1,
      "childSubtreeAvailability": 2 ,
    }
  }
}
```

```json
{
  "extensions": {
    "3DTILES_subtree": {
      "tileAvailability": 0,
      "contentAvailability": 0,
      "childSubtreeAvailability": 0,
      "tileMetadata": 0,
      "contentMetadata: 1
    },
    "EXT_structural_metadata": {
      "propertyTables": [
    {
      "class": "building",
      "count": 85,
      "properties": {
        "height": {
          "values": 1
        },
        "owners": {
          "values": 2,
          "arrayOffsets": 3,
          "stringOffsets": 4
        }
      }
    },
    {
      "class": "tree",
      "count": 52,
      "properties": {
        "height": {
          "values": 5
        },
        "species": {
          "values": 6
        }
      }
    }

    }
  },
  ],
  "tileAvailability": {
    "constant": 1
  },
  "contentAvailability": [
    {
      "constant": 1
    },
    {
      "bitstream": 0,
      "availableCount": 52
    }
  ],
  "childSubtreeAvailability": {
    "constant": 1
  },
  "contentMetadata": [0, 1]
}
```

Tile availability (`tileAvailability`) and child subtree availability (`childSubtreeAvailability`) shall always be provided for a subtree.

Content availability (`contentAvailability`) 

 If the implicit root tile does not have a content or contents property, and therefore does not define a content template URI, then this means that none of the implicit tiles has any content, and the contentAvailability in the subtrees shall be omitted.

## TODO

- Evaluate size difference between gzipped byte stream vs. gzipped bitstream for availability.