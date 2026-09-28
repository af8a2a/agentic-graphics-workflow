# VT 历史定位线索

来源为 2026-09-19/20 的 VividRP 工作记录，非当前实测；使用前搜索当前符号和实现。

- `VTPageTableUpdater`：dirty mask 对唯一页索引去重，脏列表可按 totalPageCount 预分配。测试需覆盖重复标记、全量 burst、清空复用。
- `VTStreamIO` 的 `RentBatch` / `Prepare` / `EnsureCapacity` / `ReleaseBuffers`：只池化 wrapper 仍可能每批 native malloc。按实际 chunk 元数据准备 buffer，检查 best-fit 和并发容量，提交 `CommandCount = Count`。
- `VTStreamChunkManager` retained batches 可能持有 NativeStoredData view；IO 完成不自动意味着所有 view 均可释放。
- `VividVirtualTextureAssetProducer` 的 built chunks 曾提供最大 StoredByteSize 线索；容量来自数据契约，而非任意常数。
- `AsyncGPUReadback.RequestIntoNativeArray` 的历史 28 B 分配来源未确认，不能声称已消除。
- `VTFeedbackNativeAggregator` 初始帧预热方案当时只有调研，没有实施或收益实测。

当时独立编译和 instrumented native stub 验证通过，Unity Test Framework 与真实 AsyncReadManager/Profiler 复测仍未完成。当前任务必须重新确定验证状态。
