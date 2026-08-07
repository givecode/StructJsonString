# StructJsonString for Unreal Engine 5 (UE5)

[English Documentation](README.md)

StructJsonString 是一个 Unreal Engine 运行时代码插件，用于在蓝图和 C++ 中，将受支持的反射 `USTRUCT` 值与 JSON 字符串相互转换。

插件支持原生 C++ 结构体、蓝图用户自定义结构体、通配结构体引脚、嵌套容器以及 `FInstancedStruct`。插件使用 Unreal Engine 反射系统及引擎内置的 JSON 模块，不需要任何第三方库、外部服务、账号或网络连接。

## 功能特性

- 将蓝图结构体或原生 C++ 结构体转换为 JSON 字符串。
- 将 JSON 对象同时转换为指定类型的通配结构体和 `FInstancedStruct`。
- 将 `FInstancedStruct` 直接转换为 JSON。
- 生成带缩进的易读 JSON（Pretty JSON）或紧凑 JSON（Compact JSON）。
- 按字段名称递归忽略匹配的 JSON 字段。
- 按精确的点号分隔路径忽略指定 JSON 字段。
- 保留 C++ 和蓝图中定义的字段名称，不暴露蓝图编译后产生的 GUID 后缀。
- 支持嵌套结构体、数组、集合和映射。
- 通过布尔返回值、错误字符串及 `LogStructJsonString` 报告转换失败信息。
- 可在编辑器、PIE、独立运行模式和打包后的运行时程序中使用。

## 兼容性与要求

| 项目 | 支持范围 |
| --- | --- |
| 插件版本 | `1.4.4` |
| Unreal Engine | `5.6`、`5.7` 和 `5.8` |
| 目标平台 | Windows 64 位（`Win64`） |
| 模块类型 | Runtime |
| 蓝图支持 | 是 |
| C++ 支持 | 是 |
| 第三方软件 | 无 |

请使用与项目 Unreal Engine 版本完全一致的插件安装包。每个安装包中的 `.uplugin` 描述文件会标明该安装包对应的引擎版本。

## 下载与安装

### 从 Fab 安装

1. 将本产品添加到你的 Fab 资源库。
2. 通过 Epic Games Launcher，为所需的 Unreal Engine 版本安装插件。
3. 打开目标项目。
4. 在 Unreal Editor 中打开 **Edit > Plugins**。
5. 搜索 **StructJsonString**，启用插件，并在出现提示时重启 Unreal Editor。

### 从提供的 ZIP 安装包安装

1. 从产品的 Project File Link 下载 ZIP 文件。
2. 关闭 Unreal Editor 和 IDE。
3. 解压 ZIP，将其中唯一的 `StructJsonString` 插件文件夹放入项目的 `Plugins` 目录。

最终路径必须为：

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

不要在 `Plugins` 和 `StructJsonString` 之间保留额外的压缩包目录层级。

4. 打开项目。
5. 打开 **Edit > Plugins**，搜索 **StructJsonString** 并启用插件。
6. 出现提示时重启 Unreal Editor。
7. 如果是 C++ 项目，请在必要时重新生成项目文件，并至少编译一次项目。

### 验证安装

打开任意蓝图的 Event Graph，单击右键并搜索：

```text
Convert Struct to JSON String
```

该节点应显示在 **StructJsonString** 分类中。如果节点没有出现，请确认插件已经启用，并确认插件安装包与项目所用的 Unreal Engine 版本一致。

## 5 分钟蓝图快速测试

本测试用于验证完整的蓝图工作流：创建结构体、将其序列化为 JSON，再将 JSON 反序列化回相同的结构体类型。

### 第 1 步：创建测试结构体

1. 在 Content Browser 中选择 **Add > Blueprints > Structure**。
2. 将结构体命名为 `ST_StructJsonTest`。
3. 严格按照下表添加并命名字段：

| 字段名称 | 蓝图类型 | 测试值 |
| --- | --- | --- |
| `Id` | Integer | `7` |
| `Title` | String | `StructJsonString Sample` |
| `Enabled` | Boolean | `true` |

4. 保存结构体。

### 第 2 步：将结构体转换为 JSON

1. 创建名为 `BP_StructJsonStringTest` 的 Actor Blueprint。
2. 打开其 Event Graph，并添加 **Event BeginPlay**。
3. 添加 **Make ST_StructJsonTest**，填入上表中的三个测试值。
4. 添加 **Convert Struct to JSON String**。
5. 将 **Make ST_StructJsonTest** 的输出连接到 `In Struct`。
6. 保持 `Pretty Json` 为启用状态。
7. 将执行流从 **Event BeginPlay** 连接到转换节点。
8. 使用 Branch 检查 `Return Value`。
9. 成功时，将 `Out Json` 连接到 **Print String**。
10. 失败时，将 `Out Error` 连接到 **Print String**。
11. 编译蓝图，将其放入关卡，然后按下 Play。

输出的 JSON 必须包含以下数据。空白字符和字段顺序不影响结果。

```json
{
  "Id": 7,
  "Title": "StructJsonString Sample",
  "Enabled": true
}
```

### 第 3 步：将 JSON 转换回结构体

1. 在成功执行序列化节点后，添加 **Convert JSON String to Instanced Struct and Any Struct V2**。
2. 将第一个节点的 `Out Json` 连接到 V2 节点的 `In Json String`。
3. 添加 **Break ST_StructJsonTest**。
4. 将 V2 节点的通配 `Out Struct` 引脚连接到 **Break ST_StructJsonTest** 的结构体输入引脚。建立该连接后，通配引脚会确定为具体结构体类型。
5. 连接执行流，并使用 Branch 检查 V2 节点的 `Return Value`。
6. 成功时，打印还原后的 `Id`、`Title` 和 `Enabled`。
7. 失败时，打印 `Out Error`。

如果还原出的值分别为 `7`、`StructJsonString Sample` 和 `true`，则往返转换成功。`Out Instanced` 中还会包含一个有效的 `FInstancedStruct`，其内部类型为 `ST_StructJsonTest`。

## 蓝图节点参考

所有蓝图节点均位于 **StructJsonString** 分类中。

### Convert Struct to JSON String

将连接到通配输入引脚的反射结构体转换为 JSON。

| 方向 | 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| 输入 | `In Struct` | 通配结构体 | 必填 | 要序列化的结构体值，必须连接一个具体的 `USTRUCT` 值。 |
| 输入 | `Pretty Json` | Boolean | `true` | 启用时生成带缩进的 JSON；禁用时生成紧凑 JSON。 |
| 输出 | `Out Json` | String | — | 生成的 JSON 文本。转换开始前会被重置为空。 |
| 输出 | `Out Error` | String | — | 详细的失败原因。调用成功后为空。 |
| 输出 | `Return Value` | Boolean | — | 转换成功时为 `true`。 |

### Convert Struct to JSON String (Ignore by Name)

序列化通配结构体，并删除名称与 `Ignore Fields` 中任意条目匹配的所有 JSON 字段。

| 方向 | 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| 输入 | `In Struct` | 通配结构体 | 必填 | 要序列化的结构体值。 |
| 输入 | `Ignore Fields` | Name 数组 | 空 | 要删除的字段名。匹配不区分大小写，并递归作用于所有层级。 |
| 输入 | `Pretty Json` | Boolean | `true` | 选择带缩进或紧凑的输出格式。 |
| 输出 | `Out Json` | String | — | 删除匹配字段后生成的 JSON 文本。 |
| 输出 | `Out Error` | String | — | 详细的失败原因。 |
| 输出 | `Return Value` | Boolean | — | 转换成功时为 `true`。 |

例如：忽略 `InternalNote` 会删除根对象中的该字段、嵌套对象中的该字段、数组内对象中的该字段，以及表示 Map 的 JSON 对象中名称匹配的键。

### Convert Struct to JSON String (Ignore by Path)

序列化通配结构体，并根据精确的点号分隔 JSON 路径删除指定字段。

| 方向 | 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| 输入 | `In Struct` | 通配结构体 | 必填 | 要序列化的结构体值。 |
| 输入 | `Ignore Paths` | Name 数组 | 空 | 要删除的精确点号分隔导出 JSON 路径。 |
| 输入 | `Pretty Json` | Boolean | `true` | 选择带缩进或紧凑的输出格式。 |
| 输出 | `Out Json` | String | — | 删除指定字段后生成的 JSON 文本。 |
| 输出 | `Out Error` | String | — | 详细的失败原因。 |
| 输出 | `Return Value` | Boolean | — | 转换成功时为 `true`。 |

路径示例：

```text
InternalNote
PrimaryItem.InternalNote
Items.InternalNote
```

- `InternalNote` 只删除根对象中的字段。
- `PrimaryItem.InternalNote` 只删除该嵌套字段。
- `Items.InternalNote` 删除 `Items` 数组中每个对象内的该字段。
- 路径匹配区分大小写。
- 每一段都必须使用准确的导出 JSON 键名。
- 不支持 `Items[0].InternalNote` 这样的数组索引语法。
- 如果字段名称或 Map Key 中包含英文句点（`.`），路径会产生歧义。

### Convert Instanced Struct to JSON String

将有效的 `FInstancedStruct` 值转换为 JSON。

| 方向 | 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| 输入 | `In Struct` | `FInstancedStruct` | 必填 | 已使用具体结构体类型初始化的有效实例化结构体。 |
| 输入 | `Pretty Json` | Boolean | `true` | 选择带缩进或紧凑的输出格式。 |
| 输出 | `Out Json String` | String | — | 生成的 JSON 文本。 |
| 输出 | `Return Value` | Boolean | — | 输入为空、无效或转换失败时为 `false`。 |

该节点不提供 `Out Error`。节点失败时，请在 Unreal Engine Output Log 中查看 `LogStructJsonString` 分类下的日志。

### Convert JSON String to Instanced Struct and Any Struct

将 JSON 对象反序列化为一个指定类型的通配结构体和一个 `FInstancedStruct`。

| 方向 | 参数 | 类型 | 是否必需 | 说明 |
| --- | --- | --- | --- | --- |
| 输入 | `In Json String` | String | 是 | 根值为对象的 JSON 文本。 |
| 输出 | `Out Instanced` | `FInstancedStruct` | — | 使用与 `Out Struct` 相同的类型和数据进行初始化。 |
| 输出 | `Out Error` | String | — | 详细的失败原因。 |
| 输出 | `Out Struct` | 通配结构体 | 是 | 连接到指定类型的 Set/Break Structure 节点，或指定类型的结构体变量。 |
| 输出 | `Return Value` | Boolean | — | 只有在反序列化及两个输出均成功时才为 `true`。 |

连接到 `Out Struct` 的具体类型决定目标结构体类型。插件不会根据 JSON 文本推断目标结构体类型。

### Convert JSON String to Instanced Struct and Any Struct V2

V2 节点具有相同的参数，并且在插件 `1.4.4` 版本中，与原始节点使用相同的内部转换实现。快速入门示例使用 V2 节点；原始节点继续保留，以兼容已有的蓝图图表。

## 内置示例结构体

源码中包含两个可在蓝图中使用的示例类型：

- `FStructJsonStringSampleItem`
  - `Name`（`FString`）
  - `Damage`（`int32`）
  - `InternalNote`（`FString`）
- `FStructJsonStringSamplePayload`
  - `Id`（`int32`）
  - `Title`（`FString`）
  - `bEnabled`（`bool`）
  - `Score`（`double`）
  - `PrimaryItem`（嵌套示例结构体）
  - `Items`（示例结构体数组）
  - `Tags`（字符串数组）
  - `UniqueCodes`（字符串集合）
  - `Attributes`（字符串到整数的映射）
  - `InternalNote`（`FString`）

无需在使用插件的项目中添加 C++ 代码，即可使用这些示例类型测试嵌套结构体、容器及两种忽略模式。

示例数据：

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

设置 `Ignore Fields = [InternalNote]` 时，两个 `InternalNote` 字段都会被忽略。设置 `Ignore Paths = [PrimaryItem.InternalNote]` 时，只会忽略嵌套字段。

## C++ 接入

### 模块依赖

根据代码使用 API 的位置，将 `StructJsonString` 添加到使用方模块的 Public 或 Private 依赖中：

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

包含公开头文件：

```cpp
#include "StructStringLibrary.h"
```

插件自身使用 Unreal Engine 内置的 `Json` 和 `JsonUtilities` 模块，不需要下载或集成任何第三方依赖。

### 定义原生结构体

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

C++ 模板函数要求使用能够提供 `StaticStruct()` 的反射结构体类型。

### 将原生结构体与 JSON 相互转换

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

`StructToJsonString` 通过布尔返回值和 Output Log 报告失败。`JsonStringToStruct` 还可以通过可选的 `OutError` 指针返回详细错误信息。

### 将 FInstancedStruct 转换为 JSON

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

### Blueprint CustomThunk 函数与 C++ 直接调用

使用 `CustomThunk` 声明的通配函数用于接收蓝图中动态确定类型的结构体引脚。它们的普通原生函数体只是占位实现；从 C++ 直接调用时会返回 `false`。

C++ 中请改用以下原生入口：

- `UStructStringLibrary::StructToJsonString<TStruct>()`
- `UStructStringLibrary::JsonStringToStruct<TStruct>()`
- `UStructStringLibrary::InstancedStructToJsonString()`

通配结构体转换、按名称忽略、按路径忽略以及 JSON 双输出节点应在蓝图图表中使用。

## 支持的数据与 JSON 表示形式

插件支持 Unreal Engine `FJsonObjectConverter` 能够处理的反射值，并对嵌套容器进行显式处理。

| Unreal 数据形式 | JSON 表示形式 |
| --- | --- |
| Boolean | Boolean |
| 整数和浮点数 | Number |
| String 及兼容的标量属性 | String 或 Unreal 转换器对应的表示形式 |
| 原生或蓝图 `USTRUCT` | Object |
| 嵌套结构体 | 嵌套 Object |
| `TArray` | Array |
| `TSet` | Array |
| `TMap` | Object，字段名称表示 Map Key |
| `FInstancedStruct` | 表示其内部结构体的 Object |

对于 `FString`、`FName` 和 `FText`，Map Key 会被直接导入。其他 Map Key 属性类型必须能够从 JSON 字段名称执行 Unreal 文本导入。Unreal Engine JSON 转换系统无法表示的属性不受支持。

标记为 `Transient` 或 `Deprecated` 的属性在序列化和反序列化时会被跳过。

## 字段命名规则

- 原生 C++ 结构体导出其定义时使用的反射属性名。
- 蓝图用户自定义结构体导出其在蓝图中定义的字段名。
- 蓝图编译生成的 GUID 后缀不会作为 JSON 键导出。
- `DisplayName` 元数据不会用来重命名导出的 JSON 键。
- 导入时优先精确匹配键名，然后接受兼容的旧名称或内部名称变体，以及不区分大小写的匹配。
- 如果同一结构体中的两个可序列化字段在进行不区分大小写的比较后会生成重复 JSON 键，导出将失败。

请使用唯一的字段名称；创建忽略路径时，应使用实际导出的键名拼写。

## 反序列化规则

- JSON 根值必须是 Object，不能是 Array 或标量值。
- 具体结构体类型来自蓝图中连接的 `Out Struct` 引脚，或者 C++ 模板类型。
- 在 C++ 模板 API 中，JSON 中缺少的字段会保留目标结构体的当前值。
- 蓝图 JSON 节点会先初始化一个临时目标结构体，因此缺少的字段会保留该结构体类型的初始化值或默认值。
- JSON 中的 `null` 不会覆盖属性的当前值。
- 只要至少有一个字段与目标结构体匹配，多余的 JSON 字段就会被忽略。
- 对于非空 JSON 对象，如果其中没有任何字段能与目标结构体匹配，则转换失败。
- 数组必须使用 JSON Array 表示，集合必须使用 JSON Array 表示，映射必须使用 JSON Object 表示。
- 导入的每个 JSON 值必须与其目标反射属性类型兼容。

读取输出数据前，务必先检查 `Return Value`。

## 错误处理与故障排查

节点提供 `Out Error` 时，应在调用失败后将其打印或写入日志。其他诊断信息会写入 Unreal Engine Output Log 的 `LogStructJsonString` 分类。

| 问题 | 可能原因 | 解决方法 |
| --- | --- | --- |
| 节点没有出现 | 插件未启用，或者安装包的引擎版本错误 | 启用插件并重启，然后安装与项目引擎版本完全一致的插件包。 |
| `InStruct is not a valid wildcard USTRUCT parameter` | 通配输入没有连接具体结构体 | 连接 Make Structure 节点或指定类型的结构体变量。 |
| Unable to retrieve `OutStruct` property/address | 通配输出没有确定的目标类型 | 将 `Out Struct` 连接到指定类型的 Set 或 Break Structure 节点。 |
| JSON 解析失败 | JSON 为空、格式错误或根值不是 Object | 检查 JSON 语法，并使用 Object 作为根值。 |
| 没有 JSON 字段与目标结构体匹配 | 目标类型错误或字段名称错误 | 连接正确的结构体类型，并将 JSON 键与结构体中定义的字段名进行比较。 |
| 数组、集合或映射导入失败 | JSON 容器形式错误 | 数组和集合使用 Array，映射使用 Object。 |
| 字段值导入失败 | JSON 值与属性类型不兼容 | 修正 JSON 值类型，并检查 `Out Error` 中的字段路径。 |
| `FInstancedStruct` 无效 | 该值从未初始化或已被重置 | 转换前先使用具体结构体类型对其进行初始化。 |
| 导出键重复错误 | 两个字段产生了相同的不区分大小写 JSON 键 | 重命名其中一个结构体字段。 |
| 忽略路径没有删除任何字段 | 大小写或路径段与导出的 JSON 不完全一致 | 复制实际导出的键名拼写，并移除数组索引。 |

## 运行时与打包

StructJsonString 包含 Runtime 模块，不需要 Editor 模块。转换 API 可以在 PIE、独立运行模式、Development 构建和 Shipping 构建中使用。

发布游戏前，请使用项目实际采用的插件安装包、引擎版本、目标平台和构建配置完成测试。

## 实现与修改指南

运行时转换流程如下：

1. Unreal 反射系统和 `FJsonObjectConverter` 创建或读取 JSON Object。
2. 插件为原生结构体和蓝图结构体解析稳定的原始字段名。
3. 递归处理嵌套结构体和容器。
4. 序列化完成后应用可选的忽略规则。
5. 使用 Unreal 的 Pretty 或 Condensed JSON Writer 生成最终字符串。

主要源码文件：

| 文件 | 用途 |
| --- | --- |
| `StructStringLibrary.h/.cpp` | 蓝图节点、C++ 模板 API、序列化、反序列化、字段命名、容器处理和忽略规则 |
| `StructJsonStringSampleTypes.h` | 可在蓝图中使用的示例结构体 |
| `StructJsonStringLog.h/.cpp` | `LogStructJsonString` 日志分类 |
| `StructJsonString.h/.cpp` | Runtime 模块的启动和关闭 |
| `StructJsonString.Build.cs` | Unreal 模块依赖 |
| `StructJsonString.uplugin` | 插件元数据、Runtime 模块、引擎版本和平台允许列表 |

修改插件时：

1. 确保公开的运行时 API 不依赖任何仅限 Editor 的模块。
2. 使用 `USTRUCT` 和 `UPROPERTY` 添加反射类型，使 Unreal 转换系统能够检查这些类型。
3. 添加自定义容器或属性处理时，同时更新导出名称重写逻辑和递归导入行为。
4. 保留用于通配蓝图引脚的 `CustomThunk` 栈读取实现。
5. 为新增的失败路径添加清晰的英文 `Out Error` 和 `LogStructJsonString` 信息。
6. 公共行为发生变化时，同时更新插件版本和本文档。
7. 分别编译并测试每个受支持的 Unreal Engine 版本。
8. 在 PIE 和打包后的 Win64 应用中，分别测试蓝图与 C++ 的往返转换。

## 已知限制

- 当前插件描述文件仅允许 `Win64`。
- 插件安装包必须与项目使用的 Unreal Engine 版本一致。
- JSON 转结构体要求 JSON 根值为 Object。
- 通配蓝图节点必须连接一个确定的具体结构体类型。
- JSON 文本本身不包含 Unreal 结构体类型，也不能用于自动推断类型。
- 按名称忽略为递归操作，并且不区分大小写。
- 按路径忽略区分大小写，使用精确的点号分隔键名，并且不支持数组索引。
- Map Key 和属性类型必须能够由 Unreal Engine 的 JSON 或文本转换系统表示。
- `DisplayName` 元数据不会重命名导出的 JSON 键。
- CustomThunk 通配函数是蓝图分发入口，不是用于 C++ 直接调用的原生 API。
- 在 `1.4.4` 版本中，原始 JSON 转结构体节点与 V2 节点的行为相同。

## 文档与技术支持

- 英文文档：[github.com/givecode/StructJsonString](https://github.com/givecode/StructJsonString)
- Bug 反馈、使用问题和功能需求：[GitHub Issues](https://github.com/givecode/StructJsonString/issues)

提交问题时，请提供：

- StructJsonString 插件版本
- Unreal Engine 版本
- 目标平台和构建配置
- 使用蓝图还是 C++
- 最小结构体定义，或蓝图图表截图
- 输入 JSON（如适用）
- `Out Error` 和相关的 `LogStructJsonString` 输出
- 可准确复现问题的步骤

## 许可证

StructJsonString 的使用遵循 Fab 产品随附的许可证。
