---
layout: page
title: Projects
permalink: /projects/
description: Independent Chipyard research line and selected systems projects.
nav: true
nav_order: 4
---

This page highlights my independent research line in **memory-centric architectures for accelerator-rich RISC-V systems** and a few related systems projects. For a full list of experience and coursework, see my [CV](/cv/).

---

## Chipyard research line (independent projects)

<div class="row">

  <div class="col-md-4 mb-4">
    <div class="card h-100">
      <img
        src="/assets/img/sha3_speedup_vs_size_with_arrow_colored.png"
        class="card-img-top"
        alt="SHA-3 speedup vs message size with multi-bank L2 improvement"
      >
      <div class="card-body">
        <h5 class="card-title">
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_sha3"
            target="_blank"
          >
            SHA3 Accelerator Performance Stabilization in Chipyard
          </a>
        </h5>
        <p class="card-text" style="font-size: 0.95rem;">
          Independent Chipyard project analyzing when SHA-3 accelerator speedups
          persist as message sizes scale. I integrated a SHA-3 RoCC with a Rocket core
          and built an automated benchmark pipeline for messages from 136&nbsp;B to
          544&nbsp;KB.
        </p>
        <ul style="font-size: 0.9rem;">
          <li>
            Cycle-accurate Verilator traces showed that speedup for large inputs
            shrank from 206× to 120× due to contention in a single-bank inclusive L2,
            not a saturated accelerator.
          </li>
          <li>
            Re-architected the L2 as a multi-banked inclusive cache, increasing
            sub-banking concurrency and effective capacity.
          </li>
          <li>
            At 557&nbsp;KB, recovered ≈34% throughput
            (speedup 120.27× → 160.90×), pulling large-input behavior back toward the
            ~170× plateau and showing that memory-path design, not the SHA-3 core,
            sets real-world speedup at scale.
          </li>
        </ul>
        <p class="mt-3 mb-0">
          <a
            href="/projects/chipyard_sha3"
            class="btn btn-sm btn-primary mr-2 mb-2"
          >
            <i class="ti ti-file-text mr-1"></i>
            Project details
          </a>
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_sha3"
            class="btn btn-sm btn-outline-secondary mb-2"
            target="_blank"
          >
            <i class="ti ti-brand-github mr-1"></i>
            GitHub
          </a>
        </p>
        <p class="card-text">
          <small class="text-muted">
            Branch: <code>chipyard_sha3</code> · RoCC, Verilator + DRAMSim2
          </small>
        </p>
      </div>
    </div>
  </div>

  <div class="col-md-4 mb-4">
    <div class="card h-100">
      <img
        src="/assets/img/boomgemmini_mac100_before_after.png"
        class="card-img-top"
        alt="Gemmini MAC/100 cycles before and after memory-pipeline optimization"
      >
      <div class="card-body">
        <h5 class="card-title">
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_gemmini"
            target="_blank"
          >
            Gemmini Offload Thresholds &amp; Memory-Centric Pipeline Co-Design
          </a>
        </h5>
        <p class="card-text" style="font-size: 0.95rem;">
          Independent Gemmini follow-up project testing whether matrix GEMM workloads
          share the memory-path limits seen in my SHA-3 accelerator project. I built a
          WS/OS benchmarking framework comparing CPU and Gemmini across matrix sizes
          and profiling offload thresholds.
        </p>
        <ul style="font-size: 0.9rem;">
          <li>
            Identified two regimes: launch and data-movement overheads bottlenecked
            small matrices; bandwidth and tiling limited large ones.
          </li>
          <li>
            Co-designed scratchpad and accumulator banking, system bus width, and
            Gemmini DMA bus-width alignment so that the missing speedup came from the
            memory path rather than the MAC array.
          </li>
          <li>
            Lowered offload thresholds and improved 1024³ GEMM throughput by
            ≈58%, validated on FireSim (AWS EC2 F1), showing that offloading is not
            “always good” and depends on memory hierarchy and tile shape.
          </li>
        </ul>
        <p class="mt-3 mb-0">
          <a
            href="/projects/chipyard_gemmini"
            class="btn btn-sm btn-primary mr-2 mb-2"
          >
            <i class="ti ti-file-text mr-1"></i>
            Project details
          </a>
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_gemmini"
            class="btn btn-sm btn-outline-secondary mb-2"
            target="_blank"
          >
            <i class="ti ti-brand-github mr-1"></i>
            GitHub
          </a>
        </p>
        <p class="card-text">
          <small class="text-muted">
            Branch: <code>chipyard_gemmini</code> · Gemmini, Verilator + FireSim
          </small>
        </p>
      </div>
    </div>
  </div>

  <div class="col-md-4 mb-4">
    <div class="card h-100">
      <img
        src="/assets/img/fig1_gemm_latency.png"
        class="card-img-top"
        alt="GEMM latency versus memory topology and co-run stress"
      >
      <div class="card-body">
        <h5 class="card-title">
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_hetero"
            target="_blank"
          >
            Heterogeneous SoC Memory Contention: Diagnosis &amp; Mitigation
          </a>
        </h5>
        <p class="card-text" style="font-size: 0.95rem;">
          Investigated how shared-memory contention shapes accelerator behavior when
          workloads co-run on a heterogeneous RISC-V SoC (BOOM, Rocket, Gemmini). I
          built a co-run profiling framework with a contention generator and
          cycle-accurate per-tile/per-kernel logging.
        </p>
        <ul style="font-size: 0.9rem;">
          <li>
            Sensitivity sweeps with BOOM&nbsp;+&nbsp;2×Rocket showed slowdowns from DRAM
            bandwidth saturation and L2 bank contention: a 256×256 GEMM slowed by
            ≈3× under 8&nbsp;MiB co-run stress with a single-channel, default L2.
          </li>
          <li>
            Introduced multi-channel DRAM, 4-bank L2, and tuned memory-controller
            parameters to pull GEMM latency back near the low-stress regime and
            tighten tile-latency distributions.
          </li>
          <li>
            Added lightweight phase-aware scheduling that staggers peaks across
            co-running workloads, raising aggregate throughput by up to 2.7× and
            showing that shared memory and scheduling, not raw compute, dominate
            behavior under contention.
          </li>
        </ul>
        <p class="mt-3 mb-0">
          <a
            href="/projects/chipyard_hetero"
            class="btn btn-sm btn-primary mr-2 mb-2"
          >
            <i class="ti ti-file-text mr-1"></i>
            Project details
          </a>
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_hetero"
            class="btn btn-sm btn-outline-secondary mb-2"
            target="_blank"
          >
            <i class="ti ti-brand-github mr-1"></i>
            GitHub
          </a>
        </p>
        <p class="card-text">
          <small class="text-muted">
            Branch: <code>chipyard_hetero</code> · BOOM + Rocket + Gemmini, Verilator
          </small>
        </p>
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
