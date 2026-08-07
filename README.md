# StructJsonString for Unreal Engine 5 (UE5)

StructJsonString is a runtime Unreal Engine code plugin that converts supported reflected `USTRUCT` values to and from JSON strings in Blueprint and C++.

The plugin supports native C++ structures, Blueprint user-defined structures, wildcard structure pins, nested containers, and `FInstancedStruct`. It uses Unreal Engine reflection and Unreal Engine's built-in JSON modules. No third-party library, service, account, or network connection is required.

## Features

- Convert a Blueprint or native C++ structure to a JSON string.
- Convert a JSON object into both a typed wildcard structure and an `FInstancedStruct`.
- Convert an `FInstancedStruct` directly to JSON.
- Generate indented (pretty) JSON or compact JSON.
- Omit matching JSON fields recursively by field name.
- Omit selected JSON fields by an exact dot-separated path.
- Preserve authored C++ and Blueprint field names without exposing compiled Blueprint GUID suffixes.
- Handle nested structures, arrays, sets, and maps.
- Report conversion failures through Boolean results, error strings, and `LogStructJsonString`.
- Run in the editor, PIE, standalone games, and packaged runtime applications.

## Compatibility and Requirements

| Item | Supported value |
| --- | --- |
| Plugin version | `1.4.4` |
| Unreal Engine | `5.6`, `5.7`, and `5.8` |
| Target platform | Windows 64-bit (`Win64`) |
| Module type | Runtime |
| Blueprint support | Yes |
| C++ support | Yes |
| Third-party software | None |

Use the plugin package built for the exact Unreal Engine version used by your project. The supplied `.uplugin` descriptor identifies the engine version for its package.

## Download and Installation

### Install from Fab

1. Add the product to your Fab library.
2. Install the plugin for the required Unreal Engine version through the Epic Games Launcher.
3. Open the target project.
4. In Unreal Editor, open **Edit > Plugins**.
5. Search for **StructJsonString**, enable it, and restart Unreal Editor if prompted.

### Install from the supplied ZIP package

1. Download the ZIP file from the product's Project File Link.
2. Close Unreal Editor and your IDE.
3. Extract the single `StructJsonString` plugin folder into the project's `Plugins` directory.

The final path must be:

```text
YourProject/
└── Plugins/
    └── StructJsonString/
        ├── Config/
        ├── Content/
        ├── Resources/
        ├── Source/
        └── StructJsonString.uplugin
```

Do not leave extra archive folders between `Plugins` and `StructJsonString`.

4. Open the project.
5. Open **Edit > Plugins**, search for **StructJsonString**, and enable it.
6. Restart Unreal Editor when prompted.
7. For a C++ project, regenerate project files if necessary and build the project once.

### Verify the installation

Open any Blueprint Event Graph, right-click, and search for:

```text
Convert Struct to JSON String
```

The node should appear in the **StructJsonString** category. If it does not appear, confirm that the plugin is enabled and that the package matches the project's Unreal Engine version.

## 5-Minute Blueprint Quick Test

This test verifies the complete Blueprint workflow: create a structure, serialize it to JSON, and deserialize the JSON back into the same structure type.

### Step 1: Create a test structure

1. In the Content Browser, select **Add > Blueprints > Structure**.
2. Name the structure `ST_StructJsonTest`.
3. Add and name these fields exactly as shown:

| Field name | Blueprint type | Test value |
| --- | --- | --- |
| `Id` | Integer | `7` |
| `Title` | String | `StructJsonString Sample` |
| `Enabled` | Boolean | `true` |

4. Save the structure.

### Step 2: Convert the structure to JSON

1. Create an Actor Blueprint named `BP_StructJsonStringTest`.
2. Open its Event Graph and add **Event BeginPlay**.
3. Add **Make ST_StructJsonTest** and enter the three test values above.
4. Add **Convert Struct to JSON String**.
5. Connect the output of **Make ST_StructJsonTest** to `In Struct`.
6. Leave `Pretty Json` enabled.
7. Connect the execution flow from **Event BeginPlay** to the conversion node.
8. Check `Return Value` with a Branch.
9. On success, send `Out Json` to **Print String**.
10. On failure, send `Out Error` to **Print String**.
11. Compile the Blueprint, place it in a level, and press Play.

The printed JSON must contain the following data. Whitespace and field order are not significant.

```json
{
  "Id": 7,
  "Title": "StructJsonString Sample",
  "Enabled": true
}
```

### Step 3: Convert the JSON back to the structure

1. After the successful serialization call, add **Convert JSON String to Instanced Struct and Any Struct V2**.
2. Connect the first node's `Out Json` to `In Json String` on the V2 node.
3. Add **Break ST_StructJsonTest**.
4. Connect the V2 node's wildcard `Out Struct` pin to the structure input of **Break ST_StructJsonTest**. This connection resolves the wildcard type.
5. Connect the execution flow and check the V2 node's `Return Value` with a Branch.
6. On success, print the recovered `Id`, `Title`, and `Enabled` values.
7. On failure, print `Out Error`.

The round trip succeeds when the recovered values are `7`, `StructJsonString Sample`, and `true`. `Out Instanced` also contains a valid `FInstancedStruct` whose type is `ST_StructJsonTest`.

## Blueprint Node Reference

All Blueprint nodes are in the **StructJsonString** category.

### Convert Struct to JSON String

Converts the reflected structure connected to the wildcard input into JSON.

| Direction | Parameter | Type | Default | Description |
| --- | --- | --- | --- | --- |
| Input | `In Struct` | Wildcard structure | Required | Structure value to serialize. Connect a concrete `USTRUCT` value. |
| Input | `Pretty Json` | Boolean | `true` | Produces indented JSON when enabled and compact JSON when disabled. |
| Output | `Out Json` | String | — | Generated JSON text. Reset to empty before conversion. |
| Output | `Out Error` | String | — | Detailed failure reason. Empty after a successful call. |
| Output | `Return Value` | Boolean | — | `true` when conversion succeeds. |

### Convert Struct to JSON String (Ignore by Name)

Serializes a wildcard structure and removes every JSON field whose name matches an entry in `Ignore Fields`.

| Direction | Parameter | Type | Default | Description |
| --- | --- | --- | --- | --- |
| Input | `In Struct` | Wildcard structure | Required | Structure value to serialize. |
| Input | `Ignore Fields` | Array of Name | Empty | Field names to remove. Matching is case-insensitive and recursive. |
| Input | `Pretty Json` | Boolean | `true` | Selects indented or compact output. |
| Output | `Out Json` | String | — | Generated JSON text after fields are removed. |
| Output | `Out Error` | String | — | Detailed failure reason. |
| Output | `Return Value` | Boolean | — | `true` when conversion succeeds. |

Example: ignoring `InternalNote` removes that name at the root and from nested objects, objects inside arrays, and matching keys inside JSON objects used to represent maps.

### Convert Struct to JSON String (Ignore by Path)

Serializes a wildcard structure and removes fields selected by exact dot-separated JSON paths.

| Direction | Parameter | Type | Default | Description |
| --- | --- | --- | --- | --- |
| Input | `In Struct` | Wildcard structure | Required | Structure value to serialize. |
| Input | `Ignore Paths` | Array of Name | Empty | Exact dot-separated exported JSON paths to remove. |
| Input | `Pretty Json` | Boolean | `true` | Selects indented or compact output. |
| Output | `Out Json` | String | — | Generated JSON text after selected fields are removed. |
| Output | `Out Error` | String | — | Detailed failure reason. |
| Output | `Return Value` | Boolean | — | `true` when conversion succeeds. |

Path examples:

```text
InternalNote
PrimaryItem.InternalNote
Items.InternalNote
```

- `InternalNote` removes only the root field.
- `PrimaryItem.InternalNote` removes only that nested field.
- `Items.InternalNote` removes the field from every object in the `Items` array.
- Path matching is case-sensitive.
- Each segment must use the exact exported JSON key.
- Array-index syntax such as `Items[0].InternalNote` is not supported.
- A field name or map key containing a period (`.`) makes a path ambiguous.

### Convert Instanced Struct to JSON String

Converts a valid `FInstancedStruct` value into JSON.

| Direction | Parameter | Type | Default | Description |
| --- | --- | --- | --- | --- |
| Input | `In Struct` | `FInstancedStruct` | Required | A valid initialized instanced structure. |
| Input | `Pretty Json` | Boolean | `true` | Selects indented or compact output. |
| Output | `Out Json String` | String | — | Generated JSON text. |
| Output | `Return Value` | Boolean | — | `false` when the input is empty/invalid or conversion fails. |

This node does not expose `Out Error`. When it fails, inspect the Unreal Engine Output Log under `LogStructJsonString`.

### Convert JSON String to Instanced Struct and Any Struct

Deserializes a JSON object into a typed wildcard structure and an `FInstancedStruct`.

| Direction | Parameter | Type | Required | Description |
| --- | --- | --- | --- | --- |
| Input | `In Json String` | String | Yes | JSON text whose root value is an object. |
| Output | `Out Instanced` | `FInstancedStruct` | — | Initialized with the same type and data as `Out Struct`. |
| Output | `Out Error` | String | — | Detailed failure reason. |
| Output | `Out Struct` | Wildcard structure | Yes | Connect to a typed Set/Break Structure node or a typed structure variable. |
| Output | `Return Value` | Boolean | — | `true` only when deserialization and both outputs succeed. |

The concrete type connected to `Out Struct` defines the target structure type. The plugin does not infer the target type from the JSON text.

### Convert JSON String to Instanced Struct and Any Struct V2

The V2 node has the same parameters and, in plugin version `1.4.4`, uses the same internal conversion implementation as the original node. The V2 node is used in the quick-start example; the original node remains available for existing Blueprint graphs.

## Included Sample Structures

The source includes two Blueprint-accessible sample types:

- `FStructJsonStringSampleItem`
  - `Name` (`FString`)
  - `Damage` (`int32`)
  - `InternalNote` (`FString`)
- `FStructJsonStringSamplePayload`
  - `Id` (`int32`)
  - `Title` (`FString`)
  - `bEnabled` (`bool`)
  - `Score` (`double`)
  - `PrimaryItem` (nested sample structure)
  - `Items` (array of sample structures)
  - `Tags` (array of strings)
  - `UniqueCodes` (set of strings)
  - `Attributes` (string-to-integer map)
  - `InternalNote` (`FString`)

These sample types can be used to test nested structures, containers, and both ignore modes without adding C++ code to the consuming project.

Example payload:

```json
{
  "Id": 7,
  "Title": "StructJsonString Sample",
  "bEnabled": true,
  "Score": 91.5,
  "PrimaryItem": {
    "Name": "Sword",
    "Damage": 25,
    "InternalNote": "Nested note"
  },
  "Items": [],
  "Tags": ["Demo", "Runtime"],
  "UniqueCodes": ["A01", "B02"],
  "Attributes": {
    "Health": 100,
    "Mana": 50
  },
  "InternalNote": "Root note"
}
```

With `Ignore Fields = [InternalNote]`, both `InternalNote` fields are omitted. With `Ignore Paths = [PrimaryItem.InternalNote]`, only the nested field is omitted.

## C++ Integration

### Module dependency

Add `StructJsonString` to the public or private dependencies of the consuming module, according to where your code uses the API:

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

The plugin itself uses Unreal Engine's built-in `Json` and `JsonUtilities` modules. No third-party dependency needs to be downloaded or integrated.

### Define a native structure

```cpp
// JsonExampleTypes.h
#pragma once

#include "CoreMinimal.h"
#include "JsonExampleTypes.generated.h"

USTRUCT(BlueprintType)
struct FJsonExamplePayload
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 Id = 0;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FString Title;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    bool bEnabled = false;
};
```

The C++ template functions require a reflected structure type that provides `StaticStruct()`.

### Convert a native structure to and from JSON

```cpp
#include "JsonExampleTypes.h"
#include "StructStringLibrary.h"

FJsonExamplePayload Source;
Source.Id = 7;
Source.Title = TEXT("StructJsonString Sample");
Source.bEnabled = true;

FString Json;
const bool bSerialized = UStructStringLibrary::StructToJsonString(
    Source,
    Json,
    true // Pretty JSON
);

FJsonExamplePayload Result;
FString Error;
const bool bDeserialized =
    bSerialized &&
    UStructStringLibrary::JsonStringToStruct(
        Json,
        Result,
        &Error
    );

if (!bDeserialized)
{
    UE_LOG(
        LogTemp,
        Error,
        TEXT("StructJsonString conversion failed: %s"),
        *Error
    );
}
```

`StructToJsonString` reports failure through its Boolean return value and the Output Log. `JsonStringToStruct` can also return a detailed message through the optional `OutError` pointer.

### Convert an FInstancedStruct to JSON

```cpp
#include "JsonExampleTypes.h"
#include "StructStringLibrary.h"

FInstancedStruct InstancedPayload;
InstancedPayload.InitializeAs<FJsonExamplePayload>();

FJsonExamplePayload& Payload =
    InstancedPayload.GetMutable<FJsonExamplePayload>();
Payload.Id = 7;
Payload.Title = TEXT("StructJsonString Sample");
Payload.bEnabled = true;

FString Json;
const bool bSuccess =
    UStructStringLibrary::InstancedStructToJsonString(
        InstancedPayload,
        Json,
        true
    );
```

### Blueprint CustomThunk functions and direct C++ calls

The wildcard functions declared with `CustomThunk` exist to receive dynamically typed Blueprint structure pins. Their ordinary native function bodies are placeholders and return `false` when called directly from C++.

Use these native C++ entry points instead:

- `UStructStringLibrary::StructToJsonString<TStruct>()`
- `UStructStringLibrary::JsonStringToStruct<TStruct>()`
- `UStructStringLibrary::InstancedStructToJsonString()`

Use the wildcard, ignore-by-name, ignore-by-path, and dual-output JSON nodes from Blueprint graphs.

## Supported Data and JSON Representation

The plugin supports reflected values handled by Unreal Engine's `FJsonObjectConverter` and explicitly processes nested containers.

| Unreal data shape | JSON representation |
| --- | --- |
| Boolean | Boolean |
| Integer and floating-point values | Number |
| String and compatible scalar properties | String or Unreal converter representation |
| Enum | Unreal converter representation |
| Native or Blueprint `USTRUCT` | Object |
| Nested structure | Nested object |
| `TArray` | Array |
| `TSet` | Array |
| `TMap` | Object whose field names represent map keys |
| `FInstancedStruct` | Object representing its contained structure |

Map keys are imported directly for `FString`, `FName`, and `FText`. Other map-key property types must accept Unreal text import from the JSON field name. Properties that Unreal Engine's JSON conversion system cannot represent are not supported.

Properties marked `Transient` or `Deprecated` are skipped during serialization and deserialization.

## Field Naming Rules

- Native C++ structures export the authored reflected property name.
- Blueprint user-defined structures export the authored Blueprint field name.
- Compiled Blueprint GUID suffixes are not exported as JSON keys.
- `DisplayName` metadata is not used as the exported JSON key.
- Import prefers exact key matches, then accepts compatible legacy/internal name variants and case-insensitive matches.
- Export fails when two serializable fields in the same structure would produce duplicate JSON keys when compared case-insensitively.

Use unique authored field names and use the exported key spelling when creating ignore paths.

## Deserialization Rules

- The JSON root must be an object, not an array or scalar value.
- The concrete structure type comes from the connected Blueprint `Out Struct` pin or the C++ template type.
- Missing JSON fields retain the target structure's current values in the C++ template API.
- The Blueprint JSON nodes initialize a temporary target structure first, so missing fields keep that structure type's initialized/default values.
- A JSON `null` value does not overwrite the current property value.
- Extra JSON fields are ignored when at least one field matches the target structure.
- A non-empty JSON object fails when none of its fields match the target structure.
- Arrays must be represented as JSON arrays, sets as JSON arrays, and maps as JSON objects.
- Each imported JSON value must be compatible with its target reflected property type.

Always check `Return Value` before reading output data.

## Error Handling and Troubleshooting

When a node exposes `Out Error`, print or log it after a failed call. Additional diagnostics are written to the Unreal Engine Output Log under `LogStructJsonString`.

| Problem | Likely cause | Resolution |
| --- | --- | --- |
| Node does not appear | Plugin disabled or wrong engine package | Enable the plugin, restart, and install the package for the project's exact engine version. |
| `InStruct is not a valid wildcard USTRUCT parameter` | No concrete structure is connected to the wildcard input | Connect a Make Structure node or a typed structure variable. |
| Unable to retrieve `OutStruct` property/address | The wildcard output has no concrete target type | Connect `Out Struct` to a typed Set or Break Structure node. |
| JSON parse failure | Empty/malformed JSON or a non-object root | Validate the JSON syntax and use an object at the root. |
| No JSON field matches the target structure | Wrong target type or wrong field names | Connect the correct structure type and compare the JSON keys with its authored field names. |
| Array/set/map import failure | Wrong JSON container shape | Use arrays for arrays/sets and objects for maps. |
| Field value import failure | JSON value is incompatible with the property type | Correct the JSON value type and inspect the path in `Out Error`. |
| Invalid `FInstancedStruct` | The value was never initialized or was reset | Initialize it with a concrete structure before converting. |
| Duplicate export key error | Two fields resolve to the same case-insensitive JSON key | Rename one of the authored structure fields. |
| Ignore path removes nothing | Case or path segment does not exactly match exported JSON | Copy the exact exported key spelling and remove array indexes. |

## Runtime and Packaging

StructJsonString contains a Runtime module and does not require an Editor module. The conversion APIs can be used in PIE, standalone play, Development builds, and Shipping builds.

Before releasing a game, test the exact plugin package, engine version, platform, and build configuration used by the project.

## Implementation and Modification Guide

The runtime conversion flow is:

1. Unreal reflection and `FJsonObjectConverter` create or consume a JSON object.
2. The plugin resolves stable authored field names for native and Blueprint structures.
3. Nested structures and containers are processed recursively.
4. Optional ignore rules are applied after serialization.
5. Unreal's pretty or condensed JSON writer produces the final string.

The primary source files are:

| File | Purpose |
| --- | --- |
| `StructStringLibrary.h/.cpp` | Blueprint nodes, C++ template API, serialization, deserialization, field naming, container handling, and ignore rules |
| `StructJsonStringSampleTypes.h` | Blueprint-accessible sample structures |
| `StructJsonStringLog.h/.cpp` | `LogStructJsonString` category |
| `StructJsonString.h/.cpp` | Runtime module startup and shutdown |
| `StructJsonString.Build.cs` | Unreal module dependencies |
| `StructJsonString.uplugin` | Plugin metadata, runtime module, engine version, and platform allow list |

When modifying the plugin:

1. Keep public runtime APIs free of Editor-only module dependencies.
2. Add reflected types through `USTRUCT` and `UPROPERTY` so Unreal's conversion system can inspect them.
3. Update both export rewriting and recursive import behavior when adding custom container or property handling.
4. Preserve the `CustomThunk` stack-reading implementation for wildcard Blueprint pins.
5. Add clear English `Out Error` and `LogStructJsonString` messages for new failure paths.
6. Update the plugin version and this documentation when public behavior changes.
7. Compile and test each supported Unreal Engine version separately.
8. Test Blueprint and C++ round trips in PIE and a packaged Win64 application.

## Known Limitations

- The current plugin descriptor allows `Win64` only.
- A package must match the project's Unreal Engine version.
- JSON-to-structure conversion requires a JSON object at the root.
- Wildcard Blueprint nodes require a concrete connected structure type.
- The JSON text does not carry or infer an Unreal structure type.
- Ignore-by-name is recursive and case-insensitive.
- Ignore-by-path is case-sensitive, uses exact dot-separated keys, and does not support array indexes.
- Map keys and property types must be representable by Unreal Engine's JSON/text conversion systems.
- `DisplayName` metadata does not rename exported JSON keys.
- CustomThunk wildcard functions are Blueprint dispatch entry points, not direct native C++ APIs.
- The original and V2 JSON-to-structure nodes have the same behavior in version `1.4.4`.

## Support

- Documentation: [github.com/givecode/StructJsonString](https://github.com/givecode/StructJsonString)
If you encounter a bug, have a usage question, or would like to request a new feature, please submit an issue on the [GitHub Issues page](https://github.com/givecode/StructJsonString/issues).

When reporting a problem, include:

- StructJsonString plugin version
- Unreal Engine version
- Target platform and build configuration
- Blueprint or C++ usage
- Minimal structure definition or screenshot of the Blueprint graph
- Input JSON, when applicable
- `Out Error` and relevant `LogStructJsonString` output
- Exact reproduction steps

## License

Use of StructJsonString is governed by the license supplied with the Fab product.
