---
layout: project
title: "SHA3 Accelerator Performance Stabilization in Chipyard"
permalink: /projects/chipyard_sha3
description: "When does a SHA-3 accelerator stay ~170× faster, and when does single-bank L2 quietly turn it into a 120× accelerator?"
category: research
importance: 3
image: /assets/img/sha3_speedup_vs_size_with_arrow_colored.png
links:
  - name: GitHub branch
    url: https://github.com/younghun-kim-dev/chipyard/tree/chipyard_sha3
---

Independent Chipyard project analyzing **when SHA-3 accelerator speedups survive real memory hierarchies** instead of collapsing at scale. I integrated a SHA-3 RoCC with Rocket/BOOM SoCs, swept message sizes from **136 B to 557 KB** under Verilator + DRAMSim, and showed that **single-bank inclusive L2**, not the accelerator itself, quietly capped large-input speedup near **120×**. A redesigned **multi-bank inclusive L2** recovered ≈**34%** throughput at large inputs and pulled speedup back toward the **~170× plateau**.

---

## Key questions

- How does SHA-3 accelerator speedup change as messages grow from cache-friendly to L2-stressing sizes?
- When speedup collapses, is the bottleneck **compute**, **L1/TLB**, or **L2 banking / memory path**?
- Can we recover large-input speedup **purely by changing the L2 structure**, keeping the SHA-3 core fixed?

---

## Approach

- **RoCC integration in Chipyard**
  - Integrated a **SHA-3 Rocket Custom Coprocessor (RoCC)** into Rocket and BOOM SoCs.
  - Fixed BOOM RoCC/FPU tie-offs so non-FPU accelerators (SHA3) integrate cleanly into the OOO pipeline.

- **Memory-hierarchy experiment matrix**
  - Single-bank vs **multi-bank inclusive L2** (1 / 2 / 4 / 8 banks).
  - Rocket-only vs Rocket+BOOM SoCs, SHA3 vs SW-only baselines.
  - Optional sweeps for L1 size, TLB ways, and L2 concurrency to isolate the dominant bottleneck.

- **Automated benchmark pipeline**
  - Verilator + DRAMSim2 with large timeouts to support up to **557 KB** messages.
  - Scripts that:
    - Run **HW (sha3-rocc)** and **SW (sha3-sw)** binaries per size.
    - Parse UART logs for cycle counts.
    - Emit CSVs and an aggregated `combined_sha3_speed.csv` with `size, hw_cycles, sw_cycles, speedup`.

---

## Selected results

![SHA3 speedup vs message size with multi-bank L2 improvement](/assets/img/sha3_speedup_vs_size_with_arrow_colored.png)

- **Stable mid-size plateau**
  - For **≈2–70 KB** messages, speedup stays around **168–176×**.
  - SHA-3 RoCC is well-fed; the memory path is not yet the limiter.

- **Large inputs expose L2 banking**
  - At **139–278 KB**, speedup decays from ~160× to ~145× as the working set stresses L2 capacity/associativity.
  - At **557,056 B**, speedup collapses to **120.27×** even though the SHA-3 core is unchanged.
  - Cycle-accurate traces show **single-bank inclusive L2** serializing miss handling, refills, and evictions.

- **Multi-bank L2 recovery**
  - Replaced the single-bank inclusive L2 with a **multi-bank inclusive L2** (more capacity + sub-banking concurrency).
  - At **557,056 B**, speedup improves from **120.27× → 160.90×**, a ≈**34%** throughput gain, nearly matching the mid-size plateau.
  - The same accelerator looks like a **120×** unit under a narrow L2, and like a **161×** unit once the memory path is fixed.

---

## Takeaways

- At scale, **accelerator speedup is set by the memory path**, not the SHA-3 core:
  - Single-bank inclusive L2 quietly erodes speedups as messages outgrow cache-friendly sizes.
  - Multi-bank L2 restores the promised behavior without touching the accelerator.

- This project is the first step in my broader agenda:
  - Treat **memory-system design and banking** as first-class levers for **“accelerators that keep their promises”** under realistic working sets.
