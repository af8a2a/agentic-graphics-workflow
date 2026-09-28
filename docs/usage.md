# 用法与选择

## 输入准备

提供目标 checkout、当前问题和现有证据即可开始。性能问题尽量附带场景、相机、分辨率、构建配置、缓存状态及原始日志/捕获路径；未知项明确写为未知。路径示例均需替换成本机实际路径。

| 任务示例 | 使用 skill | 所需环境 / 预期交付 |
| --- | --- | --- |
| “复核这个 experiment 包，解释为什么 reject” | metallic-shader-optimization | Metallic 当前 verifier；证据有效性、判定与恢复状态；无需重跑 GPU |
| “测试这个 WorkControl shader 候选” | metallic-shader-optimization | Metallic runner、Python 依赖、构建与真实 GPU；A/B、正确性、确认阶段与 baseline 恢复 |
| “分析现有 GPU Trace 导出” | cli-anything-nsight-graphics | 导出表及解析工具；原始列名/单位、阶段归因与数据缺口 |
| “SuperSponza 加载变慢，并出现 OutOfMemory” | metallic-loading-performance | 应用时间线与完整渲染场景；阶段耗时、资源峰值、因果证据及回归验证 |
| “增加新 pass 的离线预编译请求” | metallic-shader-warmup | MSVC/CMake/Slang 与当前 runtime 请求；缓存身份核对、完整预热结果及 cache-miss 路径检查 |
| “CreateIOBatch 每帧 malloc，按这个调用栈修复” | vividrp-allocation-diagnostics | VividRP、实际 Unity 版本、Profiler；所有权分析、局部测量与 Unity 验证状态 |
| “RenderGraph 菜单节点重复或丢失” | vividrp-rendergraph-editor | VividRP 与实际 GraphToolkit 实现；注册来源、唯一性/覆盖测试及菜单验证状态 |
| “用 PIX 捕获当前相机的 HZB pass，检查 Dispatch 和资源” | vivid-pix-workflow | 已注入 D3D12 Editor、Unity CLI、PIX/native 工具；diagnosis、retained capture 和验证证据 |
| “分析已有 .wpix 中这个事件，不重新捕获” | vivid-pix-workflow | 匹配的身份 JSON、外部 analyzer、PIX/GPU；有界定向查询与 hash 校验 |

## 组合工作流

- GPU 候选先用 Metallic runner 建立可信 case；需要更宽阶段归因时再调用 Nsight skill。诊断采集与普通 A/B 计时分开。
- 加载/流送问题先分离 CPU、IO、shader 编译与 GPU 驻留；确认是 cache miss 后再使用 warmup skill。固定帧捕获不能测量整个启动过程。
- VividRP 节点修复涉及生成器时运行独立 .NET 测试；涉及稳定热路径再增加分配诊断。不要因为两个 skill 都存在就执行全部流程。

## 依赖与安装边界

收录的 Nsight skill 依赖外部 `cli-anything-nsight-graphics` harness 或其说明允许的官方 CLI；本仓库不包含该 harness。Metallic skill 依赖目标 checkout 的 `Tools/Perf`，NvPerf live capture 另需匹配的 SDK 和 GPU 环境。VividRP skill 依赖包含该 package 的 Unity 工程及对应版本工具。

可以完整复制选定 skill 目录到 agent 配置的搜索目录；已有同名目录时先比较并合并，避免覆盖本机改动。未安装时直接指定 SKILL.md 路径也可阅读执行。这里不自动安装依赖，也不替换用户原有配置。

## 交付方式

报告所处理的 workload/版本、证据路径、改动、通过/失败/未运行的验证。注明证据范围：静态编译、替身测试、单路径采样、真实 GPU、Unity Editor 或 Player。缺失运行环境时留下具体的最小复测步骤。

## PIX 调用示例

> 使用 $vivid-pix-workflow，目标 package 为 E:/VividRP_Reborn/Packages/VividRP，Unity 工程为实际使用该包的工程。先 preflight 发现相机和精确 marker，再针对指定 pass 生成带资源证据的诊断。

> 阅读 skills/vivid-pix-workflow/SKILL.md，分析给定 diagnosis.json 对应的 retained .wpix，仅追加该 capture 的 accessed_resources 查询，不重新捕获。

先从 [skill 入口](../skills/vivid-pix-workflow/SKILL.md) 定位目标工程。`diagnose` 会新建 capture；已有 capture 的查询使用独立 analyzer。安装此 skill 时避免与 VividRP 项目内同名 skill 同时成为可发现副本。
