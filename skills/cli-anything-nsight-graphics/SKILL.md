---
name: cli-anything-nsight-graphics
description: Capture and analyze graphics workloads with NVIDIA Nsight Graphics on Windows, including GPU Trace exports, Graphics Capture replay, and evidence-based optimization comparisons. Use for Nsight Graphics profiling and capture/replay troubleshooting; not as a substitute for Nsight Systems CPU timelines or Aftermath crash analysis.
metadata:
  revision: "2026-09-19"
  harness-version: "0.2.0"
  verified-nsight: "2026.3.1 (CLI discovery; reference documents historical GPU runs)"
---

# Nsight Graphics

Use `cli-anything-nsight-graphics` for supported orchestration and exported-table summaries. Use the installed official CLI directly when the task needs options the wrapper does not expose. Report measurements separately from hypotheses and experiments.

For Metallic work, read [Metallic workflows](references/metallic-workflows.md) for replay-trace setup, capture modes and historical failures. Do not load that reference for unrelated projects.

## Route the task

| Input / goal | Action |
| --- | --- |
| Existing Graphics Capture, metadata/compatibility | Start with `replay analyze --metadata --logs`; add screenshot if useful. |
| Existing Graphics Capture, GPU bottleneck | Check replay validity, then GPU Trace the replay. Ordinary replay iteration time is not GPU frame time. |
| Existing exported GPU Trace tables | Summarize the specific run/export directory; no new capture needed. |
| Only `.ngfx-gputrace` | Locate matching exports or a supported export workflow. Do not treat it as a Graphics Capture. |
| New frame/GPU/C++ capture | Resolve target/project, working directory, arguments, trigger and output from task context. |
| Startup/CPU/streaming tails | Use relevant application timing, logs and controlled runs; a static capture cannot establish those costs. |

Proceed with authorized analysis once the target is clear. Ask only when missing information changes the workload or outcome. Analysis alone does not require rebuilding, reinstalling tools, or launching an unrelated target.

## Discover capabilities once

```powershell
cli-anything-nsight-graphics --json doctor info
cli-anything-nsight-graphics --json doctor versions
```

Use versions when selecting installs or diagnosing discovery. Record binaries, version, compatibility mode and options; reuse until the environment changes. Prefer capabilities to hard-coded version comparisons.

- `--nsight-path` selects an observed install directory or `ngfx.exe`.
- Wrapper help defines wrapper flags. Official `ngfx.exe --help-all`, `ngfx-capture.exe --help` and `ngfx-replay.exe --help` define backend flags. Do not pass backend-only flags to the wrapper.
- Wrapper frame triggers `--wait-frames` / `--wait-seconds` map to backend options such as `--frame-index` / `--elapsed-time`; they remain valid.
- Resolve architecture/metric set for the installed device/tool. Numeric metric-set IDs are not portable across configurations.
- Doctor proves discovery, not working GPU injection, counters access or capture compatibility.
- For a broken entrypoint, inspect `Get-Command cli-anything-nsight-graphics` and `python -m pip show cli-anything-nsight-graphics` first. Resolve the actual checkout and Python environment before an authorized editable reinstall; do not run `pip install -e .` blindly in the task directory.

## PowerShell commands

Set variables from actual task inputs; use a fresh output directory for new artifacts. Root options precede subcommands. Use single lines or argument arrays, not CMD `^` continuations.

```powershell
cli-anything-nsight-graphics --json --output-dir "$out" frame capture --exe "$exe" --wait-frames 10
cli-anything-nsight-graphics --json --project "$project" --output-dir "$out" frame capture --wait-seconds 5
cli-anything-nsight-graphics --json --output-dir "$out" gpu-trace capture --exe "$exe" --start-after-ms 1000 --limit-to-frames 1 --auto-export --summarize
cli-anything-nsight-graphics --json gpu-trace summarize --input-dir "$exports" --summary-limit 10
cli-anything-nsight-graphics --json replay analyze --capture-file "$capture" --output-dir "$out" --metadata --logs
cli-anything-nsight-graphics --json --output-dir "$out" cpp capture --exe "$exe" --wait-seconds 5
cli-anything-nsight-graphics --json launch detached --activity 'Graphics Capture' --exe "$exe"
cli-anything-nsight-graphics --json launch attach --activity 'Graphics Capture' --pid $targetPid
```

For launch/capture, use `--dir` for working directory, repeated `--arg` for arguments and `--env` for environment entries; check the selected command's help. Capture needs `--exe` or root `--project`; split-only capture requires `--exe`. Attach is a connection action, not proof of capture completion.

With no analysis switches, `replay analyze` exports metadata/logs and runs a performance replay. Use explicit switches for lightweight analysis. `--screenshot` exports the embedded image, not a newly rendered replay image. The wrapper provides metadata/function/object summaries, not arbitrary shader/pipeline/texture inspection.

## Validity and execution

- Record build/commit, capture path, scene/camera, render extent, quality, GPU/driver/Nsight, shader optimization, validation, clocks, queues, metric set and warm-up. Mark unknown fields; do not infer scene extent from swapchain screenshots.
- Distinguish launch, capture, replay, export and parsing success. Check exit status and fresh nonempty artifacts with required tables. Existing directories/files do not prove this run succeeded; replay uses fixed output filenames.
- Use bounded triggers. Offscreen targets may lack Present: use supported submit/time or SDK boundaries. Replay profiling should use replay begin/end when supported.
- Profile serially on the measured GPU. Bound execution and inspect logs after timeout. Clean up only processes attributable to this run by PID/path/start time.
- Reject samples with relevant buffer overflow, unavailable counters, missing metrics, incomplete export or competing failed replay. Never summarize stale tables after failed capture. Retry after an identified correction; unchanged failures call for a limitation and an alternative evidence path.
- Follow local session rules: use `cua-child` for native UI when required; never silently fall back to the main desktop. Shell launch is not automatically Child Session launch. Prefer verified headless/hidden CLI when UI is unnecessary, and verify profiling in the selected session.

## Evidence and optimization

- Nsight `.xls` files can be TSV text; inspect contents. Common tables: `D3DPERF_EVENTS.xls`, `GPUTRACE_REGIMES.xls`, `REPRO_INFO.xls`. Preserve run/table/row, units and metric names behind findings.
- Inspect warnings and inventories before rankings. Empty tables mean missing detail, not zero work. Negative/sentinel GPU times are unavailable.
- Verify important summary aliases against full metric names and units in raw exports. Wrapper substring matching can confuse similarly named counters; missing aliases may still have usable architecture-specific raw metrics. Do not silently substitute counters with different semantics.
- Preserve marker hierarchy, occurrence order, queues and early/late phase. Do not sum nested or asynchronous overlapping ranges into frame time; inspect critical path and join.
- Stall-sample fractions are not frame-time fractions or recoverable speedup. Utilization alone does not prove bottlenecks; low DRAM throughput does not exclude latency/cache issues. Barrier/submit counts alone do not prove bubbles.
- Wrapper 30/60 FPS budgets and throughput thresholds are heuristics. Use the task's actual budget; attach evidence and uncertainty to claims.
- Correlate ranges with current entrypoints, dispatch dimensions and synchronization. Broad markers cannot isolate inner kernels. Old captures describe old code: inspect subsequent changes before repeating old recommendations.
- Fixed-view A/B should preserve camera/cut/residency and quality. Validate startup, roaming and streaming independently. Report local range and whole-workload results separately. One valid capture is not a mean or P95.
- End with the key finding, values and provenance, confidence/data gaps, next useful experiment and artifact paths. Stop after relevant checks pass; do not automatically expand to long benchmarks or repeat successful checks.

## Compatibility

The `.ngfx-gputrace` “Invalid file header” observation was from 2026.1.0, not a current universal result. Official replay input is Graphics Capture. NVIDIA documentation mixes `.ngfx-bincap` prose with `.ngfx-capture` examples; the wrapper accepts `.ngfx-capture` and diagnostic `.ngfx-gputrace` inputs. Do not rename inputs or promise other formats without sample-based validation.

- [Capture/replay CLI](https://docs.nvidia.com/nsight-graphics/UserGuide/graphics-capture-cli.html)
- [GPU Trace CLI](https://docs.nvidia.com/nsight-graphics/UserGuide/gpu-trace-overview.html)
- [Releases](https://developer.nvidia.com/nsight-graphics/get-started)
