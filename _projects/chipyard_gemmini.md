---
layout: page
title: "Gemmini Offload Thresholds & Memory-Centric Pipeline Co-Design"
permalink: /projects/chipyard_gemmini/
description: >
  Independent Gemmini project studying CPU vs Gemmini offload thresholds and
  how memory-path co-design (SPM/ACC banking, bus widths, DMA alignment) shifts
  those thresholds and large-matrix throughput.
image: /assets/img/boomgemmini_mac100_before_after.png
category: work
importance: 1
tags:
  - Chipyard
  - RISC-V
  - Gemmini
  - memory systems
  - accelerators
links:
  github: "https://github.com/younghun-kim-dev/chipyard/tree/chipyard_gemmini"
---

## Overview

- **Goal.** Determine **when** GEMM workloads should be offloaded to Gemmini vs left
  on the CPU, and **how memory-path design shifts that boundary**.
- **Timeline.** Jun. 2025 – Aug. 2025 (independent follow-up to the SHA-3 project).
- **Stack.** Gemmini + Chipyard (Rocket / BOOM), Verilator + DRAMSim2,
  FireSim (AWS EC2 F1), custom WS/OS benchmarking and threshold scripts.

After seeing SHA-3 speedups collapse under a narrow L2, I wanted to know whether
matrix compute on a systolic array would exhibit the same memory-path limits and how
to **co-design scratchpads, buses, and DMA** to fix them.

---

## Research questions

1. On a simple Rocket+Gemmini SoC, how do **WS/OS GEMM** behaviors split into
   overhead-dominated vs bandwidth-dominated regimes?
2. On a BOOM+Gemmini SoC, how much of the “missing” performance is really in the
   **memory pipeline**, not the MAC array?
3. For small/medium GEMM tiles, what is the **offload threshold \(K^*(M,N)\)** where
   Gemmini first beats the CPU, and how does memory-path design shift \(K^*\)?

---

## Part 1 – WS/OS baseline: Rocket + Gemmini

On `GemminiRocketConfig` (Rocket core + Gemmini):

- **Benchmark.** `ws_os_single-baremetal`.
- **Workload.** GEMM with `M = N = K ∈ {8,16,32,64,128,256,512,1024,2048}`.
- **Metrics.** Total cycles and `MAC/100cyc` for both WS and OS.

Key observations:

- The **cycles vs size** curves are near-cubic, but `MAC/100cyc` reveals structure:
  - **Small matrices (≤32).**
    - Dominated by **launch + data-movement overhead**.
  - **Large matrices (≥256).**
    - `MAC/100cyc` plateaus → **bandwidth / tiling limits** dominate.
- WS is consistently more efficient than OS; WS and OS give two views of the same
  memory path.
- Additional WS/OS heatmaps at `K = 256` visualize how **tile shape (M,N)** changes
  throughput, with near-square tiles clearly better than very skinny ones.

This baseline sets the template for later “before vs after” comparisons.

---

## Part 2 – Memory-pipeline optimization on BOOM + Gemmini

On a BOOM+Gemmini SoC, I compared:

- **Before.** `GemminiLargeBoomV4Config`.
- **After.**  `GemminiLargeBoomV43Config`.

The MAC array is identical; only the **memory pipeline** changes via config mixins:

- Scratchpad / accumulator (**SPM/ACC**) banking and sizing.
- System bus **beat width**.
- Gemmini **DMA bus-width alignment**.

**Benchmark.**

- `ws_os_single2-baremetal`, one-shot GEMM with
  `M = N = K ∈ {8,16,32,64,128,256,512,1024}`, WS and OS.
- Metrics: `MAC/100cyc` and total cycles.

**Key result.**

- For WS at \(1024^3\):
  - Before: `MAC/100cyc ≈ 16k`.
  - After:  `MAC/100cyc ≈ 25k`.
  - → **≈58% throughput gain** and ≈37% latency reduction.
- OS also improves (≈23% throughput gain at \(1024^3\)).
- Across sizes, the “after” curves form a **higher plateau**, proving that the
  previous limit was in the memory path, not in Gemmini’s compute.

This is the matrix-compute analog of the SHA-3 L2-banking effect: **same compute,
different memory**, very different perceived accelerator.

---

## Part 3 – CPU vs Gemmini offload threshold \(K^*\)

To quantify **when offloading is actually worthwhile**, I built a small-tile threshold
pipeline on a BOOM+Gemmini SoC:

- **SoC.** `GemminiLargeBoomV4Config`.
- **Benchmark.** `ws_os_threshold-baremetal`.
- **Workload.**
  - Tile sizes `(M,N) ∈ {4,6,8,10,12} × {4,6,8,10,12}`.
  - For each `(M,N)`, sweep `K ∈ {1,…,8}`.
- **Comparison.**
  - CPU blocked GEMM vs Gemmini OS dataflow.
- **Metric.**
  - `K^*(M,N)` = smallest K where **ACC cycles < CPU cycles**.
  - When offloaded, median speedup ≈1.7×, mean ≈1.9×, best ≈4×.

Empirical patterns:

- Tiny tiles like `(4,4)` and `(8,4)` **never benefit** from Gemmini up to `K=8`:
  launch & data-movement overhead dominate.
- Near-square larger tiles such as `(10,10)` and `(12,12)` have `K^* = 1`:
  the accelerator’s setup cost is amortized immediately.
- Shape matters:
  - `(8,12)` has `K^* = 1` while `(12,8)` has `K^* = 5`,
  - showing that **loop/blocking order and layout** strongly affect offload decisions.

The takeaway is that **offloading is not “always good”**; it is a measurable policy
problem governed by the memory path and tile geometry.

---

## What I learned

- Gemmini’s raw MAC throughput is only meaningful when paired with a **co-designed
  memory pipeline** and **offload policy**.
- WS/OS sweeps, heatmaps, and `K^*(M,N)` give a compact, reusable way to characterize
  an accelerator’s memory behavior.
- Many “underperforming” accelerators are really **memory-path design** and
  **offload-policy** problems, not compute limits.

This project sets up the heterogeneous SoC work, where I push beyond single-workload
evaluation and ask how these accelerators behave when they **compete** for DRAM and L2.

---

## Code & data map

Key locations in the `chipyard_gemmini` branch:

- **SoC configs.**
  - `generators/chipyard/src/main/scala/config/GemminiBoomV4Configs.scala`  
    BOOM + Gemmini configs for before/after memory-pipeline experiments.
- **Gemmini mixins (memory pipeline).**
  - Scratchpad/accumulator tuning, system-bus width, Gemmini DMA bus-width alignment
    (file names vary with submodule version; all are documented in the README).
- **Benchmarks.**
  - `generators/gemmini/software/gemmini-rocc-tests/`:
    - `ws_os_single-baremetal` – RocketConfig WS/OS sweep.
    - `ws_os_single2-baremetal` – BOOM+Gemmini before/after GEMM.
    - `ws_os_threshold-baremetal` – CPU vs Gemmini offload thresholds.
- **Results & plots.**
  - `sims/verilator/**/` – UART logs, CSV summaries.
  - `figs/gemminirocket_mac100_vs_M.png`,
    `figs/boomgemmini_mac100_before_after.png`,
    and WS/OS heatmaps for `K=256`.
