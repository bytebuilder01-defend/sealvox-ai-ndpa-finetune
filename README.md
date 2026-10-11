# SealVox AI — NDPA Compliance Mapping Engine
NAIC 2026 | Track A: Academia & Research | Problem Statement 3: Sectoral Fine-Tuning

## What this is
A fine-tuned version of N-ATLaS (Nigeria's sovereign LLM) trained to map
technical and business findings to specific NDPA 2023 provisions.

This is the regulatory mapping engine (Layer 2) of SealVox AI — a continuous
compliance monitoring platform for Nigerian startups.

## Architecture
- Layer 1 — Scan: Integrates with startup codebase/cloud infrastructure
- Layer 2 — Map (this repo):** Maps findings to NDPA provisions using fine-tuned N-ATLaS
- Layer 3 — Generate: Produces submission-ready compliance documents

## Fine-tuned model
huggingface.co/ByteBuilderNG/sealvox-ai-ndpa-lora-adapter

## Benchmark results
| Model | Correct section |
|---|---|
| Base N-ATLaS | 4.0% |
| Base N-ATLaS (instructed) | 7.3% |
| **SealVox fine-tuned** | **20.2%** |

## Team
Muhammad Muntaka Sarki Department of Cybersecurity, Khalifa Isyaku Rabiu University (KHAIRUN)
Ibrahim Abubakar Department of Computer Science Federal University Dutsin-ma.

## Submission
NITDA/NCAIR National AI Innovation Challenge 2026
