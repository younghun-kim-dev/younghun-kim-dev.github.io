---
layout: page
title: Research
permalink: /research/
description: Memory-centric architectures for accelerator-rich systems.
nav: true
nav_order: 3
---

My work focuses on **memory-centric architectures for accelerator-rich systems**: I study when accelerator speedups survive real cache hierarchies, DRAM, and schedulers instead of collapsing once they share memory.

A failed hardware-wallet security review pushed me to treat **data movement as the core of architectural reasoning** rather than an afterthought: crypto accelerators were fast, but unsafe data paths broke key isolation. The same patterns now guide how I analyze accelerators on RISC-V SoCs.

---

## Core question & themes

> Under what memory-system and sharing conditions do accelerator speedups  
> survive integration versus collapse at scale?

**Themes**

- **Data movement & memory hierarchy**  
  How L1/L2 structure, banking, and DRAM topology limit accelerators long before compute saturates.
- **Shared-memory contention & predictability**  
  How co-running CPU/accelerator workloads reshape throughput and tail latency.
- **Hardware–software co-design**  
  Using schedulers and offload policies together with memory-path changes to keep performance in a tight, predictable band.

I prototype on **Chipyard RISC-V SoCs** (Rocket, BOOM, Gemmini, custom RoCCs) using **Verilator + DRAMSim2** and scale to FPGA-based emulation with **FireSim on AWS EC2 F1**.

---

## Independent Chipyard research line

### 1. SHA-3 accelerator: speedups vs message size (`chipyard_sha3`)

- Integrated a **SHA-3 RoCC** with Rocket/BOOM and built an automated pipeline for messages from 136&nbsp;B to 544&nbsp;KB.
- Found speedup collapsing from **206× → 120×** at large sizes due to a **single-bank inclusive L2**, not the accelerator itself.
- Redesigned the L2 as a **multi-bank inclusive cache**, recovering ≈**34% throughput** at 557&nbsp;KB and pulling behavior back toward the ~170× speedup plateau.

**Takeaway:** headline accelerator speedups are only meaningful with a **memory path that scales** to realistic working sets.

---

### 2. Gemmini: offload thresholds & memory pipeline (`chipyard_gemmini`)

- Built a **WS/OS GEMM framework** comparing CPU vs Gemmini across matrix sizes and shapes, measuring cycles and MAC/100 cycles.
- Identified **overhead-dominated small matrices** and **bandwidth-limited large matrices**, showing where the accelerator is underutilized.
- Co-designed **SPM/ACC banking**, **system bus width**, and **DMA width alignment**, lowering offload thresholds and improving **1024³ throughput by ≈58%**, validated on **FireSim**.

**Takeaway:** offloading is **not always good**—the critical **\(K^*(M,N)\)** is a measurable function of tile shape, memory layout, and hierarchy.

---

### 3. Heterogeneous SoC memory contention (`chipyard_hetero`)

- Built a **co-run profiling framework** on a BOOM + 2×Rocket + Gemmini SoC with configurable DRAM/L2 topology.
- Showed that with a single DRAM channel and default L2, co-run stress slows a 256×256 GEMM by ≈**3×** via **DRAM saturation** and **L2 bank contention**.
- Combining **2 DRAM channels + 4-bank L2 + tuned memory-controller parameters** and **phase-aware scheduling** recovered throughput by up to **2.7×** and tightened tile-latency bands.

**Takeaway:** under contention, **memory topology and scheduling**, not raw compute, dominate accelerator behavior.

---

## Methodology & tools

Across these projects, I use a **measurement-first, integration-first** loop:

- Start from **full systems** (caches, coherence, DRAM timing, co-running workloads), not idealized standalone accelerators.
- Use **Chipyard + Verilator + DRAMSim2** for cycle-accurate traces, then move to **FireSim** when workloads or interference patterns outgrow software simulation.
- Trace the **memory path explicitly** (UART logs, per-tile/per-phase cycles) to separate compute vs cache vs DRAM effects.
- Use these measurements to co-design **accelerator interfaces, memory hierarchies, and runtime policies** (offload thresholds, schedulers).

Tool stack (selected):

- **Architecture & RTL**: Chipyard, BOOM, Rocket, Gemmini, RoCC, Verilog/SystemVerilog, Chisel/Scala  
- **Simulation & emulation**: Verilator + DRAMSim2, FireSim on AWS EC2 F1  
- **Systems**: Linux, Buildroot, Docker, Git/CI

---

## Longer-term directions

I plan to extend this line of work to:

- **Multi-accelerator, multi-tenant systems**  
  Scaling co-run and offload-threshold analyses to multiple DSAs sharing memory and interconnects.
- **Emerging memory systems**  
  Applying bandwidth-sensitivity and contention profiling to **CXL-based tiered memory**, **PIM**, and **chiplet DRAM**.
- **Predictable performance under shared resources**  
  Developing **design rules and runtime policies** that keep accelerators within a tight performance band under contention and power limits.

Ultimately, I want to build **memory-centric accelerator platforms that keep their promises**: when real memory systems and multi-tenant workloads appear, performance should **degrade gracefully**, not collapse.

For concrete figures and code, see the [Projects](/projects/) page and the corresponding branches in my Chipyard fork:
- `chipyard_sha3`
- `chipyard_gemmini`
- `chipyard_hetero`
