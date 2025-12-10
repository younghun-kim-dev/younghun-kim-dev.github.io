---
layout: page
title: "Heterogeneous SoC Memory Contention: Diagnosis & Mitigation"
permalink: /projects/chipyard_hetero/
description: >
  Co-run profiling on a BOOM + 2×Rocket + Gemmini SoC to understand how shared
  memory contention shapes accelerator behavior and how DRAM/L2 topology and
  phase-aware scheduling restore predictable speedups.
image: /assets/img/fig1_gemm_latency.png
category: work
importance: 1
tags:
  - Chipyard
  - RISC-V
  - Gemmini
  - BOOM
  - memory contention
links:
  github: "https://github.com/younghun-kim-dev/chipyard/tree/chipyard_hetero"
---

<p class="mt-3 mb-4">
  <a href="{{ '/projects/' | relative_url }}" class="btn btn-outline-secondary">
    &larr; Back to Projects
  </a>
</p>

## Overview

- **Goal.** Understand how **shared-memory contention** affects a Gemmini accelerator
  when workloads co-run on a heterogeneous RISC-V SoC, and how memory topology and
  scheduling can restore predictable performance.
- **Timeline.** Aug. 2025 – Oct. 2025 (independent project).
- **Stack.** Chipyard (BOOM + 2×Rocket + Gemmini), Verilator + DRAMSim2,
  custom co-run benchmarks and tile-level logging.

Real systems rarely run a single accelerator in isolation. This project asks:
> When BOOM, two Rocket cores, and Gemmini share DRAM and L2, who actually owns the
> memory path, and what does it take to keep Gemmini’s speedups intact?

---

## Research questions

1. How do we **reproduce and control** shared-memory contention in a BOOM+Rocket+Gemmini SoC?
2. Under co-run bandwidth stress, **how much** can a 256×256 GEMM slow down and why?
3. Which changes—DRAM channels, L2 banking, memory-controller tuning, or scheduling—
   actually restore both **throughput** and **predictability**?

---

## Co-run profiling framework

I first built a reusable co-run framework on a heterogeneous SoC:

- **SoC topology.**
  - BOOM + 2×Rocket + Gemmini, with multiple DRAM/L2 configurations:
    - 1-channel / default L2.
    - 2-channel / default L2.
    - 2-channel / 4-bank L2 with tuned memory-controller parameters.
- **Harts.**
  - Hart 0 – Rocket#1: linear memory **bandwidth stressor**.
  - Hart 1 – Rocket#2: independent bandwidth stressor.
  - Hart 2 – BOOM: Gemmini host running GEMM kernels.
- **Co-run benchmarks.**
  - `hetero_gemm_bwtest-baremetal`
    - One-shot Gemmini GEMM (256×256×256).
    - Two Rocket linear “mem-stress” loops.
    - Logs per-hart start/finish cycles and GEMM total cycles.
  - `hetero_gemm_bwtest2-baremetal`
    - Same co-run setup, but splits GEMM into 16 tiles of 64×64.
    - Logs **per-tile latency**:
      - `[tile i=0 j=0] im=64 jn=64 -> cycles=...`.
- **Stress knobs.**
  - Each Rocket streams **2, 8, or 16 MiB** linearly from DRAM (stride 64).
- **Simulator.**
  - Verilator harness with DRAMSim2; generous `max-cycles` to avoid timeouts.

This framework lets me dial contention up/down and measure **per-hart and per-tile**
timing, not just aggregate runtime.

---

## One-shot GEMM under DRAM/L2 contention

### Setup

- **Workload.** 256×256×256 Gemmini GEMM.
- **Co-run stress.** Two Rocket harts each streaming 2, 8, or 16 MiB.
- **Configs.**
  - `GemminiLargeBoomV4Rocket2Config` – 1 DRAM channel, default L2.
  - `GemminiLargeBoomV4Rocket22CHConfig` – 2 channels, default L2.
  - `GemminiLargeBoomV4Rocket22CHL24BanksConfig` – 2 channels, 4-bank L2, tuned MC.
- **Metric.** GEMM cycles and slowdown vs no-stress baseline.

### Key result

- Baseline (almost no co-run stress):  
  GEMM ≈ **180k cycles** (1.00×).
- With **1 channel + default L2**, 8–16 MiB stress:
  - GEMM ≈ **552k cycles** → **3.07× slowdown**.
  - Additional stress doesn’t change latency → DRAM/L2 are saturated.
- Adding a second DRAM channel alone:
  - Improves slightly to ≈2.89× slowdown, but Gemmini is still heavily delayed.
- Combining **2 channels + 4-bank L2 + tuned controller** under 8 MiB stress:
  - GEMM drops to ≈**207k cycles** → ≈**1.15× slowdown**,
  - almost back in the low-stress regime.

This shows that **shared-memory contention alone**, with the same Gemmini array, can
turn a 256×256 GEMM from ~180k cycles into ~552k cycles—**about 3× slower**—and that
carefully co-designed memory topology can almost fully restore performance.

---

## Tile-level behavior & predictability

To understand **predictability**, not just mean throughput, I switched to tile-level
logging with `hetero_gemm_bwtest2-baremetal`:

- **Tiles.** 16 sub-GEMMs of 64×64.
- **Configs compared (8 MiB stress).**
  - 1 CH + default L2.
  - 2 CH + 4-bank L2 (tuned MC).

From the logs:

- **1 CH + default L2 (8 MiB).**
  - Mean tile latency ≈ **40.5k cycles**, std-dev ≈ **3.8k**.
  - First tile spikes above 52k; tiles are **slow and jittery**.
- **2 CH + 4-bank L2 (8 MiB).**
  - Mean tile latency ≈ **13.7k cycles**, std-dev ≈ **0.95k**.
  - All tiles cluster tightly → **fast and stable**.

So under identical co-run stress:

- Average tile latency improves by **≈2.95×**.
- Variability shrinks by **≈4×**.

It’s not just that the accelerator runs faster; it runs in a **tight, predictable
band**, which is exactly the property you want under shared-memory constraints.

---

## Phase-aware scheduling

Hardware changes alone aren’t the whole story. Based on the per-hart and per-tile
timelines, I added a lightweight **phase-aware scheduling** policy:

- Use simple phase markers from the co-run logs to stagger the peaks of:
  - Rocket stress loops, and
  - Gemmini’s most bandwidth-intensive phases.
- Avoids lining up worst-case bursts from all harts at once.

With tuned memory topology **plus** phase-aware scheduling, I observed:

- **Aggregate system throughput** improved by up to **2.7×** under heavy stress.
- Gemmini’s tile-latency distribution stayed tight, instead of occasionally
  exploding under unlucky alignments.

This reinforces the idea that **shared-memory behavior is a HW/SW co-design problem**:
DRAM channels, L2 banking, controller tuning, and scheduling all have to support the
accelerator if its speedups are to survive under real workloads.

---

## What I learned

- “Accelerator performance” is often dominated by **memory topology + scheduler**, not
  just by the array itself.
- Co-run profiling with per-tile logging exposes **structured contention patterns**
  that you can actually fix; slowdowns aren’t random noise.
- A memory system that keeps performance in a **tight, predictable band** under
  co-run stress is just as important as hitting a high single-workload peak.

This project ties together the earlier SHA-3 and Gemmini work into a more complete,
heterogeneous picture and points directly toward my current focus on **memory-centric,
integration-first architectures**.

---

## Code & data map

Key locations in the `chipyard_hetero` branch:

- **SoC configs.**
  - `generators/chipyard/src/main/scala/config/`:
    - `GemminiLargeBoomV4Rocket2Config`
    - `GemminiLargeBoomV4Rocket22CHConfig`
    - `GemminiLargeBoomV4Rocket22CHL24BanksConfig`
- **Co-run benchmarks.**
  - `generators/gemmini/software/gemmini-rocc-tests/`:
    - `hetero_gemm_bwtest.c` – one-shot 256×256 GEMM + 2×Rocket stressors.
    - `hetero_gemm_bwtest2.c` – same, but with 16× 64×64 tiles and per-tile logs.
- **Results & plots.**
  - `sims/verilator/output/chipyard.harness.TestHarness.*` – UART logs and cycle
    dumps for each config.
  - `figs/fig1_gemm_latency.png` – GEMM latency vs topology and stress.
  - `figs/fig2_tile_latency.png` – tile-latency distributions (1 CH vs 2 CH + 4-bank).

All of the numbers quoted above can be reproduced from these configs, benchmarks,
and measurement scripts in the repo.
