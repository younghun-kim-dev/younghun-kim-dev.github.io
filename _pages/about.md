---
layout: about
title: "Younghun Kim"
permalink: /
subtitle: "Memory-Centric Architectures for Accelerator-Rich Systems"

profile:
  align: right
  image: younghun.jpg
  image_circular: false
  more_info: >
    <p>Seoul, Republic of Korea</p>
    <p><a href="mailto:younghun0613@yonsei.ac.kr">younghun0613@yonsei.ac.kr</a></p>
    <p><a href="https://github.com/younghun-kim-dev" target="_blank">github.com/younghun-kim-dev</a></p>

selected_papers: false
social: true

announcements:
  enabled: false
  scrollable: true
  limit: 5

latest_posts:
  enabled: false
  scrollable: true
  limit: 3
---

I work on **memory-centric architectures for accelerator-rich systems**. Using
Chipyard, Gemmini, and FireSim, I study when accelerator speedups survive real
cache hierarchies, DRAM, and schedulers instead of collapsing under contention.

I am a **Co-Founder & Systems Architect at Berify, Inc.** (Bitcoin
hardware-wallet startup) and earned a **B.S. in Civil & Environmental
Engineering** from Yonsei University (CGPA 4.23/4.30, rank 1/46, summa cum
laude). My civil-engineering training in flows and bottlenecks now shapes how I
reason about data movement, memory hierarchies, and shared resources in
heterogeneous SoCs.

---

## Research focus

- Data movement, memory hierarchy, and cache coherence under real workloads
- Shared-memory contention and scheduling on heterogeneous RISC-V SoCs
- Hardware–software co-design of accelerator interfaces, memory systems, and runtimes

---

## Chipyard highlight projects

<div class="row">

  <div class="col-md-4 mb-4">
    <div class="card h-100 shadow-sm">
      <img
        src="/assets/img/sha3_speedup_vs_size_with_arrow_colored.png"
        class="card-img-top"
        alt="SHA-3 speedup vs message size with multi-bank L2 improvement"
      >
      <div class="card-body d-flex flex-column">
        <h5 class="card-title">
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_sha3"
            target="_blank"
          >
            SHA3 Accelerator Performance Stabilization
          </a>
        </h5>
        <p class="card-text" style="font-size: 0.9rem;">
          Independent project on when SHA-3 accelerator speedups persist as
          message sizes scale on Rocket/BOOM SoCs.
        </p>
        <ul style="font-size: 0.85rem; padding-left: 1.2rem;">
          <li>
            Single-bank inclusive L2 holds mid-sized speedup near ~170× but
            collapses toward 120× at 557&nbsp;KB.
          </li>
          <li>
            Multi-bank inclusive L2 increases capacity and sub-banking
            concurrency, recovering ≈34% throughput at large inputs.
          </li>
          <li>
            Restores large-input behavior toward the ~170× plateau, showing that
            memory-path design, not the SHA-3 core, sets real-world speedup.
          </li>
        </ul>
        <p class="mt-auto mb-0" style="font-size: 0.8rem; color: #6c757d;">
          Branch: <code>chipyard_sha3</code> · RoCC, Verilator + DRAMSim2
        </p>
      </div>
    </div>
  </div>

  <div class="col-md-4 mb-4">
    <div class="card h-100 shadow-sm">
      <img
        src="/assets/img/boomgemmini_mac100_before_after.png"
        class="card-img-top"
        alt="Gemmini MAC/100 cycles before and after memory-pipeline optimization"
      >
      <div class="card-body d-flex flex-column">
        <h5 class="card-title">
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_gemmini"
            target="_blank"
          >
            Gemmini Offload Thresholds &amp; Pipeline Co-Design
          </a>
        </h5>
        <p class="card-text" style="font-size: 0.9rem;">
          Gemmini follow-up project testing when GEMM should be offloaded vs
          left on the CPU, and how memory-path design shifts that boundary.
        </p>
        <ul style="font-size: 0.85rem; padding-left: 1.2rem;">
          <li>
            WS/OS GEMM framework comparing CPU and Gemmini across matrix sizes
            and tile shapes; profiles offload thresholds.
          </li>
          <li>
            Identifies overhead-dominated vs bandwidth-limited regimes and shows
            that many “underperforming” cases are memory-path issues.
          </li>
          <li>
            Co-design of SPM/ACC banking, system bus width, and DMA alignment
            lowers offload thresholds and improves 1024³ throughput by ≈58%
            (validated on FireSim).
          </li>
        </ul>
        <p class="mt-auto mb-0" style="font-size: 0.8rem; color: #6c757d;">
          Branch: <code>chipyard_gemmini</code> · Gemmini, Verilator + FireSim
        </p>
      </div>
    </div>
  </div>

  <div class="col-md-4 mb-4">
    <div class="card h-100 shadow-sm">
      <img
        src="/assets/img/fig1_gemm_latency.png"
        class="card-img-top"
        alt="GEMM latency versus memory topology and stress"
      >
      <div class="card-body d-flex flex-column">
        <h5 class="card-title">
          <a
            href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_hetero"
            target="_blank"
          >
            Heterogeneous SoC Memory Contention
          </a>
        </h5>
        <p class="card-text" style="font-size: 0.9rem;">
          Co-run profiling on a BOOM + 2×Rocket + Gemmini SoC to see how shared
          memory and scheduling shape accelerator behavior under stress.
        </p>
        <ul style="font-size: 0.85rem; padding-left: 1.2rem;">
          <li>
            Bandwidth stressors on two Rockets slow a 256×256 Gemmini GEMM by
            ≈3× under 8&nbsp;MiB co-run stress with 1&nbsp;DRAM channel and default L2.
          </li>
          <li>
            Adding 2 DRAM channels, 4 L2 banks, and tuned memory-controller
            parameters recovers ≈2.7× throughput and tightens tile-latency bands.
          </li>
          <li>
            Lightweight phase-aware scheduling staggers contention peaks,
            improving predictability and showing that shared memory and runtime
            policy, not raw compute, dominate under co-run workloads.
          </li>
        </ul>
        <p class="mt-auto mb-0" style="font-size: 0.8rem; color: #6c757d;">
          Branch: <code>chipyard_hetero</code> · BOOM + Rocket + Gemmini, Verilator
        </p>
      </div>
    </div>
  </div>

</div>

---

## Background

- **Co-Founder & Systems Architect, Berify, Inc.** – secure device software,
  entropy pipeline design (external TRNG + NIST SP 800-90B / Dieharder), and
  reproducible Buildroot-based OS images with CI.
- **B.S., Civil & Environmental Engineering, Yonsei University** – CGPA
  4.23/4.30, major GPA 4.28/4.30, rank 1/46, summa cum laude.

For more detail, see my [full CV](/cv/).
