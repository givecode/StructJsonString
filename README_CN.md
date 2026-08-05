# StructJsonString 中文文档

StructJsonString 是一款 Unreal Engine 运行时插件，用于在蓝图和 C++ 中实现 Unreal 结构体与 JSON 字符串之间的双向转换。

插件支持原生 C++ `USTRUCT`、蓝图结构体、通配结构体引脚和 `FInstancedStruct`，不依赖编辑器模块，也不需要第三方库。

## 主要功能

- 将连接到通配蓝图引脚的任意结构体转换为 JSON 字符串。
- 将 JSON 字符串同时转换为通配结构体和 `FInstancedStruct`。
- 将 `FInstancedStruct` 直接转换为 JSON 字符串。
- 支持格式化 JSON 和紧凑 JSON 两种输出方式。
- 按字段名递归忽略字段。
- 按点号分隔的 JSON 路径精确忽略字段。
- 为原生 C++ 结构体和蓝图结构体生成稳定的字段名。
- 通过返回值、错误字符串和 Unreal Engine 输出日志报告转换错误。
- 可在打包后的运行时程序中使用。

## 兼容性

- Unreal Engine 5.6
- Unreal Engine 5.7
- Unreal Engine 5.8
- 模块类型：Runtime

请使用与 Unreal Engine 版本完全对应的插件包。目标平台支持范围以相应 Fab 商品页面和插件包说明为准。

## 安装方法

1. 关闭 Unreal Engine。
2. 将 `StructJsonString` 文件夹复制到项目的 `Plugins` 目录：

   ```text
   YourProject/
   └── Plugins/
       └── StructJsonString/
           ├── Source/
           └── StructJsonString.uplugin
   ```

3. 打开项目。
4. 进入 **编辑（Edit）> 插件（Plugins）**，搜索 `StructJsonString`。
5. 启用插件，并在提示时重启 Unreal Engine。
6. 如果是 C++ 项目，请重新生成项目文件并编译项目。

## 蓝图节点说明

所有蓝图节点均位于 `StructJsonString` 分类中。

### Convert Struct to JSON String

中文含义：将结构体转换为 JSON 字符串。

该节点会把连接到通配 `In Struct` 引脚的结构体转换为 JSON。

主要输出和参数：

- `Return Value`：转换成功时返回 `true`。
- `Out Json`：生成的 JSON 字符串。
- `Out Error`：转换失败时返回错误信息。
- `Pretty Json`：为 `true` 时输出带缩进的格式化 JSON；为 `false` 时输出紧凑 JSON。

### Convert Struct to JSON String (Ignore by Name)

中文含义：将结构体转换为 JSON，并按字段名忽略字段。

它与普通转换节点的用法相同，但会删除名称出现在 `Ignore Fields` 中的所有字段。匹配不区分大小写，并且会递归处理所有嵌套层级。

例如，忽略 `InternalNote` 后，顶层、嵌套结构体以及结构体数组内所有名为 `InternalNote` 的字段都会被删除。

### Convert Struct to JSON String (Ignore by Path)

中文含义：将结构体转换为 JSON，并按字段路径精确忽略字段。

该节点通过 `Ignore Paths` 中的点号分隔路径，只删除指定位置的字段。

示例路径：

```text
PrimaryItem.InternalNote
Profile.Contact.Email
```

当同名字段出现在多个位置，而你只想删除其中一个时，应使用此节点。

### Convert Instanced Struct to JSON String

中文含义：将实例化结构体转换为 JSON 字符串。

该节点将有效的 `FInstancedStruct` 转换为 JSON。如果输入没有包含有效结构体，节点会返回 `false`。

### Convert JSON String to Instanced Struct and Any Struct

中文含义：将 JSON 字符串转换为实例化结构体和任意结构体。

该节点会把一个 JSON 对象反序列化为两个输出：

- `Out Instanced`：包含目标结构体类型和数据的 `FInstancedStruct`。
- `Out Struct`：通配结构体输出，其类型由连接到该引脚的结构体决定。
- `Out Error`：解析或转换失败时的错误信息。

必须把一个具体类型的结构体变量连接到 `Out Struct`，节点才能确定目标类型。

### Convert JSON String to Instanced Struct and Any Struct V2

该节点提供与上一个节点相同的输出形式，是 V2 蓝图入口。新建蓝图逻辑时建议优先使用 V2；旧项目如果已经使用前一个节点，可以继续保留。

## 蓝图使用流程

### 结构体转换为 JSON

1. 创建或获取一个结构体变量。
2. 添加 **Convert Struct to JSON String** 节点。
3. 将结构体连接到 `In Struct`。
4. 通过 `Pretty Json` 选择格式化输出或紧凑输出。
5. 使用 `Out Json` 之前，先检查 `Return Value`。
6. 如果失败，查看 `Out Error` 和输出日志。

### JSON 转换为结构体

1. 准备一个合法的 JSON 对象字符串。
2. 添加 **Convert JSON String to Instanced Struct and Any Struct V2** 节点。
3. 将目标类型的结构体变量连接到 `Out Struct`。
4. 读取输出之前，先检查 `Return Value`。
5. 通过 `Out Error` 排查 JSON 格式错误或数据类型不兼容问题。

## JSON 示例

假设结构体中包含 ID、标题、开关、分数、嵌套结构体、数组、集合和 Map，格式化输出可能如下：

```json
{
  "Id": 7,
  "Title": "StructJsonString 中文测试",
  "bEnabled": true,
  "Score": 91.5,
  "PrimaryItem": {
    "Name": "Sword",
    "Damage": 25,
    "InternalNote": "忽略后不会导出"
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

JSON 键名使用插件解析出的稳定原始字段名。导入时还会兼容 Unreal/FJsonObjectConverter 风格的名称变体，并进行不区分大小写的后备匹配。

## 支持的数据形式

插件基于 Unreal Engine 反射系统和 `FJsonObjectConverter` 实现，可处理 Unreal JSON 转换系统支持的反射属性。当前插件源码明确演示并支持以下数据形式：

- 布尔值、整数和浮点数
- 字符串及其他兼容的反射标量属性
- 原生 C++ 结构体和蓝图结构体
- 嵌套结构体
- 数组 `TArray`
- 集合 `TSet`
- 使用 JSON 兼容键的 Map `TMap`
- `FInstancedStruct`

JSON 数据必须与连接的目标结构体及其反射属性类型兼容。

## C++ 使用方法

在项目模块的依赖中添加插件模块：

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

### 将原生 USTRUCT 转换为 JSON

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

`FMyPayload` 必须是提供 `StaticStruct()` 的反射结构体。

### 将 JSON 转换为原生 USTRUCT

```cpp
FMyPayload Payload;
FString Error;

const bool bSuccess = UStructStringLibrary::JsonStringToStruct(
    Json,
    Payload,
    &Error
);
```

### 将 FInstancedStruct 转换为 JSON

```cpp
FString Json;
const bool bSuccess = UStructStringLibrary::InstancedStructToJsonString(
    InstancedPayload,
    Json,
    true
);
```

## 错误处理

请始终检查布尔返回值。如果节点或 C++ 函数提供错误字符串，应在继续处理前检查该字符串。

常见失败原因包括：

- 输入 JSON 为空或语法错误。
- JSON 根节点不是对象。
- 通配输出引脚没有连接具体的结构体类型。
- `FInstancedStruct` 无效或为空。
- JSON 值无法转换为目标属性类型。
- 属性名称冲突，导致生成重复的 JSON 键。

插件还会把详细诊断信息写入 Unreal Engine 输出日志的 `LogStructJsonString` 分类。

## 重要行为与限制

- JSON 反序列化要求根节点必须是 JSON 对象。
- JSON 中缺少的字段会保留结构体当前值或默认值。
- JSON 中的 `null` 不会覆盖属性当前值。
- 按字段名忽略时不区分大小写，并会递归影响所有同名字段。
- 按路径忽略时使用点号分隔的字段路径。
- Map 在 JSON 中表示为对象，因此 Map 键必须能够表示为 JSON 字段名。
- 通配蓝图节点必须连接具体结构体，Unreal 才能解析结构体类型。
- 必须使用与当前 Unreal Engine 版本匹配的插件包。

## 运行时与打包

StructJsonString 是运行时模块。启用插件后，可以在打包程序中调用其蓝图和 C++ 接口。正式发布项目前，建议使用项目实际采用的 Development 和 Shipping 配置分别测试。

## 技术支持

如需报告问题或提出功能建议，请在该文档仓库中创建 Issue，或者使用 Fab 商品页面提供的支持链接。

报告问题时建议附上：

- 插件版本
- Unreal Engine 版本
- 目标平台
- 蓝图或 C++ 使用方式
- 输入 JSON 或最小结构体定义
- 相关 `LogStructJsonString` 日志

## 许可说明

插件的使用许可，以 Fab 商品随附的许可条款为准。
