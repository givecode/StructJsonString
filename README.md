# StructJsonString for Unreal Engine 5 (UE5)

StructJsonString is a runtime Unreal Engine plugin that converts Unreal Engine structures to and from JSON strings in Blueprint and C++.

The plugin supports native C++ `USTRUCT` types, Blueprint structures, wildcard structure pins, and `FInstancedStruct`. It is designed for runtime use and does not require an editor-only module or third-party libraries.

## Features

- Convert any structure connected to a wildcard Blueprint pin into a JSON string.
- Convert a JSON string into both a wildcard structure and an `FInstancedStruct`.
- Convert an `FInstancedStruct` directly into a JSON string.
- Choose pretty, indented JSON or compact JSON output.
- Exclude fields recursively by field name.
- Exclude a specific field by a dot-separated JSON path.
- Preserve stable authored field names for native C++ and Blueprint structures.
- Report conversion failures through return values, error strings, and the Unreal Engine Output Log.
- Use the conversion functions in packaged runtime builds.

## Compatibility

- Unreal Engine 5.6
- Unreal Engine 5.7
- Unreal Engine 5.8
- Module type: Runtime

Use the plugin package built for your exact Unreal Engine version. Supported target platforms are listed on the corresponding Fab product page and package.

## Installation

1. Close Unreal Engine.
2. Copy the `StructJsonString` folder into your project's `Plugins` directory:

   ```text
   YourProject/
   └── Plugins/
       └── StructJsonString/
           ├── Source/
           └── StructJsonString.uplugin
   ```

3. Open the project.
4. Open **Edit > Plugins** and search for `StructJsonString`.
5. Enable the plugin and restart Unreal Engine if prompted.
6. For a C++ project, regenerate project files and compile the project.

## Blueprint Nodes

All Blueprint nodes are available in the `StructJsonString` category.

### Convert Struct to JSON String

Converts the structure connected to the wildcard `In Struct` pin into JSON.

Outputs:

- `Return Value`: `true` when conversion succeeds.
- `Out Json`: the generated JSON string.
- `Out Error`: an error message when conversion fails.
- `Pretty Json`: enables indented output when `true`; produces compact output when `false`.

### Convert Struct to JSON String (Ignore by Name)

Works like the standard conversion node, but removes every field whose name appears in `Ignore Fields`. Matching is case-insensitive and recursive at all nesting levels.

Example: ignoring `InternalNote` removes every field named `InternalNote`, including fields inside nested structures and arrays of structures.

### Convert Struct to JSON String (Ignore by Path)

Removes only fields selected by dot-separated JSON paths supplied through `Ignore Paths`.

Example paths:

```text
PrimaryItem.InternalNote
Profile.Contact.Email
```

Use this node when a field name appears in several places but only one specific occurrence should be omitted.

### Convert Instanced Struct to JSON String

Converts a valid `FInstancedStruct` value into JSON. The node returns `false` if the input does not contain a valid structure.

### Convert JSON String to Instanced Struct and Any Struct

Deserializes a JSON object into two outputs:

- `Out Instanced`: an `FInstancedStruct` containing the selected output structure type and data.
- `Out Struct`: a wildcard structure output whose type is determined by the connected pin.
- `Out Error`: details when parsing or conversion fails.

Connect a concrete structure variable to `Out Struct` so the node can determine the target type.

### Convert JSON String to Instanced Struct and Any Struct V2

Provides the same output model as the node above and is available as the V2 Blueprint entry point. For new Blueprint graphs, prefer V2 unless an existing project already uses the earlier node.

## Blueprint Workflow

### Structure to JSON

1. Create or obtain a structure value.
2. Add **Convert Struct to JSON String**.
3. Connect the structure to `In Struct`.
4. Select pretty or compact output with `Pretty Json`.
5. Check `Return Value` before using `Out Json`.
6. If conversion fails, inspect `Out Error` and the Output Log.

### JSON to Structure

1. Prepare a valid JSON object string.
2. Add **Convert JSON String to Instanced Struct and Any Struct V2**.
3. Connect a variable of the required structure type to `Out Struct`.
4. Check `Return Value` before reading either output.
5. Use `Out Error` to diagnose invalid JSON or incompatible values.

## JSON Example

Given a structure containing an ID, title, enabled flag, score, nested item, array, set, and map, pretty output can look like this:

```json
{
  "Id": 7,
  "Title": "StructJsonString sample",
  "bEnabled": true,
  "Score": 91.5,
  "PrimaryItem": {
    "Name": "Sword",
    "Damage": 25,
    "InternalNote": "Not exported when ignored"
  },
  "Items": [],
  "Tags": ["Demo", "Runtime"],
  "UniqueCodes": ["A01", "B02"],
  "Attributes": {
    "Health": 100,
    "Mana": 50
  }
}
```

JSON keys follow the stable authored names resolved by the plugin. Input matching also accepts compatible Unreal/FJsonObjectConverter-style name variants and performs case-insensitive fallback matching.

## Supported Data Shapes

The implementation uses Unreal Engine reflection and `FJsonObjectConverter`. It supports reflected properties handled by Unreal's JSON conversion system, including the following data shapes demonstrated by the plugin:

- Boolean, integer, and floating-point values
- Strings and other compatible reflected scalar properties
- Native C++ and Blueprint structures
- Nested structures
- Arrays
- Sets
- Maps with JSON-compatible keys
- `FInstancedStruct`

Values must be compatible with the connected target structure and its reflected property types.

## C++ Usage

Add the plugin module to your project's module dependencies:

```csharp
PublicDependencyModuleNames.AddRange(
    new string[]
    {
        "Core",
        "CoreUObject",
        "Engine",
        "StructJsonString"
    }
);
```

Include the public header:

```cpp
#include "StructStringLibrary.h"
```

### Convert a native USTRUCT to JSON

```cpp
FMyPayload Payload;
Payload.Id = 7;

FString Json;
const bool bSuccess = UStructStringLibrary::StructToJsonString(
    Payload,
    Json,
    true
);
```

`FMyPayload` must be a reflected structure that provides `StaticStruct()`.

### Convert JSON to a native USTRUCT

```cpp
FMyPayload Payload;
FString Error;

const bool bSuccess = UStructStringLibrary::JsonStringToStruct(
    Json,
    Payload,
    &Error
);
```

### Convert FInstancedStruct to JSON

```cpp
FString Json;
const bool bSuccess = UStructStringLibrary::InstancedStructToJsonString(
    InstancedPayload,
    Json,
    true
);
```

## Error Handling

Always check the Boolean return value. When a node or C++ function exposes an error string, inspect it before continuing.

Common causes of failure include:

- Empty or malformed JSON input.
- A JSON root that is not an object.
- No concrete structure type connected to a wildcard output pin.
- An invalid or empty `FInstancedStruct`.
- JSON values that cannot be converted to the target property types.
- Duplicate JSON keys produced by conflicting property names.

Additional diagnostics are written to the `LogStructJsonString` category in the Unreal Engine Output Log.

## Important Behavior and Limitations

- JSON deserialization requires a JSON object at the root.
- Missing JSON fields retain the structure's current/default values.
- A JSON `null` value does not overwrite the current property value.
- Ignore-by-name matching is case-insensitive and affects matching fields recursively.
- Ignore-by-path uses dot-separated field paths.
- Map data is represented as a JSON object; map keys must therefore be representable as JSON field names.
- The wildcard Blueprint nodes require a concrete structure connection so Unreal can resolve the type.
- Engine compatibility depends on using the package built for the matching Unreal Engine version.

## Runtime and Packaging

StructJsonString is a runtime module. After enabling it, its Blueprint and C++ APIs can be called in packaged applications. Test the plugin in the Development and Shipping configurations used by your project before release.

## Support

For bug reports or feature requests, open an issue in this documentation repository or use the support link provided on the Fab product page.

When reporting a problem, include:

- Plugin version
- Unreal Engine version
- Target platform
- Blueprint or C++ usage
- Input JSON or a minimal structure definition
- Relevant `LogStructJsonString` output

## License

Use of the plugin is governed by the license supplied with the Fab product.
