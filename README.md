# SAR TOPS Mode Simulator

Simulation and derivation workspace for TOPS (Terrain Observation with Progressive Scans) SAR azimuth processing, with emphasis on folding, UFR, deramping, azimuth compression, and the physical meaning of each stage.

## Table of Contents

- [Flowchart](#flowchart)
- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Documentation Standard](#documentation-standard)
- [Project Status](#project-status)

## Flowchart

### Processing Flow Chart

```mermaid
graph TD
    A[Raw Data] --> B[Range Compression]
    B --> C1

    subgraph FREQ[Frequency UFR]
        C1[Mosaicking] --> C2[Deramping]
        C2 --> C3[Low-Pass Filter]
        C3 --> C4[Reramping]
    end

    C4 --> D[Azimuth Compression]
    D --> E1

    subgraph TIME[Time UFR]
        E1[Mosaicking] --> E2[Deramping]
        E2 --> E3[Low-Pass Filter]
        E3 --> E4[Reramping]
    end

    E4 --> F[Focused Image]

    classDef main fill:#0f172a,stroke:#38bdf8,stroke-width:2px,color:#e2e8f0;
    classDef freq fill:#1e293b,stroke:#f59e0b,stroke-width:2px,color:#f8fafc;
    classDef time fill:#1f2937,stroke:#a78bfa,stroke-width:2px,color:#f8fafc;

    class A,B,D,F main;
    class C1,C2,C3,C4,C5 freq;
    class E1,E2,E3,E4 time;
```

- [Raw Data](derive/derive_main.md#1-raw-data)
- [Range Compression](derive/derive_main.md#2-range-compression)
- [Azimuth Frequency Unfolding And Resampling (UFR)](derive/derive_main.md#3-azimuth-frequency-unfolding-and-resampling-ufr)
  - [Azimuth Frequency Folding (Explain)](derive/derive_main.md#31-azimuth-frequency-folding-explain)
  - [Mosaicking](derive/derive_main.md#32-mosaicking)
  - [Deramping](derive/derive_main.md#33-deramping)
  - [Low Pass Filter](derive/derive_main.md#34-low-pass-filter)
  - [Reramping](derive/derive_main.md#35-reramping)
- [Azimuth Compression](derive/derive_main.md#4-azimuth-compression)
- [Azimuth Time Unfolding And Resampling (UFR)](derive/derive_main.md#5-azimuth-time-unfolding-and-resampling-ufr)
  - [Azimuth Time Folding (Explain)](derive/derive_main.md#51-azimuth-time-folding-explain)
  - [Mosaicking](derive/derive_main.md#52-mosaicking)
  - [Deramping](derive/derive_main.md#53-deramping)
  - [Low Pass Filter](derive/derive_main.md#54-low-pass-filter)
  - [Reramping](derive/derive_main.md#55-reramping)
- [Focused Image](derive/derive_main.md#6-focused-image)

## Overview

This repository is not just an implementation sandbox. It is a combined engineering and derivation workspace for understanding how TOPS azimuth processing evolves from raw phase history to focused output, and why additional unfolding steps are required in both frequency and time domains.

The documentation is organized so that a reader can move between:

- high-level processing flow
- stage-by-stage mathematical derivations
- teaching-style notebooks that visualize intermediate results
- repository-local writing rules for math-heavy technical notes

## Repository Structure

```text
sar_tops_mode/
├── README.md
├── derive/
│   ├── derive_main.md
│   ├── range_compression.md
│   ├── azimuth_compression.md
│   ├── azimuth_deramp_LPF.md
│   ├── azimuth_freq_folding.md
│   ├── azimuth_freq_ufr.md
│   ├── azimuth_time_folding.md
│   ├── azimuth_time_ufr.md
│   ├── freq_time_deramping.md
│   ├── explain_ufr3.ipynb
│   ├── explain_ufr4.ipynb
│   └── figures/
└── .codex/
    └── skills/
        └── github-readme-math-physics-derivation/
```

## Documentation Standard

This repository includes a local derivation-writing skill:

- [github-readme-math-physics-derivation](.codex/skills/github-readme-math-physics-derivation/SKILL.md)

That skill defines the documentation standard used in this project, including:

- GitHub-safe math formatting
- fully expanded closed-form expressions at every major stage
- stage-local signal expressions instead of nested operator chains
- notebook rhythm for teaching-style `.ipynb` documents
- explicit physical interpretation next to each mathematical step

Practical README rule:

- single-document tables of contents belong inside the corresponding derivation note
- this `README.md` should stay at repository scope and act as the project homepage, not as the local TOC for one derivation file

## Project Status

Current repository state:

- derivation-first documentation is present for the main TOPS azimuth chain
- UFR3 and UFR4 now have teaching-style notebooks
- the documentation standard is being consolidated around one repository-local derivation skill
