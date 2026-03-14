---
title: "LLM on FPGA: Squeezing Language Models by Quantization and Multi-Query Attention and Its Efficient Hardware Architecture"
collection: publications
category: "conference"
permalink: publication/2025-isocc-llm-fpga
date: 2025-10-16
venue: "22nd International SoC Design Conference (ISOCC)"
paperurl: /files/2025-isocc-llm-fpga.pdf
authors: "Seoyoon Chae, Taewook Kang"
note: "Acceptance rate: ~40%"
doi: "10.1109/ISOCC66390.2025.11329964"
githuburl: https://github.com/chae-sy/squeezing_lm
---

[Link](https://doi.org/10.1109/ISOCC66390.2025.11329964)

Abstract

We present an on-chip implementation of a compressed Transformer-based language model on a Xilinx Artix-7 FPGA. Our contributions include: (1) combining ultra-low-precision quantization (4 bits) and multi-query attention (MQA) to compress the KV cache by 8×, enabling sequence lengths up to 256 tokens; (2) a streaming hardware architecture in Verilog that implements pre-layernorm, attention, and feed-forward sublayers using block RAM (BRAM) and DSPs; and (3) post-synthesis results demonstrating real-time throughput (4.4 K tokens/s) with BRAM and DSP utilizations of 31.9% and 85%, respectively. The prototype supports generative inference entirely on-chip, paving the way for privacy-preserving, edge-scale LLMs. Code and scripts are available at https://github.com/chae-sy/squeezing_lm
