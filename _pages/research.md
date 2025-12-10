---
layout: page
title: Research
permalink: /research/
description: Memory-centric architectures for accelerator-rich systems.
nav: true
nav_order: 2
---

My work focuses on **memory-centric architectures for accelerator-rich systems**. I study when accelerator speedups survive real cache hierarchies, DRAM, and schedulers instead of collapsing once they share memory with other workloads.

A single hardware-wallet security review failure pushed me toward this question: we had integrated crypto accelerators for speed, but overlooked data movement that compromised key isolation. Fixing that flaw led me to treat **data movement as the core of architectural reasoning**, not an afterthought.

---

## Core research question

> Under what memory-system and sharing conditions do accelerator speedups  
> survive integration versus collapse at scale?

To answer this, I use RISC-V SoCs built with **Chipyard**, **Gemmini**, and **custom RoCC accelerators**, and run them under **cycle-accurate simulation (Verilator + DRAMSim2)** and **FPGA-accelerated emulation (FireSim on AWS EC2 F1)**.

### Themes

- **Data movement & memory hierarchy**
  - How L1/L2 design, banking, and DRAM topology limit accelerators long before compute saturates.
- **Shared-memory contention & predictability**
  - How co-running workloads on CPUs and accelerators reshape throughput and tail latency.
- **Hardware–software co-design**
  - Using runtime policies (scheduling, offload decisions) together with architectural changes to keep performance in a tight, predictable band.

---

## Independent Chipyard research line

### 1. SHA-3 accelerator: speedups vs message size

I started with an **independent Chipyard project** analyzing when SHA-3 accelerator speedups persist as message sizes scale.

- Integrated a **SHA-3 RoCC** with a Rocket core and built an **automated benchmark pipeline** for messages from 136&nbsp;B to 544&nbsp;KB.
- Cycle-accurate Verilator traces showed that, as sizes grew, speedup shrank from **206× → 120×** even though the accelerator itself was not saturated.
- The culprit was a **single-bank inclusive L2**: misses, refills, and coherence traffic serialized on one bank, stalling the SHA-3 accelerator.
- I **redesigned the inclusive L2 as a multi-banked cache**, increasing capacity and sub-banking concurrency.
- At 557&nbsp;KB, this raised SHA-3 speedup from **120.27× → 160.90×** (≈34% throughput gain), pulling large-input behavior back toward the ~170× plateau.

This project convinced me that “headline speedups” only mean anything when paired with a **memory-path design that scales to realistic working sets**.

---

### 2. Gemmini: offload thresholds & memory-centric pipeline co-design

Next, I asked whether **matrix GEMM workloads** on a systolic array would show the same memory-path limits.

- Built a **WS/OS benchmarking framework** comparing CPU and Gemmini across matrix sizes, measuring cycles and MAC/100 cycles.
- Identified two regimes:
  - **Small matrices**: dominated by launch and data-movement overheads; accelerator underutilized.
  - **Large matrices**: dominated by **bandwidth and tiling**, with MAC utilization plateauing.
- Co-designed the **memory pipeline**:
  - Re-banked and resized Gemmini scratchpad/accumulator.
  - Widened the system bus beat width.
  - Aligned Gemmini DMA width to the memory system.
- These changes **lowered offload thresholds** (making Gemmini worthwhile for smaller tiles) and improved **1024³ GEMM throughput by ≈58%**, validated on **FireSim (AWS EC2 F1)**.
- A separate offload-threshold study showed that offloading is not “always good”: the **critical K\*(M, N)** depends strongly on tile shape, memory layout, and hierarchy.

Here, the main result was a **quantitative, reusable pipeline** for deciding *when* to offload to an accelerator under a given memory system.

---

### 3. Heterogeneous SoC memory contention: BOOM + Rocket + Gemmini

Real systems rarely run a single accelerator in isolation, so my next question was how **shared-memory contention** shapes accelerator behavior when workloads co-run.

- Built a **co-run profiling framework** on a heterogeneous RISC-V SoC with **BOOM + 2×Rocket + Gemmini**, plus a configurable memory topology.
- Implemented a **bandwidth stress generator** on the Rocket cores and **cycle-accurate per-tile/per-kernel logging** for Gemmini.
- Sensitivity sweeps showed that, under 8&nbsp;MiB co-run stress with a single-channel DRAM and default L2:
  - A 256×256 GEMM slowed by ≈**3×**.
  - **DRAM bandwidth saturation** and **L2 bank contention** created long, structured stalls.
- Adding **more DRAM channels alone** helped only modestly; the shared L2 remained a bottleneck.
- Combining **2 DRAM channels + 4-bank L2 + tuned memory-controller parameters** pulled GEMM latency back near the low-stress regime and tightened tile-latency distributions.
- A lightweight **phase-aware scheduling** policy that staggered peaks across co-running workloads raised **aggregate throughput by up to 2.7×** and reduced jitter.

This project showed that, under contention, **memory topology and scheduling, not raw compute, dominate observed accelerator behavior**.

---

## Methodology and tools

Across these projects, I follow a consistent **measurement-first, integration-first** methodology:

- **Start from integration, not from peak FLOPs**
  - Design experiments that include caches, coherence, DRAM timing, and co-running workloads from the beginning.
- **Use cycle-accurate tools, then accelerated platforms**
  - Prototype in **Chipyard + Verilator + DRAMSim2**, then move heavy workloads to **FireSim on AWS F1** when needed.
- **Trace memory paths explicitly**
  - Collect UART logs, cycle dumps, and per-tile/per-phase traces to separate compute, cache, and DRAM effects.
- **Close the loop with co-design**
  - Use measurements to drive coordinated changes in **accelerator interfaces, memory hierarchy, and runtime policies** (e.g., offload thresholds, schedulers).

Tool stack (selected):

- **Architecture & RTL**: Chipyard, BOOM, Rocket, Gemmini, RoCC, Verilog/SystemVerilog, Chisel/Scala
- **Simulation & emulation**: Verilator + DRAMSim2, FireSim on AWS EC2 F1
- **Systems**: Linux, Buildroot, Docker, Git/CI

---

## Longer-term directions

Going forward, I want to generalize these ideas beyond single-node RISC-V prototypes:

- **Multi-accelerator, multi-tenant systems**
  - Extending my co-run and offload-threshold analyses to systems where multiple accelerators (e.g., matrix engines, crypto, DSAs) share memory and interconnects.
- **Emerging memory systems**
  - Applying the same bandwidth-sensitivity and contention profiling to **CXL-based tiered memory**, **processing-in-memory**, and **chiplet-based DRAM** architectures.
- **Predictable performance under shared resources**
  - Developing **design rules and runtime policies** that keep accelerator platforms within a tight, predictable performance band under realistic contention and power limits.

Ultimately, I aim to build **memory-centric accelerator platforms that keep their promises**: when multiple tenants and real memory systems enter the picture, performance should degrade gracefully, not collapse unexpectedly.

For concrete technical details and plots, see the [Projects](/projects/) page and the associated branches in my Chipyard fork:
- `chipyard_sha3`
- `chipyard_gemmini`
- `chipyard_hetero`
