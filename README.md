<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo_horizontal_dark.svg">
    <img src="assets/logo_horizontal_light.svg" alt="SAM: Systematic Agent for Meta-analysis" width="420">
  </picture>
</p>

<h1 align="center">SAM</h1>

<p align="center">
  <strong>Systematic Agent for Meta-analysis</strong>
</p>

<p align="center">
  <em>A multi-agent LLM system for automating systematic reviews and meta-analyses</em>
</p>

<p align="center">
  <img alt="Presented at CIPIE 2026" src="https://img.shields.io/badge/Presented_at-CIPIE_2026-1E6091.svg">
</p>

<p align="center">
  9 AI agents &middot; Multi-database search &middot; R/metafor statistics &middot; Manuscript generation &middot; Explainability report
</p>

> **Note:** The implementation of SAM is maintained in a private repository. This repository is a documentation-only overview of the system's architecture and results.

---

## What is SAM?

**SAM** (Systematic Agent for Meta-analysis) is a multi-agent pipeline that automates the full systematic review workflow, from protocol design to manuscript generation. Give it a clinical research question and it will search databases, screen papers, retrieve full texts, extract data, assess quality, run meta-analysis, and write a manuscript.

Developed as part of the **Trabajo de Fin de Grado (TFG)** at **Universidad Pontificia Comillas (ICAI)**, Engineering Mathematics (iMAT) program.

### Key Features

- **9 specialized agents** working sequentially (A1→A9)
- **4 search databases**: PubMed, Europe PMC, Semantic Scholar, Google Scholar
- **Multi-source full-text retrieval cascade** with graceful degradation to abstract-only screening
- **3 quality assessment tools**: RoB 2.0 (RCTs), ROBINS-I (observational), JBI (qualitative)
- **R/metafor statistical synthesis**: REML, Egger's test, trim-and-fill, Rosenthal's Fail-Safe N, leave-one-out, subgroups, meta-regression, GRADE; no LLM for statistics
- **RAG-based manuscript generation** with FAISS + MiniLM-L6-v2
- **Explainability & traceability** report (A9): per-paper Excel trace + explainability report in Spanish
- **Model-agnostic**: auto-detects chat templates, works with any HuggingFace model
- **Per-agent LLM parameters** (temperature, max tokens, repetition penalty)
- **991 automated tests** (879 passing; the rest skipped or xfail for deferred features / GPU-only paths) covering all 9 agents plus end-to-end pipeline

---

## Pipeline Overview

| Agent | Role | Method |
|-------|------|--------|
| **A1** | Protocol Design | PICO/SPIDER/PECO/PCC framework detection, criteria generation, PubMed query |
| **A2** | Literature Search | Multi-database search (PubMed, Europe PMC, Semantic Scholar, Google Scholar) + semantic deduplication |
| **A3** | Title/Abstract Screening | Ensemble voting (N votes/paper, configurable majority threshold) |
| **A4** | Full-Text Retrieval & Screening | Multi-source retrieval cascade + LLM full-text screening |
| **A5** | Data Extraction | 3 focused prompts per paper (study characteristics, intervention, outcomes) |
| **A6** | Quality Assessment | RoB 2.0 / ROBINS-I / JBI Critical Appraisal with weighted scores |
| **A7** | Statistical Synthesis | R/metafor: random-effects REML, heterogeneity, Egger's test, trim-and-fill, Rosenthal's Fail-Safe N, leave-one-out, subgroups, meta-regression, GRADE, R script export |
| **A8** | Manuscript Generation | RAG-based writing, section-specific retrieval, population descriptives, LaTeX/PDF output |
| **A9** | Explainability & Traceability | Per-paper Excel trace + explainability report in Spanish, no GPU |

### Design Principles

- **Single GPU**: Only one model loaded at a time, VRAM freed between agents
- **Conservative fallback**: Parse errors → EXCLUDE (A3/A4) or default quality weights (A6)
- **No LLM for statistics**: A7 uses R/metafor directly via rpy2
- **Model-agnostic**: Chat template auto-detection + token-level output slicing (works with Qwen, LLaMA, Mistral, GLM, etc.)

---

## Technologies

| Category | Stack |
|----------|-------|
| **LLM** | Qwen2.5-7B (default), any HuggingFace causal LM, 4/8-bit quantization (BitsAndBytes) |
| **Embeddings** | MiniLM-L6-v2 (sentence-transformers) |
| **RAG** | LangChain + FAISS |
| **Statistics** | R / metafor (via rpy2) |
| **Search APIs** | Biopython (Entrez), Europe PMC, Semantic Scholar, Google Scholar |
| **Full-text** | PMC, Unpaywall, OpenAlex, CrossRef, DOAJ, CORE |
| **PDF** | PyMuPDF, pdfplumber, pdfminer, pypdfium2 |
| **UI** | Streamlit, React + Vite |
| **Output** | LaTeX (pdflatex), CSV, Excel, RevMan XML |

---

## Limitations

- **Single-GPU constraint**: Only one LLM fits in VRAM at a time (8 GB). Ensemble votes are sequential with the same model. Multi-model ensemble supported but adds latency.
- **Full-text access**: Many articles are publisher-restricted. The multi-source cascade helps, but some papers still fall back to abstract-only.
- **Statistical synthesis**: Requires papers to report compatible quantitative outcomes. If extracted data lacks effect sizes/CIs, no meta-analysis is produced.
- **LLM accuracy**: A 7B model can generate incorrect extractions or quality judgments. Human validation is recommended.
- **Google Scholar**: Aggressive rate limiting and CAPTCHAs may block access after many requests.

---

## Authors

**Trabajo de Fin de Grado (TFG)**, Comillas Pontifical University, ICAI.
Presented as a poster at **CIPIE 2026** (II Congreso Internacional de
Psicología, Innovación Tecnológica y Emprendimiento), Madrid, July 2026.

- **Ignacio Queipo de Llano Pérez-Gascón**¹ *(corresponding author)*:
  [iqueipo.pg24@gmail.com](mailto:iqueipo.pg24@gmail.com)
- **Á. López-López**⁵
- **S. Lumbreras**⁵
- **P. Collazo-Castiñeira**³
- **M. Sánchez-Izquierdo**³
- **I. Echegoyen**³ˑ⁴
- **E. C. Garrido-Merchán**²ˑ⁵

**Affiliations** (all at Comillas Pontifical University, Madrid, Spain):
1. ICAI School of Engineering
2. Quantitative Methods Department
3. Psychology Department
4. Laboratory of Psychology
5. Institute for Research in Technology

### Acknowledgements
- Qwen Team (Alibaba) for Qwen2.5-7B
- Hugging Face for the Transformers ecosystem
- Wolfgang Viechtbauer for R/metafor
- Open-source communities: LangChain, FAISS, Streamlit, Biopython

---

## Citation

If you use SAM in your research, please cite it. Until the associated paper
is published, cite the software:

> Queipo de Llano Pérez-Gascón, I., López-López, Á., Lumbreras, S.,
> Collazo-Castiñeira, P., Sánchez-Izquierdo, M., Echegoyen, I., &
> Garrido-Merchán, E. C. (2026). *SAM: Systematic Agent for Meta-analysis,
> a multi-agent LLM system for automating systematic reviews and
> meta-analyses* [Software]. https://github.com/iqueipopg/SAM-overview

---

**Version:** v0.6.0. 9 agents (A1-A9), 991 automated tests (879 passing; the rest skipped or xfail for deferred features / GPU-only paths).

Copyright (c) 2026 Ignacio Queipo de Llano Perez-Gascon and the SAM authors.
