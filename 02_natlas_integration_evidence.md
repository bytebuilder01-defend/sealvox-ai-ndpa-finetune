# N-ATLaS Integration Evidence
**SealVox AI — NAIC 2026 Submission**

---

## How N-ATLaS is Used

SealVox AI integrates N-ATLaS as the reasoning backbone of its regulatory mapping engine. N-ATLaS is not used as a simple prompt wrapper — it is the foundation that the fine-tuned adapter is built on top of.

### Integration Method: Fine-Tuning via QLoRA

The project fine-tunes N-ATLaS using QLoRA (Quantised Low-Rank Adaptation):

1. **Base model loaded:** NCAIR1/N-ATLaS is loaded in 4-bit quantised format using BitsAndBytesConfig
2. **LoRA adapters attached:** Small trainable adapter layers (13.6M parameters, 0.17% of total) are attached to the attention projection layers (q_proj, k_proj, v_proj, o_proj)
3. **Domain training:** The adapters are trained on 83 verified NDPA compliance examples using the SFTTrainer from the TRL library
4. **Adapter saved:** The trained LoRA adapter is saved and published at `huggingface.co/ByteBuilderNG/sealvox-ai-ndpa-lora-adapter`
5. **Inference:** At runtime, the base N-ATLaS model is loaded and the adapter is applied using PeftModel.from_pretrained() — every inference call uses N-ATLaS as its core

### Training Pipeline Used

The N-ATLaS training pipeline (Hugging Face Transformers + PEFT + TRL) was used throughout. The chat template native to N-ATLaS was applied to all training examples using `tokenizer.apply_chat_template()`, ensuring training and inference use the same format that N-ATLaS was designed for.

---

## Evidence of Integration

| Evidence | Detail |
|---|---|
| Model card | huggingface.co/ByteBuilderNG/sealvox-ai-ndpa-lora-adapter — base_model: NCAIR1/N-ATLaS |
| Training logs | global_step=63, training_loss=1.318, epoch=3.0 |
| Trainable parameters | 13,631,488 LoRA parameters on N-ATLaS backbone |
| Inference confirmed | Model loaded and queried on Kaggle GPU; correct NDPA section cited in response |
| Benchmark | 247-question evaluation run against both base N-ATLaS and fine-tuned version |

---

## Why N-ATLaS

N-ATLaS is the appropriate model for this work because:

- It is Nigeria's sovereign AI model, built by Nigerians on Nigerian language data
- It supports English, Hausa, Igbo and Yoruba — enabling future multilingual compliance guidance in the languages Nigerian founders actually speak
- Using a Nigerian model for a Nigerian regulatory problem is the correct architectural decision, not a shortcut

---

*This document is submitted as part of the SealVox AI NAIC 2026 application.*
*Muhammad Muntaka Sarki and Ibrahim Abubakar — IBKHA Technologies*
