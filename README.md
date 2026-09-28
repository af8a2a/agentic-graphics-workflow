# Agentic Graphics

收录 Metallic / VividRP 开发中实际使用的 agent skill，整理调用方式，并把重复工作流提炼为可复用的操作指南。

## Skill 索引

| Skill | 何时使用 | 来源 |
| --- | --- | --- |
| [metallic-shader-optimization](skills/metallic-shader-optimization/SKILL.md) | 可信工作负载、shader A/B、NvPerf、编译资源与中断恢复 | 已有 skill 完整收录 |
| [cli-anything-nsight-graphics](skills/cli-anything-nsight-graphics/SKILL.md) | Nsight 捕获、回放、GPU Trace 与原始指标分析 | 已有 skill 完整收录 |
| [metallic-loading-performance](skills/metallic-loading-performance/SKILL.md) | 场景加载、流送尾延迟、上传并发与 VRAM 回归 | 工作流提炼 |
| [metallic-shader-warmup](skills/metallic-shader-warmup/SKILL.md) | 可选 SPIR-V 缓存预热、请求身份与完整目标验证 | 工作流提炼 |
| [vividrp-allocation-diagnostics](skills/vividrp-allocation-diagnostics/SKILL.md) | GC / native 分配、VT batch 生命周期、初始 JIT/Burst 开销 | 工作流提炼 |
| [vividrp-rendergraph-editor](skills/vividrp-rendergraph-editor/SKILL.md) | 节点重复/缺失、GraphToolkit 注册与代码生成验证 | 工作流提炼 |

## 使用

每个 `skills/<name>/` 是独立 skill 包，保留其中的 references 和 agents 文件。可以在任务中直接指定文件：

> 阅读 E:/agentic-graphics/skills/vividrp-allocation-diagnostics/SKILL.md，分析这份 VT Profiler 调用栈；先确认分配来源，再修复并报告实际验证范围。

将所需目录安装到当前 agent 的 skill 搜索目录后，可按名称调用，例如 `$metallic-shader-optimization`。本仓库只维护源文件，本次整理没有覆盖本机已安装的 skill。不要同时维护同名的多份可发现副本。

更多任务示例、依赖和组合方式见 [用法](docs/usage.md)，收录来源与验证状态见 [来源](docs/sources.md)。

## 维护

新增 skill 应对应可重复的任务与明确触发条件。只出现一次的修复细节优先作为带日期的案例，不直接写成通用规则。工具参数以目标项目当前源码和 help 为准。大型 capture、SDK、模型、缓存和项目 runner 留在来源项目；本仓库不复制这些产物。

修改后检查 YAML frontmatter、相对链接和 diff；脚本发生变化时运行对应脚本。文档校验不能代替 GPU 或 Unity 运行验证。
