---
name: metallic-loading-performance
description: Diagnose Metallic scene loading, streaming latency, upload concurrency and VRAM regressions using phase measurements and full-renderer evidence. Use for startup or scene-load performance problems, rather than fixed-frame shader tuning.
---

# Metallic 加载与流送性能

先定位 checkout、读取当前 AGENTS.md 和相关加载代码。记录场景、构建、缓存、纹理/几何规模、渲染配置和用户原始日志；保留首个真实错误及其之前的时间线。

## 诊断

1. 划分文件读取、解析、几何转换、图像 decode/mip、shader 编译、上传和首个可用帧。检查阶段是否重叠；阶段耗时不能直接相加成总加载时间。
2. 对比相同场景和配置下的冷/热缓存；先测具体慢路径，再决定改动。固定帧 Nsight capture 只描述该帧，不证明启动或流送尾延迟。
3. 读实际任务依赖、ready 队列与预算准入。已有并发时检查 FIFO 队头阻塞和 ready-work admission，不重复增加同类并行。
4. 上传优化同时检查在途 batch 数、staging、目标资源、延迟释放及渲染器其他 pass 的驻留峰值。局部上传更快不证明完整渲染可承受。

## 修改与验收

- 每个候选对应明确阶段及可检验假设。保留同步和所有权契约；不要仅按吞吐量扩大 batch 或并发。
- 同时报告目标阶段、首帧/端到端耗时及峰值资源。流送另查 request-to-drawable、驱逐/重载和质量收敛。
- 回归检查包含触发问题的完整渲染路径与相关功能，例如原场景的 DLSS；隔离测试不替代完整负载。
- 若候选引入实际 OOM 或渲染回归，保留日志并撤销本次候选，核对恢复后的原路径。不能用降低质量掩盖原请求的回归。

## 历史案例边界

2026-09 的 SuperSponza 工作曾把上传扩大到 192 MiB、三个在途 batch，随后完整渲染出现 DLSS `eWarnOutOfVRAM` 与 RenderGraph OutOfMemory，回退到 64 MiB/逐 batch 同步。这是该负载的历史失败案例，不是所有硬件的固定上限。应用前检查当前代码，重新测量；不要自动重试已失败配置。

交付原始日志路径、阶段定义、可比条件、峰值/耗时、正确性与回滚结果；没有实际运行的部分明确列出。
