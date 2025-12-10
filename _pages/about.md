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

<div class="project-cards">

  <!-- SHA3 project -->
  <div class="card mb-4 border-0 shadow-sm">
    <div class="row no-gutters align-items-center">
      <div class="col-md-5">
        <img
          src="/assets/img/sha3_speedup_vs_size_with_arrow_colored.png"
          alt="SHA-3 speedup vs message size with multi-bank L2 improvement"
          class="img-fluid rounded"
        />
      </div>
      <div class="col-md-7">
        <div class="card-body">
          <h5 class="card-title mb-1">
            <a
              href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_sha3"
              target="_blank"
            >
              SHA3 Accelerator Performance Stabilization
            </a>
          </h5>
          <p class="card-text" style="font-size: 0.95rem;">
            Independent project on when SHA-3 accelerator speedups persist as
            message sizes scale. A single-bank inclusive L2 drops speedup from
            206× to ~120× at 557&nbsp;KB; a multi-bank inclusive L2 recovers
            ≈34% throughput and restores large-input behavior toward the
            ~170× plateau.
          </p>
        </div>
      </div>
    </div>
  </div>

  <!-- Gemmini project -->
  <div class="card mb-4 border-0 shadow-sm">
    <div class="row no-gutters align-items-center">
      <div class="col-md-5">
        <img
          src="/assets/img/boomgemmini_mac100_before_after.png"
          alt="Gemmini MAC/100 cycles before and after memory-pipeline optimization"
          class="img-fluid rounded"
        />
      </div>
      <div class="col-md-7">
        <div class="card-body">
          <h5 class="card-title mb-1">
            <a
              href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_gemmini"
              target="_blank"
            >
              Gemmini Offload Thresholds &amp; Memory-Centric Pipeline
              Co-Design
            </a>
          </h5>
          <p class="card-text" style="font-size: 0.95rem;">
            Gemmini follow-up project testing when GEMM should be offloaded vs
            left on the CPU, and how memory-path design (SPM/ACC banking, bus
            width, DMA alignment) shifts that boundary. WS/OS GEMM sweeps and
            offload-threshold analysis improve 1024³ throughput by ≈58%
            (FireSim).
          </p>
        </div>
      </div>
    </div>
  </div>

  <!-- Heterogeneous SoC project -->
  <div class="card mb-4 border-0 shadow-sm">
    <div class="row no-gutters align-items-center">
      <div class="col-md-5">
        <img
          src="/assets/img/fig1_gemm_latency.png"
          alt="GEMM latency versus memory topology and stress"
          class="img-fluid rounded"
        />
      </div>
      <div class="col-md-7">
        <div class="card-body">
          <h5 class="card-title mb-1">
            <a
              href="https://github.com/younghun-kim-dev/chipyard/tree/chipyard_hetero"
              target="_blank"
            >
              Heterogeneous SoC Memory Contention
            </a>
          </h5>
          <p class="card-text" style="font-size: 0.95rem;">
            Co-run profiling on a BOOM + 2×Rocket + Gemmini SoC with a
            bandwidth stress generator and per-tile logging. Under 8&nbsp;MiB
            co-run stress, DRAM saturation and L2 bank contention slow a
            256×256 GEMM by ~3×; multi-channel DRAM, 4-bank L2, and
            phase-aware scheduling recover up to 2.7× throughput and tighten
            tile-latency bands.
          </p>
        </div>
      </div>
    </div>
  </div>

</div>

---

## Background

- **Co-Founder & Systems Architect, Berify, Inc.** – secure device software, entropy pipeline design (external TRNG + NIST SP&nbsp;800-90B / Dieharder), and reproducible Buildroot-based OS images with CI.
- **B.S., Civil & Environmental Engineering, Yonsei University** – CGPA 4.23/4.30, major GPA 4.28/4.30, rank 1/46, summa cum laude.

For more detail, see my [full CV](/cv/).
