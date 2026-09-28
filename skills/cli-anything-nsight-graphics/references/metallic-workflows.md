# Metallic profiling workflows

Derived from local evidence reviewed on 2026-09-19. Paths are source pointers, not instructions to repeat historical benchmarks. Read current project AGENTS.md and implementation before applying a workaround or recommendation. Historical GPU runs used Nsight 2026.3.1, RTX 5070 Ti / GB203 and driver 616.64; discover the current environment.

## Source map and scope

Under `E:/metallic/Documentation/`:

- `MiniZorahNsightAnalysis.md` and its `Result.json`: fixed capture profile, rejected samples and source attribution.
- `MiniZorahParallelCandidates.md`: later count/prefix/scatter implementation and A/B validation. Do not recommend implementing it again from the earlier profile.
- `MiniZorahRoaming.md`, `MiniZorahRoamingOptimization.md`, `MiniZorahStartupStalls.md`: dynamic workload and alternative timing evidence where Nsight capture failed.
- `NsightKhrOpacityMicromapInvestigation.md`, `NsightCaptureReplayInvestigation.md`: distinct capture/replay failures and targeted workarounds.
- `ProjectArchitecture.md`, section 12.2: capture versus shader-debug compilation.

GPUDriven kernel questions benefit from replay GPU Trace plus source/dispatch inspection. Streaming/frontier tails need RHI phase timestamps and request/residency reports; startup stalls need time-aligned application/CPU/GPU evidence. Checkpoint intervals include adjacent preparation/barriers. Failed capture provides no kernel attribution.

## Project capture modes

Inspect `Source/Main.cpp`, `Source/Runtime/Render/Profiling/NsightGraphicsCapture.cpp` and the selected build config; do not assume a build directory or default injection mode.

- `--nsight-capture` enables application capture support and optimized Slang source/line info (`-g1`). Use optimized shaders for performance.
- `--nsight-shader-debug` enables unoptimized full shader debug info for the appropriate Shader Debugger activity. Do not use its timings as representative performance.
- Editor profiler can export the current view. Use Child Session UI only when needed and available.
- `tests/rhi/main.cpp` supports `--rhi-nsight-export`. Historical `*gpu_driven_sponza_realtime_pipeline` exports a warmed-up offscreen frame using NGFX boundaries and waits for completion before resource release. Discover current executable/flags; this Sponza reproduction does not replace a requested MiniZorah workload.
- Editor uses Present boundaries; offscreen tests may have no UI/swapchain/Present. Do not wait on a Present trigger there.

## GPU Trace of an existing Graphics Capture

Export metadata/logs first and the embedded screenshot if useful. Visual validity requires an actual replay-output/reference comparison when available; embedded screenshot export alone does not validate replay.

Historical MiniZorah `iteration_times.csv` reported 10.521 ms CPU submission plus waiting with `msGpuTime=-1`. The accepted GPU Trace measured 8.20451 ms. They are different quantities. The 2400 x 1350 swapchain included editor UI; scene viewport extent was unknown.

The wrapper lacks replay-boundary triggers and metric-set names. Use official CLI after checking each option in installed help. This PowerShell template adapts historical setup; it is not a new measurement. Set `$capture` and fresh `$out`; resolve `$architecture` / `$metricSet` for current hardware. Historical values were `Blackwell GB20x` / `Top-Level Triage`, not a universally valid numeric ID.

```powershell
$probe = cli-anything-nsight-graphics --json doctor info | ConvertFrom-Json
$ngfx = $probe.binaries.ngfx
$replayer = $probe.binaries.ngfx_replay
$replayArgs = '--present-hidden --vsync-off --no-present-blit --no-block-on-incompatibility --inject-full-frame-perf-marker --loop-count 3 "{0}"' -f $capture
$traceArgs = @(
    '--activity', 'GPU Trace Profiler',
    '--exe', $replayer, '--dir', 'E:/metallic', '--args', $replayArgs,
    '--output-dir', $out,
    '--start-on-replay-begin', '--stop-on-replay-end',
    '--max-duration-ms', '1000', '--auto-export', '--trace-timeout', '60',
    '--architecture', $architecture, '--metric-set-name', $metricSet,
    '--set-gpu-clocks', 'unaltered', '--disable-collect-shader-pipelines', '--verbose'
)
& $ngfx @traceArgs
$traceExitCode = $LASTEXITCODE
```

Use a hidden host launch when needed by the execution environment. The replay target uses hidden presentation. Check exports to confirm replay reset is excluded. `--no-block-on-incompatibility` avoids a dialog, not incompatibility. Disabling shader-pipeline collection suits this top-level triage recipe; omit it for supported pipeline/source attribution. Keep clock policy consistent for A/B; `unaltered` is not locked clocks.

Original script: `E:/metallic/build-relwithdebinfo/minizorah-nsight-20260913/RunTrace.ps1`. Its default repeat attempt historically failed: script existence is not success evidence. Sibling `Analyze.py` and `gpu-trace-clean/BASE_UNLOCKED/` document the accepted export.

Reject missing single-pass metric set, hardware-event buffer exhaustion, a competing failed replay, and unavailable performance counters. Fix the identified configuration/process issue before retrying. Do not change system-wide counter permissions automatically. If no valid capture is available, report the limitation and use RHI timing/controlled experiments with narrower claims.

## Attribution lessons

- Rechecking the historical export with harness 0.2.0 returned L2 3.54373% and DRAM null. Raw counters distinguish `GPUTrace.syslts__throughput...` (3.54373%) from `GPUTrace.lts__throughput...` (6.9127%); the project parser uses the latter and `dram__sectors.avg.pct_of_peak_sustained_elapsed`. Match exact names/units and explain metric semantics before using these values. Summary aliases are not authoritative; do not treat null as zero or silently equate sectors with a differently defined throughput metric.
- Map repeated markers to occurrence/early/late and queues. Broad `GPUDriven` ranges contain multiple stages; add instrumentation before calling all of it LOD traversal.
- Historical HW 0.911 ms and SW 1.020 ms overlap: their sum is not raster critical-path time. Range-wide counters can include the simultaneous branch.
- Historical classify barrier fractions 65.6% / 71.8% used stall-sample sums including selected/not_selected. They are not frame-time percentages or precise per-source-barrier attribution.
- Low SM/DRAM throughput plus single-workgroup dispatch can indicate insufficient parallelism. A long range does not prove bandwidth saturation; `HostUpload` does not prove physical allocation location.
- API counts and compatibility warnings are observations, not causal proof.

## Optimization acceptance

Preserve scene/camera, render extent, LOD/quality, page budget, material settings, queues and relevant residency/cut. Historical 1920 x 1080, 1 GiB and 1.5 px are comparison conditions, not mandatory defaults.

Candidate changes need stable visibility record/triangle IDs and ordering; cover zero/overflow, early/late recovery, coverage/equal-depth and reference cut/image. Classification must retain near-plane/conservative HW fallback semantics; measure classification plus raster join. Streaming needs request-to-drawable tails, upload/eviction/reload and quality convergence separately.

Later candidate A/B lowered fixed-view intervals 1.555 to 0.525 ms and 1.405 to 0.497 ms with identical PNGs, then used roaming validation. Those checkpoint intervals are not pure kernel durations or directly comparable with the old editor capture. Check current code before choosing the next optimization.

Wall-clock routes with asynchronous streaming vary per-frame residency/cut. Report sample count, warm-up, clocks and percentiles only for valid time series. One successful capture is one sample; failed repeats add no confidence. Use focused tests and proportional duration; the historical 660-second endurance run is not required for every edit.

## Historical compatibility failures

Local observations dated 2026-09-12, not universal claims. Inspect current engine workarounds first.

| Failure stage | Historical resolution | Boundary |
| --- | --- | --- |
| Capture interceptor in KHR OMM build | Active injection selects EXT OMM; ordinary execution retains KHR | Installation/SDK compilation does not imply injection. Keep OMM enabled. |
| DeviceLost with injection + Aftermath | Disable only automatic checkpoints for the tested combination | Retain dumps, shader info, resource tracking. The force-on env override is for deliberate retesting. |
| Replay restoring EXT OMM implicit indices | Engine supplies UINT32 identity indices; rebuild, restart, recapture | Old captures retain old data; replacing the executable cannot repair them. |

`VK_NV_low_latency` revision warning persisted in successful replay; it did not identify the OMM crash cause. Do not disable Reflex/OMM/async compute wholesale or routinely patch NVIDIA binaries. Removing a workaround needs fresh targeted reproduction plus correctness/replay checks and recorded versions. Existing unrelated VUIDs mean the run is not globally validation-clean.
