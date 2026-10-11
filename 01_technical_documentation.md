# SealVox AI — Technical Documentation
**NAIC 2026 | Track A: Academia & Research | Problem Statement 3: Sectoral Fine-Tuning**

---

## 1. Problem Statement

Every Nigerian startup that collects, stores, or processes user data is legally obligated to comply with the Nigeria Data Protection Act 2023 (NDPA). In practice, most Nigerian founders cannot afford legal counsel, do not know which specific NDPA provisions apply to their systems, and cannot produce the structured compliance documentation that NDPC auditors require.

The core bottleneck is a reasoning gap: given a concrete technical or business finding — such as "user data is stored without encryption" — no affordable, automated system exists that maps it to the exact NDPA clause, explains the legal obligation, and tells the startup what to do.

---

## 2. System Architecture — SealVox AI

SealVox AI is a continuous compliance monitoring platform for Nigerian startups, built around three layers:

```
LAYER 1 — SCAN
Integrate with a startup's codebase or cloud infrastructure.
Continuously extract security, business, and compliance findings.
Output: structured findings (e.g. "no encryption at rest on customer database")

LAYER 2 — MAP  ← this submission
Take each finding and map it to the exact NDPA provision it violates.
Identify the legal obligation and the required remediation action.
Output: provision citation + compliance explanation + next-action plan

LAYER 3 — GENERATE
Produce submission-ready compliance documentation formatted for
NDPC, CBN, or NITDA auditors.
Output: structured compliance report
```

**This submission demonstrates Layer 2 — the mapping engine** — which is the most critical component. Without accurate regulatory mapping, Layers 1 and 3 produce useless output.

The architecture is **agentless by design**: the mapping engine does not browse the internet or make autonomous decisions. The law is baked into the model's fine-tuned weights. This makes output deterministic, auditable, and usable in air-gapped or restricted environments — important for regulated sectors like fintech and healthtech.

---

## 3. N-ATLAS Integration

The mapping engine is built by fine-tuning **N-ATLaS** (NCAIR1/N-ATLaS on Hugging Face) — Nigeria's sovereign multilingual large language model — on a hand-crafted dataset of NDPA compliance examples.

**Why N-ATLaS:** N-ATLaS was built by Nigerians, for Nigerian language and context. Using it for a Nigerian regulatory compliance system is the natural and strategically correct choice. It also supports Hausa, Igbo, and Yoruba — making future multilingual compliance guidance possible.

**Fine-tuning method:** QLoRA (4-bit quantisation + Low-Rank Adaptation via the PEFT library). Only 13.6 million of 8 billion parameters are trained, making this executable on a single free-tier Kaggle GPU (T4) with no paid infrastructure.

**Integration point:** The deployed API loads the base N-ATLaS model and applies the trained LoRA adapter at inference time. Every API call uses N-ATLaS as the reasoning backbone.

---

## 4. Dataset

- **Training set:** 83 hand-crafted input/output pairs
- **Format:** Each example pairs a real-world business or technical scenario (the "finding") with a compliance analysis citing the specific NDPA section, explaining the obligation, and stating the required action
- **Verification:** Every NDPA citation was verified against the official Federal Republic of Nigeria Official Gazette (Act No. 37, 1st July 2023). The verification script enumerated all 584 valid citation paths in the Act and confirmed zero invalid citations in the training or benchmark data
- **Coverage:** 26 of the Act's 66 sections are represented in training data

---

## 5. Training Setup

| Parameter | Value |
|---|---|
| Base model | NCAIR1/N-ATLaS (Llama-3 8B) |
| Method | QLoRA |
| LoRA rank (r) | 16 |
| LoRA alpha | 32 |
| Target modules | q_proj, k_proj, v_proj, o_proj |
| Dropout | 0.05 |
| Epochs | 3 |
| Batch size | 1 (gradient accumulation: 4) |
| Learning rate | 2e-4 |
| Hardware | Kaggle free-tier GPU (T4 x2) |
| Trainable parameters | 13,631,488 of 8,043,892,736 (0.17%) |

---

## 6. Benchmark Results

Evaluation was conducted on a held-out benchmark of 247 questions — never used during training — each verified against the official Act text.

| Model | Cites any section | **Correct section** | Exact subsection |
|---|---|---|---|
| Base N-ATLaS (no instruction) | 4.5% | 4.0% | 0.8% |
| Base N-ATLaS (instructed to cite NDPA) | 74.1% | 7.3% | 1.6% |
| **SealVox N-ATLaS (fine-tuned)** | **98.0%** | **20.2%** | **6.5%** |

The fine-tuned model is **2.8× more accurate** than the instructed base model on correct section citation. Improvement holds on the strict 220-question subset that shares no wording with any training example (19.5% vs 3.6%).

**Honest limitations:** The model sometimes cites the right section with incorrect subsection detail, and occasionally cites provisions that do not exist (14% of answers). These are expected limitations of a first-iteration fine-tune on 83 examples. The next version will add retrieval-augmented grounding from the Act text.

---

## 7. API Usage

The mapping engine is deployed as a documented REST API.

**Endpoint:** POST /ask

**Request:**
```json
{ "finding": "A fintech company stores customer BVN and phone numbers without encryption." }
```

**Response:**
```json
{ "answer": "This violates Section 39(1) of the NDPA, which requires data controllers to implement appropriate technical and organisational measures to ensure the security, integrity, and confidentiality of personal data. Section 39(2)(b) specifically names encryption as an example of such a measure. Required action: implement encryption at rest and in transit." }
```

Auto-generated API documentation is available at the `/docs` endpoint (FastAPI/OpenAPI standard).

**Adapter repository:** huggingface.co/ByteBuilderNG/sealvox-ai-ndpa-lora-adapter

---

## 8. Setup and Reproduction

1. Clone or access the repository
2. Install dependencies: `pip install transformers peft bitsandbytes accelerate fastapi uvicorn`
3. Load base model and adapter:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from peft import PeftModel
import torch

tokenizer = AutoTokenizer.from_pretrained("NCAIR1/N-ATLaS", token=HF_TOKEN)
base = AutoModelForCausalLM.from_pretrained("NCAIR1/N-ATLaS",
    quantization_config=BitsAndBytesConfig(load_in_4bit=True,
        bnb_4bit_quant_type="nf4", bnb_4bit_compute_dtype=torch.bfloat16),
    device_map="auto", token=HF_TOKEN)
model = PeftModel.from_pretrained(base, "ByteBuilderNG/sealvox-ai-ndpa-lora-adapter", token=HF_TOKEN)
model.eval()
```

4. Start the API server (see repository README for full instructions)
5. Access the UI at the published artifact link or self-host the single HTML file

---

*Submitted by Muhammad Muntaka Sarki and Ibrahim Abubakar — IBKHA Technologies / SealVox AI*
*NAIC 2026 | Track A | Problem Statement 3: Sectoral Fine-Tuning*
