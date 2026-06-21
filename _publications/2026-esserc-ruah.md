---
title: "RUAH: A Reconfigurable Unified Acoustic Processor with Hierarchical Inference for Keyword Spotting and Speaker Verification"
collection: publications
category: "conference"
permalink: publication/2026-esserc-ruah
date: 2026-09-07
venue: "2026 IEEE 52nd European Solid-State Electronics Research Conference (ESSERC)"
authors: "Donghwan So, Seojin Kim, Huijeong Woo, Jeongmin Lee, Yejin Hwang, Seoyoon Chae, John Kustin, Seungjong Lee, and Taewook Kang"
doi: "10.1109/ISOCC66390.2025.11329964"
---

Abstract

Always-on acoustic inference for edge devices requires an energy-efficient and scalable processor, yet prior designs remain limited by fixed-task operation and redundant computation. RUAH is a reconfigurable, unified, hierarchical AI processor for acoustic processing that supports VAD, KWS, SV, and more (Spoofing and Emotion) within a single hardware architecture. It employs a shared neural front-end and a modified BC-ResNet backbone to enable progressive inference by reusing intermediate features across task stages. RUAH reduces redundant computation through frame-wise incremental processing and a dynamic global buffer that maximizes feature reuse during streaming inference. It also supports reconfigurable execution of multiple models and task modes through a hierarchical control unit and a multi-mode PE. Fabricated in 65-nm CMOS, RUAH achieves 97.6% VAD, 95.2% KWS, and 96.8% SV accuracy while consuming 39.59µW, demonstrating a scalable and energy-efficient solution for always-on acoustic intelligence. 

Keywords: Reconfigurable processor, Hierarchical inference, Voice activity detection (VAD), Keyword spotting (KWS), Speaker verification (SV), BC-ResNet 