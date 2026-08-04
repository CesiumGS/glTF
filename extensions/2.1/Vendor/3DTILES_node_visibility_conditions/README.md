<!--
SPDX-FileCopyrightText: 2026 Bentley Systems, Incorporated

SPDX-License-Identifier: CC-BY-4.0
-->

# 3DTILES_node_visibility_conditions

## Contributors

- Marco Hutter, Cesium
- Sean Lilley, Cesium

## Status

Draft

## Dependencies

Written against the glTF 2.1 spec.

Depends on [KHR_node_visibility](../../../2.0/Khronos/KHR_node_visibility).

## Optional vs. Required

This extension is optional.

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

This extension does not define the exact runtime behavior. It only offers the mechanism for _defining_ the conditions. The _evaluation_ of the conditions is application-specific. 

A common implementation could be to evaluate the condition using simple equality checks. The application could inspect the top-level extension object, and allow the user to select the values for each condition variable from the given values in the `domain`. The node that carries the extension object would then turn `visible` if an only if each condition value matches the corresponding value that was selected from the top-level object. In this case, the _values_ of the `conditions` dictionary would directly correspond to elements of the respective `domain`.

## JSON Schema

- [glTF.3DTILES_node_visibility_conditions.schema.json](schema/glTF.3DTILES_node_visibility_conditions.schema.json)
- [3DTILES_node_visibility_conditions.schema.json](schema/3DTILES_node_visibility_conditions.schema.json)

