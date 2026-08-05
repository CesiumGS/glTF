<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# 3DTILES_node_visibility_conditions

## Contributors

- Marco Hutter, Cesium
- Sean Lilley, Cesium
- Björn Blissing, Vantor

## Status

Draft

## Dependencies

Written against the glTF 2.1 specification.

Depends on [KHR_node_visibility](../../../2.0/Khronos/KHR_node_visibility).

## Optional vs. Required

This extension is optional. 

Implementations that do not support this extension (but only the `KHR_node_visibility` extension) will simply use the value of the `visible` flag that is defined in the `KHR_node_visibility` extension.

## Overview

This extension adds support for conditions for the `KHR_node_visibility` extension. These conditions allow the client to derive a value for the `visible` property at runtime.

## Extending the Visibility Extension

The `KHR_node_visibility` extension is associated with a node, and defines the `visible` flag that indicates whether a node and all its descendants. The `3DTILES_node_visibility_conditions` extension extends this extension, and consists of two parts:

- The main extension object is contained in the `KHR_node_visibility` extension object, and defines the conditions for the `visible` flag to be `true`
- A top-level extension object defines the structure of the conditions that may appear in the extension object.

The following example is a glTF asset that uses the extension to define the condition for a node with an external asset to be visible:

```jsonc
{
  "asset": {
    "version": 2.1,
  },
  "extensions": {
    "3DTILES_node_visibility_conditions": {
      "dimensions": [
        {
          "name": "exampleTimeStamp",
          "domain": ["2025-09-25", "2025-09-26"],
        },
        {
          "name": "exampleRevision",
          "domain": ["revision0", "revision1"],
        },
      ],
    },
  },
  "files": [
    {
      "mimeType": "model/gltf-binary",
      "uri": "example.glb",
    },
  ],
  "externalAssets": [
    {
      "file": 0,
    },
  ],
  "nodes": [
    {
      "externalAsset": 0,
      "extensions": {
        "KHR_node_visibility": {
          "visible": false,
          "extensions": {
            "3DTILES_node_visibility_conditions": {
              "conditions": {
                "exampleTimeStamp": "2025-09-25",
                "exampleRevision": "revision0",
              },
            },
          },
        },
      },
    },
  ],
  "scene": 0,
  "scenes": [{ "nodes": [0] }],
}
```

The top-level extension object defines the `dimensions` for the conditions. These are given as a dictionary that maps the name of the condition to the `domain`, which is an array containing all possible values for the condition variables.

The `3DTILES_node_visibility_conditions` object that is associated with the `KHR_node_visibility` object of the only node defines the set of conditions based on which the `visible` flag of the `KHR_node_visibility` extension object should be `true` or `false`.

## Runtime Behavior

The process for determining the value of the `visible` flag is as follows:

- A client defines a _visibility criteria_ object. If no such object is provided, then the `visible` flag keeps the value that it has in the file (defaulting to `false` if it was not defined)
- The visibility criteria object MAY have arbitrary properties.
- The client examines all `3DTILES_node_visibility_conditions` objects, and determines if they are matching the visibility criteria criteria object as follows:
  - The client examines every property in the visibility criteria object.
  - If the name of the property does not appear in the dimensions defined in the `dimensions` property of the top-level extension object, then the property is ignored.
  - Otherwise, the client tests whether the value of the property in the visibility criteria object matches the corresponding value that is given in the `conditions`. The exact matching process is implementation-defined. Implementations MAY interpret the values as opaque values and perform an exact equality comparison, or they MAY interpret them using application-defined semantics (for example, interpreting a string as a date range and considering a criterion date to satisfy the condition when it falls within that range). This extension does not define how values are interpreted.
  - If all property values from the conditions match the values from the visibility criteria object, then the `visible` flag of the `KHR_node_visibility` extension object is set to `true`. Otherwise, it is set to `false`. 


> TODO_GLTF There currently is no easy mechanism for "wildcards" that allow activating ALL contents (regardless of their keys). This would be required in order to let this extension easily mimic the 'multiple contents' extension.


## JSON Schema

- [glTF.3DTILES_node_visibility_conditions.schema.json](schema/glTF.3DTILES_node_visibility_conditions.schema.json)
- [3DTILES_node_visibility_conditions.schema.json](schema/3DTILES_node_visibility_conditions.schema.json)

