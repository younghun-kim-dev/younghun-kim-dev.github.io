---
layout: page
title: Projects
permalink: /projects/
description: Independent Chipyard research line and selected systems projects.
nav: true
nav_order: 3
---

This page highlights my independent research line in **memory-centric architectures for accelerator-rich RISC-V systems** and a few related systems projects. For a full list of experience and coursework, see my [CV](/cv/).

---

## Chipyard research line (independent projects)

<div class="row">

  <!-- SHA3 project card -->
  <div class="col-md-4 mb-4">
    <div class="card h-100">
      <img
        src="/assets/img/sha3_speedup_vs_size_with_arrow_colored.png"
        class="card-img-top"
        alt="SHA-3 speedup vs message size with multi-bank L2 improvement"
      >
      <div class="card-body d-flex flex-column">
        <h5 class="card-title">
          SHA3 Accelerator Performance Stabilization in Chipyard
        </h5>
        <p class="card-text" style="font-size: 0.95rem;">
          Independent Chipyard project studying when SHA-3 accelerator speedups
          survive real memory hierarchies instead of collapsing at large message
          sizes. Integrated a SHA-3 RoCC with Rocket/BOOM SoCs and built an
          automated benchmark pipeline sweeping 136&nbsp;B–557&nbsp;KB.
        </p>
        <p class="card-text" style="font-size: 0.9rem;">
          Cycle-accurate profiling showed that a single-bank inclusive L2
          quietly pulls large-input speedup down toward ~120×. Redesigning the
          inclusive L2 as a multi-bank cache recovers ≈34% throughput at
          557&nbsp;KB and restores behavior toward the ~170× plateau, proving
          that memory-path design, not the SHA-3 core, sets real-world speedup.
        </p>
        <div class="mt-auto">
          <a
            href="{{ '/projects/chipyard_sha3' | relative_url }}"
            class="btn btn-sm btn-primary"
          >
            Project details
          </a>
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_sha3"
            target="_blank"
            class="btn btn-sm btn-outline-secondary"
          >
            GitHub branch
          </a>
          <p class="card-text mt-2">
            <small class="text-muted">
              Branch: <code>chipyard_sha3</code> · RoCC, Verilator + DRAMSim2
            </small>
          </p>
        </div>
      </div>
    </div>
  </div>

  <!-- Gemmini project card -->
  <div class="col-md-4 mb-4">
    <div class="card h-100">
      <img
        src="/assets/img/boomgemmini_mac100_before_after.png"
        class="card-img-top"
        alt="Gemmini MAC/100 cycles before and after memory-pipeline optimization"
      >
      <div class="card-body d-flex flex-column">
        <h5 class="card-title">
          Gemmini Offload Thresholds &amp; Memory-Centric Pipeline Co-Design
        </h5>
        <p class="card-text" style="font-size: 0.95rem;">
          Follow-up Gemmini project testing whether GEMM workloads share the
          memory-path limits seen in SHA-3. Built a WS/OS benchmarking framework
          to compare CPU and Gemmini across matrix sizes and profile offload
          thresholds.
        </p>
        <p class="card-text" style="font-size: 0.9rem;">
          Identified overhead-dominated and bandwidth-limited regimes, then
          co-designed scratchpad/accumulator banking, system-bus width, and
          Gemmini DMA alignment. The optimized memory pipeline improves
          1024³ throughput by ≈58% and shifts the CPU↔Gemmini offload threshold
          K*—showing that offloading is not “always good” and depends on tile
          shape and memory hierarchy.
        </p>
        <div class="mt-auto">
          <a
            href="{{ '/projects/chipyard_gemmini' | relative_url }}"
            class="btn btn-sm btn-primary"
          >
            Project details
          </a>
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_gemmini"
            target="_blank"
            class="btn btn-sm btn-outline-secondary"
          >
            GitHub branch
          </a>
          <p class="card-text mt-2">
            <small class="text-muted">
              Branch: <code>chipyard_gemmini</code> · Gemmini, Verilator + FireSim
            </small>
          </p>
        </div>
      </div>
    </div>
  </div>

  <!-- Heterogeneous SoC project card -->
  <div class="col-md-4 mb-4">
    <div class="card h-100">
      <img
        src="/assets/img/fig1_gemm_latency.png"
        class="card-img-top"
        alt="GEMM latency versus memory topology and co-run stress"
      >
      <div class="card-body d-flex flex-column">
        <h5 class="card-title">
          Heterogeneous SoC Memory Contention: Diagnosis &amp; Mitigation
        </h5>
        <p class="card-text" style="font-size: 0.95rem;">
          Heterogeneous RISC-V SoC study (BOOM, 2×Rocket, Gemmini) on how
          shared-memory contention shapes accelerator behavior when workloads
          co-run on the same DRAM and L2.
        </p>
        <p class="card-text" style="font-size: 0.9rem;">
          Built a co-run profiling framework with a bandwidth stress generator
          and per-tile logging. DRAM saturation and L2 bank contention slowed a
          256×256 GEMM by ≈3× under 8&nbsp;MiB stress. Adding 2 DRAM channels,
          4 L2 banks, tuned controller settings, and phase-aware scheduling
          recovered up to 2.7× throughput and tightened tile-latency bands,
          showing that shared memory and scheduling, not raw compute, dominate
          behavior under contention.
        </p>
        <div class="mt-auto">
          <a
            href="{{ '/projects/chipyard_hetero' | relative_url }}"
            class="btn btn-sm btn-primary"
          >
            Project details
          </a>
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_hetero"
            target="_blank"
            class="btn btn-sm btn-outline-secondary"
          >
            GitHub branch
          </a>
          <p class="card-text mt-2">
            <small class="text-muted">
              Branch: <code>chipyard_hetero</code> · BOOM + Rocket + Gemmini, Verilator
            </small>
          </p>
        </div>
      </div>
    </div>
  </div>

</div>

---

## Other systems & research projects

### Hardware-Accelerated AES on an FPGA-Based RISC-V SoC

On a LiteX/VexRiscv-based SoC, I engineered a custom AES instruction and enabled OpenSSL via a Buildroot patch, creating a Linux-on-FPGA environment for hardware–software co-design.

- Implemented AES as a custom VexRiscv instruction and integrated it into the SoC datapath.
- Analyzed cache behavior across message sizes and applied cache-line alignment and buffering to keep the hardware AES path busy.
- Achieved ~4× throughput over software-only OpenSSL, again confirming that data layout and memory behavior are first-class design levers.

---

### Berify, Inc. – Secure Systems & Entropy Pipeline

As **Co-Founder & Systems Architect** of Berify, Inc. (Bitcoin hardware wallet), I built end-to-end secure device and infrastructure workflows:

- Designed a hardened entropy pipeline moving from software multi-source and SoC HWRNG to an external TRNG, reaching ≈350&nbsp;kbit/s (≈140× previous throughput) and validating output via automated NIST SP&nbsp;800-90B and Dieharder testing.
- Developed a reproducible Buildroot-based OS build system (Docker external tree) with CI for image builds and SHA-256 hash verification, plus an emulator-first integration workflow.
- Systematized an air-gapped release and installation process with offline SHA-256/GPG verification and USB-OTG/UART flashing checklists to keep releases audit-ready.

---

### Earlier research (Civil & Environmental Engineering)

Before moving into computer architecture, I worked on computational methods in hydrodynamics and structural engineering at Yonsei University:

- Built a MATLAB pipeline for surface-velocity-only discharge estimation using the Maximum Entropy Method and validated it against published Shimada Bridge STIV+MEM results and KICT Andong field data.
- Computed and validated lateral velocity distributions on a Han River cross-section using a depth-integrated, RANS-based lateral distribution method.
- Optimized a high-rise piled raft foundation design for a 50-story tower (56→20 piles, ~67% reduction in concrete volume), presented at the 8th Mooyoung CM National Undergraduate Competition.

---

For detailed timelines and additional smaller projects, please see my [CV](/cv/) or my code on
<a href="https://github.com/younghun-kim-dev" target="_blank">GitHub</a>.
