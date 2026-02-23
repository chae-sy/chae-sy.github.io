---
title: "Research"
permalink: /research/
author_profile: true
---
My research includes KV-cache quantization and tile-level precision control for LLM inference, optimized CUDA kernels leveraging block tiling and Tensor Core WMMA instructions, and Multi-Query Attention and W4A4 for FPGA LLM deployment.

I also work on hardware-friendly deployment of CNN keyword-spotting models, PTQ/QAT quantization, Verilog RTL design, ASIC tape-out, and a 1D-CNN LSTM–based fault diagnosis system using vibration and acoustic signals.

## 📦 Large Language Model Inference

### KV-cache quantization and GPU inference

- Developed adaptive KV-cache quantization and tile-level precision control for LLM inference, co-designing quantization policies with memory bandwidth and hardware dataflow constraints to reduce KV memory traffic while preserving model quality.

- Built optimized CUDA kernels for LLM inference leveraging block tiling, vectorized memory access, shared-memory reuse, warp-level tiling, and Tensor Core WMMA instructions. Reached near-PyTorch training/inference performance. [link]

### FPGA LLM deployment

![LLM on FPGA](/images/llm_on_fpga.jpg)

- Implemented Multi-Query Attention and W4A4 for FPGA LLM deployment; designed RTL in SystemVerilog for Transformer blocks; conducted synthesis and bitstream generation for FPGA implementation in Xilinx Vivado. [link](https://chae-sy.github.io/LLM-on-FPGA/)

---

## 📦 Hardware-aware Deep Learning and ASIC Implementation

![quantized BC-ResNet models for keyword spotting and speaker verification](/images/bc_resnet.jpg)

- Fine-tuned CNN keyword-spotting (KWS) models for hardware-friendly deployment using framewise incremental computation scheme; designed Verilog RTL blocks for efficient CNN operations. Participated in the Multi-Project Wafer (MPW) program for undergraduate and graduate students organized by IDEC and contributed to the tape-out of KWS ASIC tape-out. [link](https://chae-sy.github.io/CNN-Tape-Out/)

- Quantized BC-ResNet for keyword spotting (KWS) and speaker-verification(SV) embeddings; implemented PTQ/QAT (CLE, bias correction, AWQ, SmoothQuant, PACT). Participated in the Multi-Project Wafer (MPW) program organized by IDEC and contributed to KWS and SV ASIC tape-out. [link](https://chae-sy.github.io/Quantized-BC-ResNet-Tape-Out/)

---

## 📦 Intelligent Sensing and Fault Diagnosis

[link](https://chae-sy.github.io/Smart-Factory/)
- Developed a 1D-CNN LSTM–based fault diagnosis system for rotating machinery using vibration and acoustic signals, achieving up to 97.4% classification accuracy across multiple fault conditions.

- Utilized time-series feature extraction and sequence learning techniques to detect mechanical anomalies such as mechanical looseness, improving predictive maintenance capability.

---

## 📦 Hardware Accelerator Project

![Front-end and Back-end Design of Digital CNN Accelerator for SubPixel Rendering Algorithm](/images/subpixel_render.jpg)
[link](https://chae-sy.github.io/Capstone-Design/)

- Led a team of four undergraduates to implement and train a CNN-based subpixel rendering algorithm, optimizing network parameters using Incremental Network Quantization scheme.

- Designed its hardware accelerator in Verilog: developed a pipelined structure and validated its functionality.

- Executed the full digital ASIC tape-out flow, including RTL synthesis, timing-driven place-and-route, power and timing sign-off, and final GDSII generation. Awarded Engineering Innovation Prize.
