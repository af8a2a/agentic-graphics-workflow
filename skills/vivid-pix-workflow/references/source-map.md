# 来源映射与验收边界

收录日期：2026-09-28。适配自 VividRP 项目内的 `.agents/skills/vivid-pix-workflow`，保留 skill 名称与 UI 元数据；改为显式发现目标 package，增加 retained capture、独立 analyzer 和恢复说明。不是来源文件的逐字副本。

本机来源：`E:/VividRP_Reborn/Packages/VividRP`。它是独立嵌套 Git 仓库，HEAD 为 `917c9c2399d6cd6a35bc3dd6b47ada79045de220`。宿主仓库 HEAD 不能标识这个 package。所读路径的 Git status 未报告变更；下表哈希精确标识本次读取的工作区文件。

| 目标 package 内路径 | 何时读取 |
| --- | --- |
| `Tools~/PIX/README.md`、`vivid_pix_agent.py --help` | M3 launch/diagnose 命令、界限和输出 |
| `Editor/AgenticDebugger/PIX.md` | 相机 capture、queue scope、异步查询与字段覆盖 |
| `PluginSource~/PixCaptureNative~/README.md` | 可选工具构建部署、独立 analyzer 和测试 |
| `Documentation~/AgenticPixM3Acceptance.md` | 历史真实场景验收和剩余限制 |

依赖为 Python 3.10+ 标准库、Unity CLI、匹配工程的 Editor、Windows D3D12、PIX 与匹配 native plugin/analyzer。需要构建时，来源指南使用 VS C++、CMake、.NET 10；构建脚本会下载并校验依赖，部署会更新目标 package 的 ignored 输出。仅分析旧 capture 不需要部署或重新打开 Editor。本 skill 不携带 DLL、SDK、runner 或 capture。

## 最新报告的证据范围

来源 M3 报告记载 2026-09-28 在 `E:/VividRPSample`、Unity 6000.7.0b1、PIX 2606.18-preview 下完成 SampleScene 正常相机捕获、精确 HZB marker 的非零 Dispatch、pipeline/资源/timing/counter 和 accessed-resource 查询；Console 区间无新增错误且游标完整，错误 pass 被拒绝并释放会话。

该报告中的 Async Compute 选项开启后，目标 Dispatch 仍在 graphics queue，不能称为真实 async-only 场景验收。四个逻辑 scope 不等于四个物理队列。occupancy E_NOTIMPL、counter format 7 未解码、DXBC shader hash 缺失均是当时观测，不是所有版本的永久结论。

raster/RT 全覆盖、真实 async-only 相机负载、捕获中取消/Domain Reload、失败或跳过相机矩阵仍不在报告完成范围。启动时已有 shader UAV-limit 错误也未由该工作解决。

本次只核对本地 skill、文档、runner 源码和 `--help`；未重跑 capture/replay，未启动 Unity 或 Unity Test Framework。报告引用的真实 capture 未在本次读取为新验证证据。

## 来源文件 SHA-256

| 文件 | SHA-256 |
| --- | --- |
| `.agents/skills/vivid-pix-workflow/SKILL.md` | `a16d87477103f7227198155068a4cba1854ba8fd36e5cb2f989f6657f8b7f305` |
| `.agents/skills/vivid-pix-workflow/agents/openai.yaml` | `30a37a861baf2d6c5b6a277138689fee41aacdd8a25c852a194d6d31ca55024f` |
| `Tools~/PIX/vivid_pix_agent.py` | `b6364864a0386c22fb894deb736ab559e8068bdca6801a15cae14fdb724cb2de` |
| `Tools~/PIX/README.md` | `4d5b4804ea13dcba0c2545bbf871e0c54547615d99a8404ca9a4051fd1b26902` |
| `Editor/AgenticDebugger/PIX.md` | `1cc45fdf072c2f6be0c48ecfdc29ab5c3edc34fb9da2faffd62ea2b4d09c96e9` |
| `PluginSource~/PixCaptureNative~/README.md` | `14bf30959044c7ddda8521bbc225cb56dd569608308696f3d18e934b8f55b7b1` |
| `Documentation~/AgenticPixM3Acceptance.md` | `6485fe3d2327ed29ee67d5d3c1731a7f53e378061e9428982b18abbd34d0e3b5` |
