---
title: "[project] Senior Year Capstone Design : Front-end and Back-end Design of Digital CNN Accelerator for SubPixel Rendering Algorithm"
date: 2025-06-20
teaser: /images/2025-06-20/image.png
---

This project was carried out as a Senior Capstone Design from March to June 2025.
It received the Engineering Innovation Prize at the SKKU Senior Capstone Design Contest 2025.
I served as the team leader, and my teammates were Somin Kang, Miri Kim, and Jaehwan Shin.

[https://github.com/chae-sy/capstone_design](https://github.com/chae-sy/capstone_design)

![발표자료_2조-1-1.jpg](/images/2025-06-20/1-1.jpg)

![발표자료_2조-1-2.jpg](/images/2025-06-20/1-2.jpg)

we will present our project titled **‘Subpixel Rendering CNN Accelerator Based on Hardware–Software Co-Design Methodology.**

![발표자료_2조-1-4.jpg](/images/2025-06-20/1-4.jpg)

![image.png](/images/2025-06-20/image.png)

The demand for high-resolution displays continues to grow, yet achieving high-quality visual performance remains challenging due to physical pixel density limits and fabrication constraints.

To overcome these limitations, *Subpixel Rendering (SPR)* technology enhances perceived resolution by manipulating not only pixels but also individual subpixels

However, conventional filter-based subpixel rendering approaches have inherent performance limitations.

To address this, we adopted a CNN-based model called **SPRNN**, which learns nonlinear characteristics and improves both PSNR and SSIM metrics.

Since CNNs involve heavy computation, real-time processing on mobile devices is difficult.

![발표자료_2조-1-6.jpg](/images/2025-06-20/1-6.jpg)

![발표자료_2조-1-12.jpg](/images/2025-06-20/1-12.jpg)

To solve this, we applied a **hardware–software co-design** strategy that combines model compression and hardware acceleration to meet both performance and power requirements.

Notably, very few previous studies have designed accelerators specialized for subpixel rendering, and even fewer have applied HW–SW co-design methodologies

Our final design target is a **45-nm ASIC** that achieves:

- **PSNR ≥ 30 dB**, **SSIM ≥ 0.8**,
- **Chip area ≤ 2 mm × 2 mm**,
- **Power consumption ≈ 800 mW**, and
- **Processing speed ≥ 60 FPS**.”

![발표자료_2조-1-7.jpg](/images/2025-06-20/1-7.jpg)

![발표자료_2조-1-8.jpg](/images/2025-06-20/1-8.jpg)

The development schedule is shown here, and all stages were completed according to plan.

![발표자료_2조-1-9.jpg](/images/2025-06-20/1-9.jpg)

![발표자료_2조-1-10.jpg](/images/2025-06-20/1-10.jpg)

![발표자료_2조-1-11.jpg](/images/2025-06-20/1-11.jpg)

The goal of this work is to implement the **Subpixel Rendering module within the Display Controller** as an **ASIC**.

We adopted a **spatial architecture** that enables spatially parallel dataflow computation.

Additionally, a **pipelined processing method** was applied by dividing operations into multiple stages to enhance throughput.

We used **Incremental Network Quantization (INQ)** during CNN training to constrain weights to powers of two.

This allows multiplication operations to be replaced entirely by **bit-shift operations** significantly simplifying hardware.

INQ progressively quantizes the most critical weights to power-of-two values while continuing to fine-tune the remaining parameters, thus maintaining accuracy.

![발표자료_2조-1-13.jpg](/images/2025-06-20/1-13.jpg)

![발표자료_2조-1-14.jpg](/images/2025-06-20/1-14.jpg)

![발표자료_2조-1-15.jpg](/images/2025-06-20/1-15.jpg)

We implemented the CNN model identical to that in the reference paper using **PyTorch**.

The training achieved **PSNR = 30.22 dB** and **SSIM = 0.86**, meeting our target performance.

Next, we applied **INQ-based model compression**.

After retraining with power-of-two weights, the model achieved **30.01 dB PSNR**, showing minimal degradation compared to the original model.

![발표자료_2조-1-16.jpg](/images/2025-06-20/1-16.jpg)

The top-level hardware diagram consists of **computation modules, buffers, memory blocks, and a controller**, as shown in the figure

![발표자료_2조-1-17.jpg](/images/2025-06-20/1-17.jpg)

![발표자료_2조-1-18.jpg](/images/2025-06-20/1-18.jpg)

![발표자료_2조-1-19.jpg](/images/2025-06-20/1-19.jpg)

The **pipelined PE array** consists of **16 superscalar PEs**, each performing a **3×3 MAC** operation for RGB channels over **9 cycles**, producing **20-bit outputs**. Both design and simulation were completed.
A **two-stage adder tree** was designed and simulated to sum the partial results from 16 channels

![발표자료_2조-1-20.jpg](/images/2025-06-20/1-20.jpg)

![발표자료_2조-1-21.jpg](/images/2025-06-20/1-21.jpg)

The **ReLU–bias module** aligns a **24-bit input** with a **32-bit bias** via shift operations, performs bias addition, applies the **ReLU** activation, and truncates the result to **8 bits**. The module was fully implemented and verified.
We also designed and verified the **Maxpool unit**, which performs max-pooling operations.

![발표자료_2조-1-22.jpg](/images/2025-06-20/1-22.jpg)

![발표자료_2조-1-23.jpg](/images/2025-06-20/1-23.jpg)

![발표자료_2조-1-24.jpg](/images/2025-06-20/1-24.jpg)

![발표자료_2조-1-25.jpg](/images/2025-06-20/1-25.jpg)

The **buffer** temporarily stores and outputs data fetched from SRAM.
The **input buffer** was designed to perform **data-shift operations** to maximize **data reuse**.
The **weight buffer** supports **simultaneous read and write operations**, minimizing latency.
The **output buffer** stores each channel output and performs a **single write-back to SRAM** once all 16 channel results are ready.

![발표자료_2조-1-26.jpg](/images/2025-06-20/1-26.jpg)

![발표자료_2조-1-27.jpg](/images/2025-06-20/1-27.jpg)

![발표자료_2조-1-28.jpg](/images/2025-06-20/1-28.jpg)

The **memory** stores the **input feature maps, weights, and biases**.
The **input feature memory** is divided into two banks, **A and B**, forming a **ping-pong structure** that alternately stores feature maps between layers.
The **weight memory** and **bias register file** store and output the respective parameters.

![발표자료_2조-1-29.jpg](/images/2025-06-20/1-29.jpg)

![발표자료_2조-1-30.jpg](/images/2025-06-20/1-30.jpg)

![발표자료_2조-1-31.jpg](/images/2025-06-20/1-31.jpg)

The **top controller** manages the entire system operation.

As shown in the dataflow diagram, each module is orchestrated so that **input features are fetched into buffers** while **PEs concurrently perform 3×3 convolution** over a 9-cycle pipeline, minimizing overall latency.

The left figure shows the CNN model computation mapped onto the right hardware architecture, illustrating the **overall processing flow**.

![발표자료_2조-1-32.jpg](/images/2025-06-20/1-32.jpg)

![발표자료_2조-1-33.jpg](/images/2025-06-20/1-33.jpg)

![발표자료_2조-1-34.jpg](/images/2025-06-20/1-34.jpg)

The top-level simulation confirmed correct memory address mapping and pipeline operation, as well as proper data transfer between layers.

![발표자료_2조-1-35.jpg](/images/2025-06-20/1-35.jpg)

Project management was conducted via **GitHub**, with a total of approximately **300 commits** recorded.

![발표자료_2조-1-36.jpg](/images/2025-06-20/1-36.jpg)

![발표자료_2조-1-37.jpg](/images/2025-06-20/1-37.jpg)

Through quantization profiling, we analyzed the integer and fractional bit requirements for each operation.

We selected **Input = int2 + frac5**, **Weight = int1 + frac6**, and **Bias = int1 + frac31** configurations. The final bit-width configuration is summarized in the table.

![발표자료_2조-1-38.jpg](/images/2025-06-20/1-38.jpg)

In the verification stage, layer-wise output comparisons confirmed that the quantization error was within **±1 LSB**, corresponding to **0.027** in real value.

![발표자료_2조-1-39.jpg](/images/2025-06-20/1-39.jpg)

The synthesis result of the full **SPRNN_TOP** module showed a **cell area of 0.189 mm²** and **total power consumption of 217 mW**.

![발표자료_2조-1-40.jpg](/images/2025-06-20/1-40.jpg)

![발표자료_2조-1-41.jpg](/images/2025-06-20/1-41.jpg)

Comparing PE arrays implemented with **multipliers** versus **shifters**, synthesis results demonstrated an **11% area reduction** and **27% power savings**.

![발표자료_2조-1-42.jpg](/images/2025-06-20/1-42.jpg)

![발표자료_2조-1-43.jpg](/images/2025-06-20/1-43.jpg)

Using OpenLane, we performed automatic place-and-route (auto PnR).
The layout achieved DRC zero, LVS pass, and positive setup/hold margins across all corners.
(Post-layout simulation, though not included in the slides, was also confirmed as TEST PASSED.)

![발표자료_2조-1-44.jpg](/images/2025-06-20/1-44.jpg)

![발표자료_2조-1-45.jpg](/images/2025-06-20/1-45.jpg)

Finally, we will present our conclusion.

- Implemented a **CNN-based Subpixel Rendering module** as an **ASIC**.
- Maximized efficiency through **multi-level parallelism** and **input-stationary dataflow**.
- Achieved substantial **area and power reductions** via **INQ-based shift-only computation**.

The key contribution of our work lies in achieving target specifications through a **hardware–software co-design approach**, surpassing prior research.