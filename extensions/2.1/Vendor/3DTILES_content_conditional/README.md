<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# 3DTILES\_content\_conditional

## Contributors

- Marco Hutter, Cesium
- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 specification.

Depends on [3DTILES_tileset](../3DTILES_tileset/README.md).

## Optional vs. Required

This extension is required, meaning it **MUST** be placed in both `extensionsRequired` and `extensionsUsed`.

## Overview

This extension adds support for conditional content in 3D Tiles, by defining the following elements:

- A new content *type* that can be referred to as an external asset.
- An *extension object* in the top-level tileset that describes the structure of the conditional content

In the context of this extension, 'conditional content' is an external asset where the actual content data that is loaded and rendered depends on user-selectable criteria. In the context of this specification, the content data item that should be loaded and rendered is referred to as the '_active_' content data.

> **Implementation Note**
>
> This extension specification itself does not define the behavior of clients regarding this 'active' content data. Clients could choose to pre-load all content data items, to allow them to quickly switch between them when a certain item becomes active. Alternatively, they can choose to download a content item lazily when it becomes active. When they download the content lazily, they can choose to display the previously active content until the newly active content is available and ready to be rendered, or they can choose to display a placeholder or loading indicator instead.

### Conditional Content Type

The new content type for the conditional content is represented as a JSON file. Such a file can be referred to as an external asset. The file contains an array `conditionalContents`, where each item is a [3D Tiles `content`](https://github.com/CesiumGS/3d-tiles/blob/1.1/specification/schema/content.schema.json) with additional properties.

> TODO_GLTF The `content` does not exist in glTF. The original schema JSON for `content` could be replicated here. This has to be aligned with how (content) metadata is supposed to be represented in 3D Tiles 2.0.

The additional properties that are defined for each item are the `keys`. These keys serve as the basis for deciding whether the respective content item should be active. The keys are objects with arbitrary properties that have the type `string`, `number`, or `boolean`. The set of properties is defined by the top-level extension object (as described below).

 An example conditional content data is shown here: It defines two different content items. These contents contain different keys. The keys allow the application to select the contents to be 'active', depending on a time stamp and a revision indicator.

```jsonc
{
  "conditionalContents" : [ {
   "uri" : "content-0-0-2025-09-25-revision0.glb",
   "keys" : {
     "exampleTimeStamp" : "2025-09-25",
     "exampleRevision" : "revision0"
   }
 }, {
   "uri" : "content-0-0-2025-09-26-revision1.glb",
   "keys" : {
     "exampleTimeStamp" : "2025-09-26",
     "exampleRevision" : "revision1"
   }
 } ]
}
```

The `uri` of an item in the `conditionalContents` array is resolved relative to the conditional content JSON file.

The conditional content JSON files should use the `.json` extension and the [application/json](https://www.iana.org/assignments/media-types/application/json) Media Type.

### Top-level Extension Object

The structure of the conditional contents is defined with a top-level extension object. This object is stored as the `3DTILES_content_conditional` object in the `extensions` dictionary of the `3DTILES_tileset` extension. An example of such an object - corresponding to the example content from the previous section - is shown here:


```jsonc
{
  "dimensions" : [ {
    "name" : "exampleTimeStamp",
    "keySet" : [ "2025-09-25", "2025-09-26" ]
  }, {
    "name" : "exampleRevision",
    "keySet" : [ "revision0", "revision1" ]
  } ]
}
```

It defines the `dimensions` that are available for selecting the content that should be active. Each dimension has a `name` that corresponds to the property name in the `keys` of the content object. And it defines a `keySet`, which defines the domain of the respective property, as an array of all values that appear as the values for the respective key in any content.

### Content Structure Constraints

- The items that are contained in the `conditionalContents` may refer to different content types. This includes the common content types that are supported as the [3D Tiles 1.1 tile formats](https://github.com/CesiumGS/3d-tiles/tree/1.1/specification/TileFormats). But the items may _not_ refer to _external tilesets_. And they may _not_ refer to other conditional contents.

> TODO_GLTF This has to be aligned with the concept of "tile formats" for 3D Tiles 2.0


## Schema

- [`3DTILES_content_conditional` extension object schema](./schema/glTF.3DTILES_content_conditional.schema.json)
- [Conditional content JSON schema](./schema/conditionalContent.schema.json)
  - [Conditional content item JSON schema](./schema/conditionalContentItem.schema.json)

