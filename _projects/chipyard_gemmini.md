---
layout: project
title: "Gemmini Offload Thresholds & Memory-Centric Pipeline Co-Design"
permalink: /projects/chipyard_gemmini
description: "When should GEMM stay on the CPU, when should it move to Gemmini, and how does the memory path shift that boundary?"
category: research
importance: 2
image: /assets/img/boomgemmini_mac100_before_after.png
links:
  - name: GitHub branch
    url: https://github.com/younghun-kim-dev/chipyard/tree/chipyard_gemmini
---

Independent Gemmini follow-up project testing whether **matrix GEMM workloads share the same memory-path limits** I saw in SHA-3. I built a **WS/OS GEMM framework** comparing CPU and Gemmini across matrix sizes, identified two regimes (overhead-dominated vs bandwidth-limited), and co-designed **scratchpad banking, bus width, and DMA alignment**. The optimized memory pipeline improves **1024³** throughput by ≈**58%** and lowers the **CPU↔Gemmini offload threshold K\***, validated on FireSim (AWS EC2 F1).

---

## Key questions

- For GEMM, when is it actually worth offloading to Gemmini instead of staying on the CPU?
- Do GEMM workloads exhibit the same **“compute is cheap, data movement sets the pace”** behavior as SHA-3?
- How much of the missing performance is in **Gemmini’s memory path** (SPM/ACC, buses, DMA), not its MAC array?

---

## Part 1 – RocketConfig WS/OS baseline

![MAC/100cyc vs matrix size (Rocket + Gemmini)](/assets/img/gemminirocket_mac100_vs_M.png)

- **Setup**
  - SoC: **GemminiRocketConfig** (Rocket core + Gemmini).
  - Sweep **M = N = K ∈ {8, …, 2048}**, with **WS** and **OS** dataflows.
  - Metrics: cycles, **MAC/100cyc**, correctness checks.

- **Findings**
  - Clear **two-regime behavior**:
    - Small matrices (≤32): dominated by **launch + data-movement overheads**.
    - Large matrices (≥256): **MAC/100cyc plateaus** → **bandwidth / tiling** dominate.
  - **WS > OS** across all sizes in both latency and MAC utilization.
  - This curve becomes the **baseline** for later BOOM+Gemmini and memory-path changes.

---

## Part 2 – BOOM+Gemmini memory-pipeline co-design

![WS/OS MAC/100cyc – before vs after memory-pipeline optimization](/assets/img/boomgemmini_mac100_before_after.png)

- **Setup**
  - SoCs:
    - **Before:** `GemminiLargeBoomV4Config`.
    - **After:** `GemminiLargeBoomV43Config`.
  - Changes implemented as Gemmini/Chipyard **config mixins**:
    - Scratchpad/accumulator (**SPM/ACC**) banking & sizing.
    - System-bus beat width.
    - Gemmini **DMA bus-width alignment**.
  - Workload: one-shot GEMM with **M = N = K ∈ {8, …, 1024}**, WS/OS.

- **Results (selected MAC/100cyc)**

  - At **1024³**:
    - WS: **16,047 → 25,433 MAC/100cyc** (≈**+58%** throughput).
    - OS: **13,828 → 17,004 MAC/100cyc** (≈**+23%** throughput).
  - Similar gains at 256–512³ show that the gap is **systematic**, not a one-off artifact.

- **Interpretation**
  - **Same MAC array, different memory path**:
    - The “missing” performance was sitting in **SPM/ACC layout, bus width, and DMA alignment**.
    - Memory-centric co-design raises Gemmini into a **higher-throughput plateau** without changing compute.

---

## Part 3 – CPU vs Gemmini offload threshold K\*

- **Offload-threshold pipeline**
  - On a BOOM+Gemmini SoC, I built a **CPU vs Gemmini threshold benchmark**:
    - Sweep tile shapes **(M, N)** and depths **K ∈ {1,…,8}**.
    - Compare blocked CPU GEMM vs Gemmini (OS) cycles.
    - Define **K\*(M, N)** = smallest K where Gemmini beats the CPU.

- **Empirical behavior**
  - Tiny tiles like **(4,4)** and **(8,4)** never beat the CPU for **K ≤ 8**  
    → launch & data-movement overhead dominate.
  - Larger, near-square tiles like **(10,10)** and **(12,12)** already win at **K\* = 1**.
  - Shape matters:
    - (8,12) has **K\* = 1**, while (12,8) has **K\* = 5**  
      → **loop/blocking order and memory layout** strongly affect offload decisions.

- **Speedups when offloaded**
  - When offloading is beneficial (Offload = 1):
    - Median speedup ≈ **1.7×**, mean ≈ **1.9×**, best cases ≈ **4×**.

---

## Takeaways

- GEMM confirms the same pattern I saw in SHA-3:
  - **Compute is cheap; data movement and overheads decide real speedup.**
- Offloading is **not “always good”**:
  - There is a real, measurable **offload threshold K\*(M, N)** that shifts with the memory path.
- This project turns that intuition into **measurement infrastructure**:
  - For any new Gemmini/SoC config, re-running the pipeline shows how memory-centric changes move K\* and reshape the performance envelope.
