Delighted to share that my research has been accepted to an oral session of ISOCC 2025 – 22th International SoC Design Conference! (presented on October 16, 2025)

you can find the whole paper : LLM on FPGA: Squeezing Language Models by Quantization and Multi-Query Attention and its Efficient Hardware Architecture [link](https://ieeexplore.ieee.org/abstract/document/11329964)

The paper got recognized "For enabling analytical evaluation of various quantization schemes (SmoothQuant and QLoRA) and implementing on-chip caches of large language models without siginificant accuracy drop by using Multi-Query Attention and efficient RTL design."

# LLM on FPGA: Squeezing Language Models by Quantization and Multi-Query Attention and Its Efficient Hardware Architecture

**Authors:**

Seoyoon Chae (Dept. of Electronic and Electrical Engineering, Sungkyunkwan University)

Taewook Kang (Dept. of Semiconductor Convergence Engineering, Sungkyunkwan University)

🔗 **Code:** [https://github.com/chae-sy/squeezing_lm](https://github.com/chae-sy/squeezing_lm)

---

## Abstract

We present an **on-chip implementation** of a compressed Transformer-based language model on a **Xilinx Artix-7 FPGA**.

Our main contributions are:

1. **Hybrid compression:** Combining **4-bit quantization** and **multi-query attention (MQA)** compresses the KV cache by **8×**, supporting sequences up to **256 tokens**.
2. **Streaming hardware design:** A **Verilog implementation** of pre-layernorm, attention, and feed-forward sublayers optimized with **BRAM** and **DSP utilization**.
3. **Post-synthesis performance:** Achieving **4.4 K tokens/s** at 250 MHz with **31.9% BRAM** and **85% DSP** utilization.

This prototype enables **fully on-chip generative inference**, advancing privacy-preserving and edge-deployable large language models.

**Keywords:** Large Language Model (LLM), Quantization, Multi-Query Attention, Transformer Accelerator, FPGA, On-Device Inference, Hardware–Software Co-Design

---

## 1. Introduction

Transformer-based **large language models (LLMs)** achieve state-of-the-art performance but require substantial compute and memory resources
Autoregressive generation typically caches **key–value (KV) tensors**, reducing attention complexity from *O(S²d)* to *O(Sd)* per token. However, storing 8-bit KV tensors on-chip severely limits sequence length.

For example, with **608 KiB BRAM** on an **Artix-7 FPGA**, the on-chip KV capacity is:

$text{Token capacity} = \frac{608 KiB}{12 \text{layers} × 2 \text{KV tensors} × 768 B} ≈ 33 \text{tokens.}$

To overcome this limitation, we integrate **sub-8-bit quantization** and **MQA** to compress the KV cache over **8×**, enabling **256-token inference** without DRAM.

**Key contributions:**

- **4-bit KV quantization** using SmoothQuant + LoRA-QAT achieves near-lossless accuracy (< 0.02 drop).
- **MQA (shared K/V across heads)** cuts bandwidth while preserving quality.
- **FPGA co-design:** A streaming Transformer block implemented entirely in Verilog.

---

## 2. Related Work

Previous research explored:

- Multi-head attention acceleration on systolic arrays [2].
- ASICs using per-vector 4-bit scaling [3].
- FPGA frameworks applying pruning [4].

However, none jointly exploit **sub-8-bit quantization** and **KV caching** for **generative LLM inference** under FPGA-scale memory constraints.

---

## 3. Model Compression Methodology

![image.png](/images/2025-10-16/image.png)

### 3.1 SmoothQuant for Range Equalization

SmoothQuant [5] was used to mitigate activation outliers before quantization.

- Stable results for **W8A8–W6A6** (PPL ≈ 60, Accuracy ≈ 0.6).
- Instability observed at **W4A4** (PPL = 116.1, Accuracy = 0.49).

### 3.2 LoRA-Based Quantization-Aware Training

LoRA adapters (rank = 8) were inserted into Q/K/V projections and fine-tuned on **WikiText-2** for 10–100 steps (frozen backbone).

- PPL = 45.5 after 100 steps.
- Combining with SmoothQuant reduced PPL to **22.2** (100 steps) and **55.6** (10 steps).

### 3.3 Multi-Query Attention (MQA)

Standard **MHA** computes separate K/V per head → high memory cost.

**MQA** shares K/V across all heads; **GQA** shares within groups.

In our experiments:

- **Group-3 MQA** saved significant memory.
- Slight trade-off: +2.6 PPL vs. baseline MHA.

**Result Summary:**

| Configuration | Training Steps | PPL | Accuracy |
| --- | --- | --- | --- |
| W4A4 + SmoothQuant (10 steps) + G3 MQA | 10 | 55.6 | 0.60 |
| W4A4 + SmoothQuant (100 steps) + G3 MQA | 100 | **22.6** | 0.58 |

---

## 4. Hardware Architecture

The hardware pipeline consists of **pre-layernorm**, **attention**, and **feed-forward network (FFN)** units

![image.png](/images/2025-10-16/image%201.png)

![image.png](/images/2025-10-16/image%202.png)

### 4.1 Pre-LayerNorm

Computes per-channel mean/variance, approximates $\frac{1}{\sqrt{\sigma^2 + \epsilon}}$ using a **LUT-based reciprocal square root**, and applies **4-bit affine parameters** (γ, β).

### 4.2 Attention and FFN Engines

- **Attention Unit:**
    - Generates concatenated Q/K/V via 1×1 convolution.
    - Stores 4-bit compressed K/V in shared BRAM.
    - Computes Q–K dot-products on a **PE array**.
    - Applies masked softmax using **exp/reciprocal LUTs**.
    - Produces context vector via weighted sum over V.
- **Feed-Forward Network (FFN):**
    - Expands C → 4C → C.
    - Uses LUT-based GELU activation.
    - Implemented as **streaming matrix-vector (mat-vec) computations** using DSPs and BRAM.

### 4.3 Resource Utilization

| Resource | Used | Available | Utilization (%) |
| --- | --- | --- | --- |
| **DSP slices** | 204 | 240 | **85.0** |
| **BRAM** | 193.5 KB | 608 KB | **31.9** |
| **LUTs** | 10,000 | 63,400 | **15.7** |

---

## 5. Evaluation

- **Platform:** Xilinx Artix-7 100T (CSG324)
- **Tool:** Vivado 2018.3
- **Clock Frequency:** 250 MHz

The fully pipelined design sustains **1 token per cycle** (after pipeline fill), reaching **4.4 K tokens/s** throughput.

On-chip memory supports **sequence length ≤ 256 tokens**, entirely without external DRAM.

---

## 6. Conclusion and Future Work

This work demonstrates the **first fully on-chip FPGA-based LLM inference** using:

- **4-bit quantization**,
- **Multi-Query Attention**, and
- A **streaming Verilog architecture**.

The prototype achieves **real-time generation** within **31.9% BRAM** and **85% DSP** budgets.

Future directions include:

- Scaling to larger LLMs,
- **Mixed-precision inference**, and
- **Energy-efficiency optimization**.

---

## References

1. Vaswani et al., *“Attention Is All You Need,”* NeurIPS 2017.
2. Lu et al., *“Hardware Accelerator for Multi-Head Attention and FFN,”* IEEE SOCC 2020.
3. Keller et al., *“A 95.6-TOPS/W Inference Accelerator with 4-bit Quantization in 5 nm,”* IEEE JSSC 2023.
4. Zhang et al., *“Algorithm-Hardware Co-Design of Attention on FPGA Devices,”* ACM TECS 2021.
5. Xiao et al., *“SmoothQuant: Accurate and Efficient Post-Training Quantization for LLMs,”* ICML 2023.
6. Hu et al., *“LoRA: Low-Rank Adaptation of LLMs,”* ICLR 2022.
7. Shazeer, *“Fast Transformer Decoding: One Write-Head Is All You Need,”* arXiv 2019.
8. Ainslie et al., *“GQA: Generalized Multi-Query Transformers,”* arXiv 2023.
