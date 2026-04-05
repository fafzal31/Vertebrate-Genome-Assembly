# 🧬 VGP Genome Assembly — Galaxy Bioinformatics Pipeline

> A step-by-step implementation of the **Vertebrate Genomes Project (VGP)** de-novo genome assembly pipeline using the Galaxy Workflow System, based on the [Galaxy Training Network tutorial](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/vgp_genome_assembly/tutorial.html).

---

## 📋 Table of Contents

- [Overview](#overview)
- [Background](#background)
- [Pipeline Architecture](#pipeline-architecture)
- [Technologies & Tools](#technologies--tools)
- [Workflow Stages](#workflow-stages)
- [Repository Structure](#repository-structure)
- [Outputs & Visualizations](#outputs--visualizations)
- [Key Results](#key-results)
- [How to Reproduce](#how-to-reproduce)
- [References](#references)

---

## Overview

This repository contains the **outputs, visualizations, and results** from completing the Galaxy Training Network's VGP Genome Assembly tutorial. The VGP (Vertebrate Genomes Project) pipeline is a fully modular, automated de-novo genome assembly workflow capable of producing high-quality, near-error-free, chromosome-level, haplotype-phased genome assemblies.

The pipeline was executed using the **Galaxy Workflow System (GWS)** on the public Galaxy instance, using PacBio HiFi reads combined with Hi-C chromatin conformation data.

---

## Background

The **Vertebrate Genomes Project (VGP)**, launched by the Genome 10K consortium (G10K), aims to generate high-quality reference genome assemblies for every vertebrate species on Earth. De-novo genome assembly — reconstructing the original DNA sequence from short fragment reads alone — is foundational to modern Molecular Biology and Evolutionary Biology research.

This tutorial follows **Analysis Trajectory B**, the most common assembly path: using HiFi reads alongside Hi-C data derived from the same individual, without Bionano optical map data.

---

## Pipeline Architecture

The VGP Galaxy pipeline is organized into **ten modular workflows**:

```
Raw Reads (HiFi + Hi-C)
        │
        ▼
┌─────────────────────┐
│  Workflow 1 & 2      │  ← K-mer Profiling (GenomeScope2 + Meryl)
│  Genome Profiling    │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Workflow 3 & 4      │  ← HiFi Assembly (hifiasm)
│  Contig Assembly     │     GFA → FASTA (gfastats)
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Workflow 5 & 6      │  ← Duplicate Purging (purge_dups)
│  Post-processing     │     (invoked only if QC indicates false duplicates)
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Workflow 7, 8, 9   │  ← Hybrid Scaffolding
│  Scaffolding         │     Hi-C (SALSA / YaHS)
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Final QC            │  ← gfastats + BUSCO + Merqury
│  Assembly Evaluation │
└─────────────────────┘
```

---

## Technologies & Tools

| Tool | Purpose |
|------|---------|
| **PacBio HiFi Reads** | Primary long-read sequencing input (~10–20 kbp reads, high accuracy) |
| **Hi-C Data** | Chromatin conformation data for scaffolding and phasing |
| **Meryl** | K-mer counting and database construction |
| **GenomeScope2** | Reference-free genome profiling (size, heterozygosity, ploidy) |
| **hifiasm** | HiFi read assembly into haplotype-phased contig graphs (GFA format) |
| **gfastats** | GFA → FASTA conversion and assembly statistics (N50, contig count, etc.) |
| **purge_dups** | Detection and removal of false duplicates / haplotigs |
| **SALSA / YaHS** | Hi-C-based scaffolding |
| **BUSCO** | Assembly completeness evaluation against conserved gene sets |
| **Merqury** | Reference-free quality value (QV) assessment using k-mers |
| **Galaxy Workflow System** | Workflow orchestration, reproducibility, and parameter tracking |

---

## Workflow Stages

### Stage 1 — Genome Profile Analysis
K-mer profiles are generated from raw HiFi reads using **Meryl** at k=31. The resulting histograms are analyzed by **GenomeScope2** to estimate:
- Haploid genome size
- Heterozygosity rate
- Genome repetitiveness
- Sequencing error rate

> The k-mer profile follows a bimodal distribution, characteristic of a diploid genome. GenomeScope2 estimated a haploid genome size of ~11.7 Mb with ~0.576% sequence variation, consistent with the *Saccharomyces* genome used as the tutorial dataset.

---

### Stage 2 — HiFi Contig Assembly
HiFi reads are assembled into fully phased contig graphs using **hifiasm** (Hi-C mode), yielding separate **Hap1** and **Hap2** contig graphs in GFA format. These are converted to FASTA using `gfastats`, which also generates standard assembly statistics:
- Total number of contigs
- Largest contig length
- N50 / NG50

---

### Stage 3 — Post-Assembly Processing
**purge_dups** is used to identify and remove heterozygous duplications and false duplicates from the primary assembly, using coverage cutoffs derived from the GenomeScope2 output. Haplotigs are retained in the alternate assembly.

---

### Stage 4 — Hybrid Scaffolding (Hi-C)
Hi-C chromatin interaction data is used to scaffold contigs into chromosome-level sequences. The Hi-C contact maps provide long-range linking information that bridges contig gaps, generating scaffolds with unknown sequence gaps represented by N-runs.

---

### Stage 5 — Assembly Quality Control
Final assemblies are evaluated using a multi-pronged QC strategy:
- **gfastats** — Structural statistics (contig count, scaffold count, N50, gap count)
- **BUSCO** — Gene-space completeness using conserved single-copy orthologs
- **Merqury** — K-mer based Quality Value (QV) score and completeness without a reference

---





## Outputs & Visualizations

### GenomeScope2 — K-mer Profile
The GenomeScope2 linear and log-scale plots show the observed k-mer profile, fitted models, and estimated genome parameters. The bimodal distribution confirms a diploid genome model fit > 93%.

### Hi-C Contact Maps
Heatmaps generated from Hi-C interaction data visualize the chromosome-level organization of the scaffolded assemblies. Well-defined blocks along the diagonal indicate accurate scaffolding.

### BUSCO Completeness
BUSCO results quantify the proportion of conserved single-copy orthologs that are:
- **Complete (single-copy)**
- **Complete (duplicated)**
- **Fragmented**
- **Missing**

### Merqury QV Scores
Quality Value scores computed using k-mer completeness provide a reference-free estimate of base-level accuracy in the final assemblies.

---

## Key Results

| Metric | Hap1 | Hap2 |
|--------|------|------|
| Estimated Genome Size | ~11.7 Mb | ~11.7 Mb |
| Heterozygosity | ~0.576% | ~0.576% |
| Assembly Tool | hifiasm (Hi-C mode) | hifiasm (Hi-C mode) |
| Scaffolding | Hi-C | Hi-C |
| BUSCO Completeness | (see `/outputs/qc/`) | (see `/outputs/qc/`) |
| Merqury QV | (see `/outputs/qc/`) | (see `/outputs/qc/`) |

> ⚠️ **Note:** Exact numeric results may vary slightly depending on the version of tools used, as algorithms can change between software releases. Results shown in this repository reflect the versions available at the time of tutorial completion.

---

## How to Reproduce

### Prerequisites
- A Galaxy account on [usegalaxy.eu](https://usegalaxy.eu), [usegalaxy.org](https://usegalaxy.org), or [usegalaxy.org.au](https://usegalaxy.org.au)
- Familiarity with Galaxy (importing datasets, running workflows, managing histories)

### Steps

1. **Import input datasets from Zenodo**
   Upload the FastQ/SANGER.gz HiFi and Hi-C datasets from the Zenodo record linked in the tutorial into a Galaxy history.

2. **Import VGP workflows**
   Navigate to **Workflow → Import** in Galaxy and import the VGP workflows from WorkflowHub or Dockstore. Search for `name:vgp` in the TRS Server: `workflowhub.eu`.

3. **Run the workflows in order**
   Execute each workflow sequentially (Workflows 1 → 3/4 → 5/6 if needed → 7/8/9), passing outputs from each stage as inputs to the next.

4. **Evaluate results**
   Inspect QC outputs (GenomeScope2 plots, BUSCO reports, Merqury QV scores) after each major stage.

### Reference Tutorial
📖 [GTN Tutorial — VGP Genome Assembly (Step by Step)](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/vgp_genome_assembly/tutorial.html)

---

## References

- **Rhie et al. (2021)** — Towards complete and error-free genome assemblies of all vertebrate species. *Nature*, 592, 737–746.
- **Nurk et al. (2022)** — The complete sequence of a human genome. *Science*, 376, 44–53.
- **Hiltemann et al. (2023)** — Galaxy Training: A Powerful Framework for Teaching! *PLOS Computational Biology*, 10.1371/journal.pcbi.1010752.
- **Batut et al. (2018)** — Community-Driven Data Analysis Training for Biology. *Cell Systems*, 10.1016/j.cels.2018.05.012.
- [Galaxy Training Network (GTN)](https://training.galaxyproject.org/)
- [Vertebrate Genomes Project](https://vertebrategenomesproject.org/)
- [VGP Workflows on Galaxy Community Hub](https://galaxyproject.org/projects/vgp/workflows/)

---

<p align="center">
  Made with ❤️ using the <a href="https://galaxyproject.org">Galaxy Project</a> &nbsp;|&nbsp;
  Tutorial by <a href="https://training.galaxyproject.org">Galaxy Training Network</a>
</p>
