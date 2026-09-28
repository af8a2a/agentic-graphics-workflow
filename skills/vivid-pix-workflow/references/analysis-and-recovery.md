# 已有 capture 的分析与恢复

只读既有证据时，不调用 `diagnose`：它总会创建新 capture。先读匹配的 `diagnosis.json`、`capture-ready.json`、`capture-retention.json` 和原始分析 JSON。缺少 session、expected pass 或 boundary mode 时先找回原验证证据，不编造身份。

## 独立 analyzer

从确认的 package 根目录执行。`$pix`、`$capture`、`$session`、`$marker`、`$mode`、`$hash` 均来自该份 capture 的证据；`$capture` 优先使用 retained copy。选择新结果文件，不覆盖旧证据。使用当前 native README 核对命令，示例为 M3 收录时接口：

```powershell
$analyzer = Join-Path $package 'Tools~/PIX/bin/vivid-pix-analyzer.exe'
$result = Join-Path $evidenceDir ('events-' + [guid]::NewGuid().ToString('N') + '.json')
$request = [guid]::NewGuid().ToString('N')
& $analyzer events $pix $capture $session $marker $result $mode --marker $marker --count 100 --request_id $request --capture_hash $hash --timeout_seconds 120
$analysisExit = $LASTEXITCODE
```

`$evidenceDir` 为已创建的证据目录。读取结果并确认 exit 0、`success=true`、`code=ok`、schema 3，以及 request/session/path/pass/mode/hash 匹配。`nextOffset` 用于翻页；偏移量不是 event ID。从本次结果选择 `gpuWork=true` 的 `queueId` 与 `eventIndex`，保存为 `$queue` 和 `$eventIndex`：

```powershell
$result = Join-Path $evidenceDir ('pipeline-' + [guid]::NewGuid().ToString('N') + '.json')
$request = [guid]::NewGuid().ToString('N')
& $analyzer pipeline $pix $capture $session $marker $result $mode --queue $queue --event $eventIndex --capture_hash $hash --request_id $request --timeout_seconds 120
$analysisExit = $LASTEXITCODE
```

同样校验 queue/event 身份。每个查询会重新打开并验证内容与 hash；查询期间不要改 capture。无需运行 Unity Editor，但 replay 仍需兼容的 PIX/runtime/GPU。不是任意 `.wpix` 都符合此接口：缺少 VividRP session marker/content gate 的捕获不能冒充通过验证。

- `resources` 表示静态绑定；`accessed_resources` 成功后才有实际访问证据。
- `counters` 先查有界目录，再用返回 ID 的 `--counter` 查询该事件。
- `occupancy` 目录给出 type/stage，再用 `--type`、`--stage` 选择；runner 的单独 `--occupancy` 选项本身不等于已取得 occupancy points。
- `drpix` 先查目录，再用返回 GUID 的 `--experiment` 查询；保留 SDK 原始标签，不推测单位。
- `event` API XML 有 16 KiB 上限；pipeline、资源列表与诊断摘要均可能截断。查看 total/nextOffset/截断标志，不能声称已穷举。

## Editor 内的异步查询

需要继续使用尚未释放的 live session 时，按目标 `PIX.md` 使用 `agentic_gpu_debugger` 的 events/pipeline 等动作。返回 `analysisId` 后轮询 `analysis_status`，同时提供 `session_id` 和 `analysis_id`；capture `status` 不能代替分析状态。

M3 runner 在外部分析前已释放 capture owner，所以不能把最终 diagnosis 中的历史 session token 当作当前 Editor 所有权。改用上面的独立 analyzer。

## 失败和清理

- `cleanupPending=true`：用本次 token 查询并再次 release，直到 idle；owner mismatch 时停止，不释放别人的会话。
- 启动超时：检查 launch.json、pixtool/Editor 日志和对应进程。不要删 UnityLockfile 或再启动一个实例；原 Editor 可能仍在启动。
- analyzer exit 2 为失败，4 为 watchdog timeout；缺 JSON、非零退出或身份不符都不能凭已有文件判成功。
- cancel/release 只处理本次 analyzer。Domain Reload/Editor exit 后无自动重附着；保留的外部 helper 可在有界时间内独立完成 JSON。
- capture 与 analysis 使用不同状态和 token；失败 capture、partial output 和 Console 错误全部保留。只在找到具体修正后另建输出重试。
