# 来源与验证状态

整理日期：2026-09-28。当前仓库仅收录指南和用法，不迁移运行产物，不修改 Metallic / VividRP 或全局 skill。

## 完整收录

- `metallic-shader-optimization`：来自本机 `C:/Users/11252/.codex/skills/metallic-shader-optimization`，保留 SKILL、references 和 agents 元数据。执行实现仍属于 Metallic 的 Tools/Perf。
- `cli-anything-nsight-graphics`：来自本机同名 skill 目录，保留 SKILL 和 Metallic workflows 引用；harness 未收录。

这些是收录日期的快照。更新时比较来源目录与仓库目录，不自动覆盖任一侧的后续编辑。文中的旧版本、绝对路径和测量值是历史定位信息。

## 新提炼

| Skill | 依据 | 验证边界 |
| --- | --- | --- |
| metallic-loading-performance | SuperSponza 加载/上传回归工作记录；当前 Metallic AGENTS.md | 历史 192 MiB 并发上传回归；没有本次性能复测 |
| metallic-shader-warmup | 可选 shader warmup 工作记录；当前 cmake/ShaderWarmup.cmake | 历史最终完整运行有两个失败，不能认为当前已经解决 |
| vividrp-allocation-diagnostics | VT 分配及 PlayMode JIT 工作记录；当前 VividRP AGENTS.md | 局部/stub 验证和真实 Unity 验证分开；预热是待验证方案 |
| vividrp-rendergraph-editor | 节点重复工作记录；当前节点/graph 声明和 package.json | 历史 Roslyn 通过，Unity 菜单待实测 |

本次读取的来源 checkout HEAD：Metallic `aafa778a897dc7915819546e1cd1f8d27f253749`，VividRP `6fbde38d85e3be42c5e99a30d3cec426fb87dd42`。HEAD 只是定位线索；读取内容也可能包含来源工作区未提交修改，不表示是这两个提交的纯净快照。

## 核验范围

本次检查 skill frontmatter、相对文档链接、收录副本一致性及文档差异。未启动 GPU 实验、完整 shader warmup、Unity Editor/tests 或性能复测。历史工作记录提炼的是操作方法，不是当前项目健康状态或性能结论。

## PIX M3 补充收录（2026-09-28）

`vivid-pix-workflow` 适配自 VividRP 项目内 `.agents/skills/vivid-pix-workflow`，核对了最新 M3 runner、PIX 接口指南、native 指南与实机验收报告。保留名称和 UI 元数据，修正跨仓库路径定位，补充旧 capture 的分析与恢复。执行实现继续由 VividRP 维护。

本次确认 package 自身为嵌套 Git 仓库，HEAD 为 `917c9c2399d6cd6a35bc3dd6b47ada79045de220`；前文 VividRP `6fbde38...` 是宿主工程 HEAD，不能作为 package 提交身份。来源文件哈希与验证范围见 [PIX 来源映射](../skills/vivid-pix-workflow/references/source-map.md)。本次执行 runner `--help` 并校验文档，未重跑实机验收。
