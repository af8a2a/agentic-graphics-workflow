---
name: metallic-shader-warmup
description: Maintain and validate Metallic optional Slang SPIR-V cache warmup requests, cache identity and the complete manual CMake target. Use when adding precompile coverage or diagnosing warmup failures.
---

# Metallic 可选 shader 预热

定位当前 checkout，读取 AGENTS.md、`cmake/ShaderWarmup.cmake`、`Tools/ShaderWarmup.cpp`、`Tools/ShaderWarmupRequests.h` 及 runtime 编译请求构建代码。复用匹配的 CMake build tree，在 MSVC x64 环境下构建；不要为预热改动默认运行配置。

## 请求与缓存契约

- 预热保持手动可选，不加入默认构建或 editor/sample 的前置依赖。运行时 cache miss 编译路径继续可用。
- 请求与所属 runtime pass 一致：module、entry、profile/capabilities、搜索路径、宏、debug mode、descriptor mode 及其他 key 输入都需核对。同名 entry 不等于同一请求。
- 复用当前 SlangCompiler 与缓存实现，不另写一套 key 或编译逻辑。默认缓存位置的历史线索是 `.cache/shaders/spirv`，执行前检查实际配置。
- 枚举失败先检查 owning module/import。历史 `StreamClusterClassify` 由 `Features/GPUDriven/GPUDrivenStreamAsset` 拥有，不能仅按文件名编译为独立根模块。

## 验证

从当前工具 help 确认 `--list`、`--filter`、`--cache-dir`、`--debug-mode` 等能力。用独立缓存目录做目标请求验证，避免破坏用户已有缓存；检查产物能由当前缓存 reader 读取。覆盖 cache miss、cache hit 和相关 key 变化，确认 runtime 请求能消费相同缓存。

目标请求通过后，若声称完整预热成功，必须完成整个目标并确认零失败。例如在确认 build tree 后：

```powershell
cmake --build build-release --target MetallicShaderWarmup --config Release
```

保留每个失败请求、编译诊断、请求总数、hit/compile/failure 计数及退出码。不要用局部测试覆盖最终失败结果。

历史 2026-09-25 记录为 177 请求、144 existing cache hits、2 failures；这不是当前目标状态，也不是已完成验收。请求数量和失败是否解决需以本次完整输出为准。

交付请求与 runtime 的对应关系、验证命令、缓存复用结果和未验证范围；保持本任务外的构建行为不变。
