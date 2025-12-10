---
layout: project
title: "Heterogeneous SoC Memory Contention: Diagnosis & Mitigation"
permalink: /projects/chipyard_hetero
description: "BOOM + 2×Rocket + Gemmini co-run profiling that turns noisy slowdowns into a measured, fixable memory-path problem."
category: research
importance: 1
image: /assets/img/fig1_gemm_latency.png
links:
  - name: GitHub branch
    url: https://github.com/younghun-kim-dev/chipyard/tree/chipyard_hetero
---

Independent Chipyard project investigating **how shared-memory contention shapes accelerator behavior** when workloads co-run on a heterogeneous RISC-V SoC (BOOM, 2×Rocket, Gemmini). I built a **co-run profiling framework** with a configurable contention generator and per-tile logging, showed that DRAM saturation and L2 bank contention can slow a 256×256 GEMM by ≈**3×**, and then used **memory-topology tuning + phase-aware scheduling** to recover up to **2.7×** throughput and tighten tile-level latency bands.

---

## Key questions

- When BOOM+Gemmini share DRAM and L2 with CPU cores, what actually causes **multi× slowdowns**?
- Do slowdowns look random, or do they follow **structured phases** tied to memory topology and stress patterns?
- Can we restore throughput and **predictability** with coordinated HW/SW changes, without touching Gemmini’s MAC array?

---

## Co-run profiling framework

![GEMM latency vs memory topology and stress](/assets/img/fig1_gemm_latency.png)

- **SoC topology (Chipyard configs)**
  - BOOM + 2×Rocket + Gemmini with variants like:
    - `GemminiLargeBoomV4Rocket2Config` – 1 DRAM channel, default L2.
    - `GemminiLargeBoomV4Rocket22CHConfig` – 2 DRAM channels, default L2.
    - `GemminiLargeBoomV4Rocket22CHL24BanksConfig` – 2 DRAM channels + 4-bank L2, tuned controller.

- **Role assignment**
  - Hart 0 – Rocket #1: memory-bandwidth stressor.
  - Hart 1 – Rocket #2: independent memory-bandwidth stressor.
  - Hart 2 – BOOM: Gemmini host running GEMM kernels.

- **Benchmarks**
  - `hetero_gemm_bwtest-baremetal`
    - One-shot Gemmini GEMM (256×256×256) co-running with two Rocket linear “mem-stress” loops.
    - Sweeps stress sizes (2 / 8 / 16 MiB) and records per-hart completion.
  - `hetero_gemm_bwtest2-baremetal`
    - Same co-run setup, but splits the GEMM into **16× 64×64 tiles** and logs **per-tile cycles**.

- **Simulator**
  - Verilator harness + DRAMSim2 to model realistic DRAM timing, with large max-cycles to survive heavy stress.

---

## One-shot GEMM under co-run stress

- **Workload**
  - Gemmini WS GEMM: **256×256×256**.
  - Two Rocket harts each streaming **2 / 8 / 16 MiB** from DRAM (stride 64).

- **Behavior**
  - Baseline (almost no stress): **≈180k cycles**.
  - Single-channel DRAM + default L2, **8–16 MiB** stress:
    - GEMM latency jumps to **≈552k cycles** → **3.07× slowdown**.
    - Increasing stress beyond 8 MiB doesn’t change latency → **fully bandwidth-limited**.
  - Adding a **second DRAM channel alone** only reduces slowdown slightly (~2.89×).

- **Topology + controller tuning**
  - With **2 DRAM channels + 4 L2 banks + tuned controller**:
    - 8 MiB stress run recovers to **≈207k cycles** (≈**1.15×** vs baseline).
    - This is ≈**2.7× faster** than the 1-channel/8 MiB case, with the **same Gemmini compute array**.

---

## Tile-level predictability

![Tile-latency distributions under contention](/assets/img/fig2_tile_latency.png)

- **Setup**
  - Compare:
    - 1CH + default L2 vs
    - 2CH + 4-bank L2 (tuned controller),
  - Both under **8 MiB** co-run stress.
  - GEMM split into **16 tiles (64×64)**; each tile’s cycles are logged.

- **Results**
  - **1CH + default L2**
    - Mean tile latency ≈ **40.5k cycles**, std-dev ≈ **3.8k cycles**.
    - First tile can hit **>50k cycles**; distribution is slow and jittery.
  - **2CH + 4-bank L2**
    - Mean tile latency ≈ **13.7k cycles**, std-dev ≈ **0.95k cycles**.
    - All tiles sit in a **tight band** around the mean.

- **Interpretation**
  - The tuned memory topology does more than increase average throughput:
    - It keeps Gemmini tile latency in a **narrow, predictable band**, even under strong co-run contention.
    - This directly matches my broader goal: **accelerators that keep their promises under shared-memory constraints**.

---

## Takeaways

- Co-run slowdowns are **structured, not random**:
  - DRAM saturation and L2 bank contention appear in predictable phases.
- Small, coordinated **HW/SW changes**—more DRAM channels, L2 banking, tuned controller, phase-aware scheduling—can:
  - Recover **up to 2.7×** throughput, and  
  - Dramatically tighten **latency variation** across tiles.
- This project generalizes the earlier SHA-3 and Gemmini work to **true heterogeneous, multi-workload settings**, pushing my research from “single-accelerator speedups” toward **system-level, memory-centric guarantees**.
