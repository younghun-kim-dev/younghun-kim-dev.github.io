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

I work on **memory-centric architectures for accelerator-rich systems**. Using Chipyard, Gemmini, and FireSim, I study when accelerator speedups survive real cache hierarchies, DRAM, and schedulers instead of collapsing under contention.

I am a **Co-Founder & Systems Architect at Berify, Inc.** (Bitcoin hardware-wallet startup) and earned a **B.S. in Civil & Environmental Engineering** from Yonsei University (CGPA 4.23/4.30, rank 1/46, summa cum laude). My civil-engineering training in flows and bottlenecks now shapes how I reason about data movement, memory hierarchies, and shared resources in heterogeneous SoCs.

---

## Research focus

- Data movement, memory hierarchy, and cache coherence under real workloads
- Shared-memory contention and scheduling on heterogeneous RISC-V SoCs
- Hardware–software co-design of accelerator interfaces, memory systems, and runtimes

For a deeper overview, see the [Research](/research/) page.

---

## Chipyard highlight projects

<div class="row">

  <!-- SHA3 card -->
  <div class="col-md-4 mb-4">
    <div class="card h-100 shadow-sm">
      <img
        src="/assets/img/sha3_speedup_vs_size_with_arrow_colored.png"
        class="card-img-top"
        alt="SHA-3 speedup vs message size with multi-bank L2 improvement"
      >
      <div class="card-body d-flex flex-column">
        <h5 class="card-title">
          <a href="/projects/chipyard_sha3/">
            SHA3 Accelerator Performance Stabilization
          </a>
        </h5>
        <p class="card-text" style="font-size: 0.9rem;">
          Independent Chipyard project on when SHA-3 accelerator speedups
          survive as message sizes scale on Rocket/BOOM SoCs by redesigning the
          L2 cache.
        </p>
        <ul class="mb-3" style="font-size: 0.85rem;">
          <li>Speedup collapse: 206× → ~120× at large inputs from single-bank L2.</li>
          <li>Multi-bank L2 recovers ≈34% throughput at 557&nbsp;KB.</li>
        </ul>
        <a
          href="/projects/chipyard_sha3/"
          class="btn btn-sm btn-outline-primary mt-auto"
        >
          Project details
        </a>
      </div>
    </div>
  </div>

  <!-- Gemmini card -->
  <div class="col-md-4 mb-4">
    <div class="card h-100 shadow-sm">
      <img
        src="/assets/img/boomgemmini_mac100_before_after.png"
        class="card-img-top"
        alt="Gemmini MAC/100 cycles before and after memory-pipeline optimization"
      >
      <div class="card-body d-flex flex-column">
        <h5 class="card-title">
          <a href="/projects/chipyard_gemmini/">
            Gemmini Offload Thresholds &amp; Pipeline Co-Design
          </a>
        </h5>
        <p class="card-text" style="font-size: 0.9rem;">
          Gemmini follow-up project on when GEMM should be offloaded vs left on
          the CPU, and how memory-path design shifts that boundary.
        </p>
        <ul class="mb-3" style="font-size: 0.85rem;">
          <li>WS/OS GEMM framework for sizes and tile shapes; offload thresholds.</li>
          <li>Memory-pipeline tuning improves 1024³ throughput by ≈58% on FireSim.</li>
        </ul>
        <a
          href="/projects/chipyard_gemmini/"
          class="btn btn-sm btn-outline-primary mt-auto"
        >
          Project details
        </a>
      </div>
    </div>
  </div>

  <!-- Hetero card -->
  <div class="col-md-4 mb-4">
    <div class="card h-100 shadow-sm">
      <img
        src="/assets/img/fig1_gemm_latency.png"
        class="card-img-top"
        alt="GEMM latency versus memory topology and stress"
      >
      <div class="card-body d-flex flex-column">
        <h5 class="card-title">
          <a href="/projects/chipyard_hetero/">
            Heterogeneous SoC Memory Contention
          </a>
        </h5>
        <p class="card-text" style="font-size: 0.9rem;">
          Co-run profiling on a BOOM + 2×Rocket + Gemmini SoC to see how shared
          memory and scheduling shape accelerator behavior under stress.
        </p>
        <ul class="mb-3" style="font-size: 0.85rem;">
          <li>Bandwidth stressors slow 256×256 GEMM by ≈3× under default DRAM/L2.</li>
          <li>Multi-channel DRAM, banked L2, and phase-aware scheduling lift throughput up to 2.7×.</li>
        </ul>
        <a
          href="/projects/chipyard_hetero/"
          class="btn btn-sm btn-outline-primary mt-auto"
        >
          Project details
        </a>
      </div>
    </div>
  </div>

</div>

---

## Background

- **Co-Founder & Systems Architect, Berify, Inc.** – secure device software, entropy pipeline design (external TRNG + NIST SP&nbsp;800-90B / Dieharder), and reproducible Buildroot-based OS images with CI.
- **B.S., Civil & Environmental Engineering, Yonsei University** – CGPA 4.23/4.30, major GPA 4.28/4.30, rank 1/46, summa cum laude.

For more detail, see my [full CV](/cv/).
