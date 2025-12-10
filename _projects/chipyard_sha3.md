---
layout: page
title: "SHA3 Accelerator Performance Stabilization in Chipyard"
permalink: /projects/chipyard_sha3/
description: >
  Independent Chipyard project analyzing when SHA-3 accelerator speedups survive
  as message sizes scale, and how multi-bank L2 design restores large-input
  performance.
image: /assets/img/sha3_speedup_vs_size_with_arrow_colored.png
category: work
importance: 1
tags:
  - Chipyard
  - RISC-V
  - SHA-3
  - memory hierarchy
  - accelerators
links:
  github: "https://github.com/younghun-kim-dev/chipyard/tree/chipyard_sha3"
---

<p class="mt-3 mb-4">
  <a href="{{ '/projects/' | relative_url }}" class="btn btn-outline-secondary">
    &larr; Back to Projects
  </a>
</p>

## Overview

- **Goal.** Understand under what memory-system and sharing conditions a SHA-3 RoCC
  retains its ~170–200× speedup as messages grow from cache-sized to hundreds of KB.
- **Timeline.** Apr. 2025 – Jun. 2025 (independent project).
- **Stack.** Chipyard (Rocket / BOOM + RoCC), Verilator + DRAMSim2,
  custom benchmarking scripts in `sims/verilator/`.

A hardware-wallet security failure pushed me toward a simple question:

> If an accelerator advertises a 200× speedup, under what memory conditions does that
> speedup actually survive?

This project uses SHA-3 as a concrete case and shows that a single-bank inclusive L2
can silently turn a “~170× accelerator” into a “120× accelerator” at scale, and that
a multi-bank L2 almost completely recovers the loss.

---

## Research question

As message size scales from **136 B → 557 KB**, I asked:

1. **When does the SHA-3 RoCC keep its headline speedups (≈170–200×)?**
2. **When and why do speedups collapse, even though the accelerator core is unchanged?**
3. **Which part of the memory path (L1, TLB, L2 banking, DRAM channels) is
   responsible?**

---

## Methodology & setup

- **SoC & configs.**
  - Integrated a SHA-3 RoCC into Rocket and mixed Rocket+BOOM SoCs.
  - Defined a config matrix in Chipyard:
    - **Baseline.** Single-bank inclusive L2 with SHA-3 + Rocket/BOOM.
    - **Variants.** 1/2/4/8-bank L2, SW-only baselines, extra memory channels.
- **Benchmark pipeline.**
  - Two binaries per config:
    - `sha3-rocc.riscv` – SHA-3 via RoCC accelerator.
    - `sha3-sw.riscv` – software-only SHA-3 on the same core.
  - For each size, log:
    - `hw_cycles`, `sw_cycles`, `speedup = sw_cycles / hw_cycles`.
- **Message sizes.**
  - Swept **136 × 2ᵖ B** for p = 0…12:
    - 136, 272, …, 557,056 bytes.
- **Automation.**
  - `sha3_compare.sh` runs HW vs SW once and emits `sha3_speed.csv`.
  - `gather_sha3_data.sh` aggregates per-size CSVs into
    `combined_sha3_speed.csv` used to plot the speedup curve.

This keeps the experiment fully reproducible from the repo: new configs can reuse the
same scripts to generate new curves.

---

## Key findings

### 1. Plateau then collapse with a single-bank inclusive L2

With a **single-bank inclusive L2**, speedup behaves as follows:

- **Small–mid sizes (≈2–70 KB).**
  - Speedup stays in a **stable ~168–176× band**.
  - The accelerator is well-fed; memory is not yet the bottleneck.
- **Medium sizes (≈140–280 KB).**
  - Speedup drifts down to **~145–160×**.
  - Working set begins to stress L2 capacity / associativity.
- **Largest size (557,056 B).**
  - Speedup **collapses to 120.27×**.
  - Cycle-accurate traces show the core is not saturated; instead:
    - misses, refills, and evictions serialize on the single L2 bank,
    - coherence and refill traffic back up behind a narrow bottleneck.

So from the system’s point of view, the same RoCC looks like:

- **≈170×** at mid-size messages, but only  
- **~120×** at application-scale messages.

The difference is entirely in the memory path.

### 2. Multi-bank inclusive L2 recovers ~34% throughput at scale

Redesigning the inclusive L2 as a **multi-bank cache** (with more effective capacity
and sub-banking concurrency):

- At **557,056 B**:
  - Single-bank L2: `speedup ≈ 120.27×`.
  - Multi-bank L2: `speedup ≈ 160.90×`.
- This is roughly a **+34% improvement in throughput**, and the new point at 557 KB
  sits **back near the ~170× plateau** seen at mid sizes.

The SHA-3 core and ISA are unchanged; only L2 structure is different.

**Interpretation.**

- “Speedup” is meaningless without a **memory-path contract**.
- At realistic message sizes, the platform advertised “170×” but delivered “120×”
  purely because of L2 banking.
- Once L2 is redesigned, the same accelerator again behaves like the promised
  high-speedup unit.

---

## What I learned

- Accelerator evaluation must treat **memory hierarchy as part of the accelerator**.
- A single number (“206× speedup”) hides the structure of the curve; profiling across
  sizes is essential.
- Multi-bank L2 design and concurrency matter as much as the RoCC datapath for
  real-world throughput.

This project directly seeded my later work on Gemmini and heterogeneous SoCs, where I
treated the memory subsystem and scheduler as first-class levers for stabilizing
accelerator performance.

---

## Code & data map

Key locations in the `chipyard_sha3` branch:

- **Configs.**
  - `generators/chipyard/src/main/scala/config/AccelMemSweep.scala`  
    SHA-3 + Rocket/BOOM configs; 1/2/4/8-bank L2, SW-only baselines.
  - `generators/chipyard/src/main/scala/config/RocketSha3Configs.scala`  
    Rocket-only SHA-3 configs, including 4-bank L2 + extra memory channels.
- **Benchmarks.**
  - `generators/sha3` (forked submodule) – SHA-3 RoCC implementation.
  - `sims/verilator/sha3_compare.sh` – run HW vs SW once and emit `sha3_speed.csv`.
  - `sims/verilator/gather_sha3_data.sh` – sweep sizes and aggregate into
    `combined_sha3_speed.csv`.
- **Results & plots.**
  - `sims/verilator/**/` – per-size logs (`*.log`, `hw.cycs`, `sw.cycs`, CSVs).
  - `figs/sha3_speedup_vs_size_with_arrow_colored.png` – speedup curve with
    the 557 KB improvement highlighted.
