<h1 align="center">Inventor Dev Skills</h1>

<p align="center">面向 Autodesk Inventor C# 进程内插件（Add-In）开发的模块化 Agent 技能库。</p>

<p align="center">
  <a href="./README.md">English</a> | <a href="./README.zh-CN.md">简体中文</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/C%23-.NET%208%20%2F%20.NET%204.8-3776AB?style=flat-square" alt="C# .NET 8 and .NET 4.8"> <img src="https://img.shields.io/badge/Autodesk%20Inventor-Windows%20x64-4B5563?style=flat-square" alt="Autodesk Inventor on Windows x64"> <a href="./SKILL.md"><img src="https://img.shields.io/badge/Skills-1%20Master%20%2B%2020%20Focused-4F46E5?style=flat-square" alt="1 Master Skill and 20 Focused Skills"></a> <a href="./LICENSE"><img src="https://img.shields.io/badge/License-Apache%202.0-22C55E?style=flat-square" alt="License: Apache 2.0"></a>
</p>

Inventor Dev Skills 是一套面向 **Autodesk Inventor C# 进程内 Add-In 二次开发** 的工程技能规范库。它将 Inventor 开发中高频复现的故障模式——目标框架选错、`Any CPU` 编译、`.addin` 清单与 GUID 失配、命名空间冲突、厘米数据库单位误用、后台线程访问 COM 对象、模型重生导致几何引用失效以及工程图自动标注不稳定——沉淀为按任务拆分的可复用 Agent 技能。

本仓库仅覆盖进程内 Inventor C# Add-In 开发，不包含 MCP Server、独立进程自动化 EXE、AI 对话/工作流架构或 Fusion 360 API。

## 核心亮点

| 亮点 | 工程价值 |
| --- | --- |
| 1 个主路由技能 + 20 个专项技能 | 先加载 [`SKILL.md`](./SKILL.md)，再按任务按需引入细分领域的 `SKILL.md`，保持 Agent 上下文精简，避免无关 API 干扰。 |
| 双运行时版本基线 | 明确区分 Inventor 2025+（`.NET 8` / `net8.0-windows`、Visual Studio 2022 17.8+）与 Inventor 2024 及更早版本（`net48` 或对应 SDK 目标），防止跨版本强行套用。 |
| COM 互操作与类型冲突护栏 | 统一约束高频冲突类型别名（`System.IO.Path` 与 `Inventor.Path`、`Inventor.Application` 与 `System.Windows.Application` 等）、主线程 COM 访问规范以及禁止空 `catch` 的 HRESULT 诊断日志。 |
| 确定性建模与工程图规则 | 倡导单次 COM 读取转纯 C# DTO、使用 `AttributeSets`/`ReferenceKey` 替代脆弱集合索引，以及确定性的自动尺寸规划管线。 |
| 无注册表部署与排障指引 | 覆盖 Windows 支持的 `.addin` 部署目录、PowerShell 变量插值陷阱，以及构建、加载、COM、BOM 和部署阶段的故障特征排查。 |

## 架构概览

仓库以 [`SKILL.md`](./SKILL.md) 作为总入口路由，先确立开发边界与强制规则，再将具体任务分发到 `skills/` 下的 20 个专项技能：

```text
┌────────────────────────────────────────────────────────────────────────────────┐
│                         Master Skill (SKILL.md)                                │
│    Scope Guard | Mandatory Rules | Version Baseline | Task-to-Skill Router     │
└───────────────────────────────────────┬────────────────────────────────────────┘
                                        │ load only task-relevant skills
                                        ▼
┌────────────────────────────────────────────────────────────────────────────────┐
│                          skills/ (20 Focused Skills)                           │
│                                                                                │
│  ┌───────────────────────────────────┐  ┌───────────────────────────────────┐  │
│  │ Add-In Foundation & Interop       │  │ Core Modeling & Assemblies        │  │
│  │ - api-map                         │  │ - documents-units-transactions    │  │
│  │ - project-setup                   │  │ - geometry-selection-references   │  │
│  │ - addin-lifecycle                 │  │ - sketch-modeling                 │  │
│  │ - installation-deployment         │  │ - part-features                   │  │
│  │ - ribbon-events                   │  │ - sheet-metal                     │  │
│  │ - com-interop-conflicts           │  │ - assembly                        │  │
│  └───────────────────────────────────┘  └───────────────────────────────────┘  │
│                                                                                │
│  ┌───────────────────────────────────┐  ┌───────────────────────────────────┐  │
│  │ Drawings & Annotations            │  │ Data, I/O & Reliability           │  │
│  │ - drawing-sheets-tables           │  │ - bom-properties                  │  │
│  │ - drawing-views                   │  │ - import-export                   │  │
│  │ - drawing-dimensions              │  │ - performance-reliability         │  │
│  │ - drawing-annotations             │  │ - troubleshooting                 │  │
│  └───────────────────────────────────┘  └───────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────────┘
```

## 快速安装

将仓库克隆到本地，以便在编码 Agent 或工作区中加载 [`SKILL.md`](./SKILL.md) 及 `skills/` 下的专项技能文件：

```bash
git clone https://github.com/zly258/inventor-dev-skills.git
```

目标 Inventor Add-In 项目的环境前提：

- Windows（`x64`），已安装目标版本的 Autodesk Inventor 及对应的 Add-In SDK/项目模板。
- **Inventor 2025 及以上：** Visual Studio 2022 17.8+、`net8.0-windows`、`x64`、无注册表 `.addin` 清单部署。
- **Inventor 2024 及更早版本：** 使用对应版本 Autodesk SDK/模板支持的 .NET 目标框架（常见为 `net48`）。

## 快速开始

1. 首先加载仓库级主技能 [`SKILL.md`](./SKILL.md)，确保任务范围限制和强制工程规则生效。
2. 仅按当前任务加载所需的专项技能。常见任务组合如下：

| 任务目标 | 配合 `SKILL.md` 加载的专项技能 |
| --- | --- |
| 构建 Inventor Add-In 插件 | `project-setup`、`addin-lifecycle`、`installation-deployment`、`ribbon-events`、`com-interop-conflicts` |
| 创建参数化零件 | `documents-units-transactions`、`geometry-selection-references`、`sketch-modeling`、`part-features` |
| 开发装配体命令 | `documents-units-transactions`、`geometry-selection-references`、`assembly` |
| 自动生成制造工程图 | `drawing-sheets-tables`、`drawing-views`、`drawing-dimensions`、`drawing-annotations`、`bom-properties`、`performance-reliability` |
| 排查编译或加载失败 | `project-setup`、`addin-lifecycle`、`installation-deployment`、`com-interop-conflicts`、`troubleshooting` |

3. 修改已有插件项目前，先核查目标 Inventor 版本、目标框架、`x64` 构建配置、互操作程序集引用、现有 `AddInServer` GUID、`.addin` 清单与部署目录。
4. 交付前按 [`SKILL.md`](./SKILL.md) 完成检查：干净的 `x64` 构建且无新增告警、全新启动 Inventor 正常加载、正确拦截错误文档类型、异常时事务回滚，以及重复执行不产生重复图元。

## 兼容性与核心规范

### 运行时兼容基线

- **Inventor 2025+：** Autodesk 自 Inventor 2025 起将 Add-In 模板迁移至 `.NET 8`（`net8.0-windows`），推荐 `x64` 构建并采用无注册表 `.addin` 部署。
- **Inventor 2024 及更早版本：** 遵循对应版本 Autodesk SDK/模板支持的框架（`net48` 为常见传统基线，未经验证不要强行套用于新版 Inventor）。
- **跨代版本共存：** 可在合理范围内共享核心业务源码，同时为不同运行时世代分别生成对应的 Add-In 构建产物与 `.addin` 清单。

### 为冲突类型声明显式别名

在同时引用 `System.*`、WPF 与 `Inventor.*` 时，显式限定高频冲突类型：

```csharp
using IOPath = System.IO.Path;
using IOFile = System.IO.File;
using InventorPath = Inventor.Path;
using InventorApplication = Inventor.Application;
using WpfApplication = System.Windows.Application;
using InventorTransaction = Inventor.Transaction;
```

文件系统路径使用 `System.IO.Path`，扫掠或建模路径使用 `Inventor.Path`。

### 保持 Inventor COM 对象在主线程访问

禁止在 `Task.Run` 后台线程中直接操作实时 Inventor COM 对象：

```csharp
// 错误做法
await Task.Run(() => part.ComponentDefinition.Features.Count);
```

推荐采用四步数据流：

1. 在 Inventor 线程读取所需状态；
2. 转换为纯 C# DTO 对象；
3. 如有需要，在后台线程执行纯计算；
4. 回到 Inventor 上下文统一提交模型或图纸修改。

### 严格遵守内部数据库单位

无论界面显示单位是否为毫米，Inventor 内部数据库单位固定为：**长度采用厘米（cm）、角度采用弧度（rad）、质量采用千克（kg）**。所有 API 边界处必须通过 `UnitsOfMeasure` 完成换算。

### 确定性工程图自动标注

避免无界候选池重试循环，采用确定性的单次管线：

```text
extract drawing geometry once
    -> normalize DTOs
    -> generate semantic required dimensions
    -> deduplicate
    -> classify top/bottom/left/right
    -> assign offset layers
    -> resolve spacing/collisions
    -> create dimensions once
    -> validate
```

对于阶梯轴零件，先从工程图几何中识别轴肩工位，依次标注相邻轴段长度，再补齐一个总长尺寸。

### 异常处理与数据分层

- **禁止空 `catch` 块：** 捕获 Inventor API 异常时，至少记录操作名称、`HRESULT`、文档信息、语义实体及关键输入参数。
- **分离模型数据与工程图展示层：** 材料、计算质量、装配体 BOM、工程图 `PartsList` 与单元格覆盖属于不同层级，明细表可见单元格并不自动等同于可写源属性。

### 无注册表 `.addin` 部署

Windows 下常见的官方支持部署目录包括：

- `%ALLUSERSPROFILE%\Autodesk\Inventor Addins\`
- `%PROGRAMFILES%\Autodesk\Inventor 20xx\Bin\Addins\`（Inventor 2024+ 按版本区分的所有用户清单目录）
- `%APPDATA%\Autodesk\Inventor 20xx\Addins\`
- `%APPDATA%\Autodesk\ApplicationPlugins`

面向单用户且绑定版本的轻量部署，可将 `.addin` 清单、插件 DLL 与托管依赖放在同一个扁平产品目录中。

编写 PowerShell 部署脚本时，注意变量紧接冒号引发的插值解析陷阱：

```powershell
# 错误
throw "Deployment validation failed for Inventor $version: $required"

# 正确
throw "Deployment validation failed for Inventor ${version}: $required"
```

## 文档导航

### 技能目录

| 主题 | 涵盖内容 | 链接 |
| --- | --- | --- |
| 主路由技能 | 适用范围、强制规则、版本基线与任务路由索引 | [SKILL.md](./SKILL.md) |
| API 对象映射 | Inventor 常用对象模型与按任务分类的入口速查 | [skills/api-map/SKILL.md](./skills/api-map/SKILL.md) |
| 项目配置 | 目标框架、`x64`、互操作引用、WPF 与编译输出 | [skills/project-setup/SKILL.md](./skills/project-setup/SKILL.md) |
| Add-In 生命周期 | `ApplicationAddInServer`、`Activate`/`Deactivate` 与事件生命周期 | [skills/addin-lifecycle/SKILL.md](./skills/addin-lifecycle/SKILL.md) |
| 安装与部署 | `.addin` 清单、部署目录、加载规则与 PowerShell 脚本 | [skills/installation-deployment/SKILL.md](./skills/installation-deployment/SKILL.md) |
| COM 互操作与冲突 | `Path`/`File`/`Application`/`Point`/`Transaction` 冲突与 COM 线程模型 | [skills/com-interop-conflicts/SKILL.md](./skills/com-interop-conflicts/SKILL.md) |
| 文档、单位与事务 | 文档类型检查、数据库单位、瞬态对象、更新与保存规则 | [skills/documents-units-transactions/SKILL.md](./skills/documents-units-transactions/SKILL.md) |
| 几何、选择与引用 | `SelectSet`、`ObjectCollection`、`ReferenceKey`、`AttributeSets` 与代理对象 | [skills/geometry-selection-references/SKILL.md](./skills/geometry-selection-references/SKILL.md) |
| 草图建模 | 工作几何、二维草图、约束与轮廓 | [skills/sketch-modeling/SKILL.md](./skills/sketch-modeling/SKILL.md) |
| 零件特征 | 拉伸、旋转、扫掠、放样、打孔、圆角、倒角、抽壳与阵列 | [skills/part-features/SKILL.md](./skills/part-features/SKILL.md) |
| 钣金 | `SheetMetalComponentDefinition`、展开模式与 DXF/DWG 制造导出 | [skills/sheet-metal/SKILL.md](./skills/sheet-metal/SKILL.md) |
| 装配体 | 零部件实例、变换矩阵、代理对象、约束、遍历与 BOM 影响 | [skills/assembly/SKILL.md](./skills/assembly/SKILL.md) |
| 工程图图纸与表格 | 模板、图纸、样式、标题栏、明细表、孔表与修订表 | [skills/drawing-sheets-tables/SKILL.md](./skills/drawing-sheets-tables/SKILL.md) |
| 工程图视图 | 基础、投影、剖视、辅助与局部放大视图、比例及布局 | [skills/drawing-views/SKILL.md](./skills/drawing-views/SKILL.md) |
| 工程图尺寸 | `GeometryIntent`、`GeneralDimensions` 与确定性自动标注规划 | [skills/drawing-dimensions/SKILL.md](./skills/drawing-dimensions/SKILL.md) |
| 工程图注释 | 中心标记、中心线、孔/螺纹标注、指引线、球标与形位公差 | [skills/drawing-annotations/SKILL.md](./skills/drawing-annotations/SKILL.md) |
| BOM 与属性 | BOM、`PartsList`、iProperties、材料、质量与模型状态作用域 | [skills/bom-properties/SKILL.md](./skills/bom-properties/SKILL.md) |
| Ribbon UI 与事件 | 按钮定义、Ribbon 选项卡/面板、事件生命周期与 WPF 集成 | [skills/ribbon-events/SKILL.md](./skills/ribbon-events/SKILL.md) |
| 导入与导出 | `TranslatorAddIn`、STEP、DWG、DXF、PDF 与展开图导出 | [skills/import-export/SKILL.md](./skills/import-export/SKILL.md) |
| 性能与可靠性 | 减少 COM 往返、批量更新、DTO 架构、事务与幂等性 | [skills/performance-reliability/SKILL.md](./skills/performance-reliability/SKILL.md) |
| 故障排查 | 构建、加载、COM、建模、工程图、BOM 与部署常见故障特征 | [skills/troubleshooting/SKILL.md](./skills/troubleshooting/SKILL.md) |

### Autodesk 官方参考

- [Inventor 2026 API Help](https://help.autodesk.com/view/INVNTOR/2026/ENU/)
- [Creating an Add-In](https://help.autodesk.com/cloudhelp/2026/ENU/Inventor-API/files/CreatingAnAddIn_Overview.htm)
- [ApplicationAddInServer](https://help.autodesk.com/cloudhelp/2023/ENU/Inventor-API/files/ApplicationAddInServer.htm)
- [PartComponentDefinition](https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/PartComponentDefinition.htm)
- [DrawingViews](https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/DrawingViews.htm)
- [GeneralDimensions](https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/GeneralDimensions.htm)
- [UnitsOfMeasure](https://help.autodesk.com/cloudhelp/2023/ENU/Inventor-API/files/UnitsOfMeasure.htm)
- [TranslatorAddIn](https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/TranslatorAddIn.htm)

## 许可证

本项目基于 Apache License 2.0 协议开源，详见 [LICENSE](./LICENSE)。

Autodesk 和 Inventor 是 Autodesk, Inc. 的商标。本仓库为独立开发者资源，与 Autodesk 无关联，亦未获得其官方背书。
