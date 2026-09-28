---
name: vividrp-rendergraph-editor
description: Diagnose VividRP RenderGraph editor node discovery, duplicate or missing menu entries and source-generator integration. Use for GraphToolkit registration and generated-node regressions.
---

# VividRP RenderGraph 编辑器

先读 package AGENTS.md、package.json、宿主 Unity 版本与锁定依赖。定位受影响 graph、node 基类、程序集和生成注册表；不要先在菜单最终显示层去重。

## 追踪注册来源

```powershell
rg "UseWithGraph|DisableAutoInclusionOfNodesFromGraphAssembly|RenderGraphNodeData" Editor Tests
rg "GeneratedRenderPassNodeRegistry|RenderPassNodeSourceGenerator|GetRegisteredPassType" Editor Runtime Tests SourceGenerators~
```

检查当前 GraphToolkit 的节点发现实现：特性继承、派生类遍历、同程序集自动包含、显式注册和生成列表可能叠加。核对可见性与 graph 归属，确认重复从何处进入。

历史 GraphToolkit 0.5.0-exp.1 会通过继承的 UseWithGraph 再遍历派生类型，导致重复。该版本 VividRP 曾通过移除基类特性、恢复同程序集自动包含解决；这不是让所有版本删除特性的通用规则，先查当前代码是否已经修复。

## 完整性与生成器

- 测试同时覆盖主 graph 和 subsystem graph 的唯一性、预期节点覆盖和测试程序集隔离；只断言数量减少会漏掉节点丢失。
- 优先检查 `Tests/Editor/RenderGraph/RenderGraphNodeMenuVisibilityTests.cs` 与实际调用方。
- 生成器改动按当前 README 执行独立测试；需要部署时使用项目目标，不编辑生成 DLL：

```powershell
dotnet test SourceGenerators~/VividRP.RenderPassNodeGenerator.Tests/VividRP.RenderPassNodeGenerator.Tests.csproj
dotnet build SourceGenerators~/VividRP.RenderPassNodeGenerator/VividRP.RenderPassNodeGenerator.csproj -c Release -t:DeployToUnity
```

不要为普通节点显示问题无条件部署生成器。资源或包路径改动需同时审计 `Packages/VividRP` 与 `Packages/com.af8a2a.vividrp` 常量以及 `Editor/PipelineResource/PipelineResourceUpdater.cs`，通过同步管线更新资源。

## 验证

遵守当前 AGENTS.md 的 Unity 会话限制：Editor 活跃时做针对性 Roslyn/程序集检查，留给用户运行相关 Unity tests 和重新打开菜单；不通过 UI 自动化主动启动测试。独立生成器测试、Roslyn 编译和 Unity 菜单验证分别报告。

历史修复只完成了 Roslyn 编译，菜单行为当时未实测。交付当前注册来源、修复方式、唯一性/覆盖范围和本次实际验证状态。
