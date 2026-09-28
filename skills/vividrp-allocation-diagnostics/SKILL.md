---
name: vividrp-allocation-diagnostics
description: Diagnose VividRP managed and native rendering allocations, VT batch lifetimes and initial PlayMode JIT or Burst costs from profiler call stacks. Use for allocation regressions and evidence-based warmup investigations.
---

# VividRP 分配诊断

读取当前 package AGENTS.md、package.json、宿主工程的 ProjectVersion 和相关设置。以用户给出的 Profiler 栈和首个可操作的源码分配点为入口，区分 GC.Alloc、native malloc/free、JIT/Burst 和资源初始化。历史版本和阈值只作定位线索。

## 稳定路径

- 先明确热路径、预热边界、线程、对象/缓冲区所有者和生命周期。按真实输入上界预分配，容量上界需由去重/索引契约证明。
- 托管测量先预热，将输入、委托、断言消息和采样自身移到窗口外；`GC.GetAllocatedBytesForCurrentThread()` 只证明当前线程，不覆盖 native allocation 或其他线程。
- NativeArray 复用必须保留在途 IO、Job、readback 和外部 view 的生命周期。只重置逻辑长度；实际提交数量与有效 Count 一致，不能提交整个容量。
- 复用测试覆盖首次 burst、重复复用、容量增长/拆分、不同 batch 隔离与最终释放。stub 的零分配不能证明真实 Unity backend 零分配。
- 无法从当前 Unity 托管源码解释的内部小额分配先保留安全实现，通过 Profiler 复测，不靠删掉同步或所有权保护来消除指标。

## 初始帧

分开测冷启动、进入 PlayMode、预热后稳定帧及目标 Player。确认 Domain Reload、Burst 和 scripting backend；同步编译可能移动首次等待而非消除成本。改重载设置前审查静态状态和事件重置。

预热应覆盖实际非空分支并等待工作完成。历史 VT 聚合实现以 64 为分界，1/65 条请求用于覆盖两路；先检查当前实现，不能把此阈值固化为通用策略。只有实测后才能报告预热收益。

## 工具与验证边界

遵守目标项目的当前 Editor 会话约束。当前 VividRP 指引要求 Editor 运行时不启动 Unity Test Framework，也不通过 UI 自动化绕过；用实际 Unity references/response files 做局部 C# 编译，shader 用匹配 defines/entry 的编译检查。Editor 关闭时才按项目支持方式运行 Unity tests。

不要手工生成 `.meta`、编辑同步资产或生成 DLL。需要实时 Editor 检查时使用可用的 Unity 工具，不能假定其已安装。

交付分配来源与因果链、所有权和释放规则、局部测量方法、真实 Profiler 结果或最小手动复测步骤。历史案例见 [VT 线索](references/vt-cases.md)。
