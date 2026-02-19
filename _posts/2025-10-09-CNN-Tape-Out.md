
Design and Implementation of an End-to-End Keyword Spotting System Based on CNN Neural Networks

**Research Period:** September 2, 2024 – December 20, 2024

**Researcher:** *Seoyoon Chae*

**Affiliation:** Sungkyunkwan University

---

## 1. Paper Review and Algorithm Design

### (1) Paper Review

To investigate keyword-spotting (KWS) techniques, several papers were reviewed. Among them, one notable study proposed generating two-dimensional representations of speech signals by dividing them into **frequency and time axes**, then processing them through a **CNN-based network** [1].

The paper also described an **integer-quantized approach** to reduce power consumption and a **custom processing-element (PE) architecture** with a dedicated **memory structure**, optimized for low-power hardware. The study further minimized on-chip memory area by reducing SRAM size, which inspired the memory design adopted in this work.

![image.png](/images/2025-10-09/image.png)

However, the reference architecture could detect only one or two keywords, which is insufficient for large-scale KWS deployment. Therefore, in this research, the quantized bit-width was expanded from **1-bit to 8-bit**, enabling the detection of **five distinct keywords**. The speech signal was sampled every **32 ms**, and the **feature-extraction circuit** transformed the incoming waveform into **10 frequency bands** using FFT before applying CNN operations.

The **Depthwise Separable Convolutional Neural Network (DSCNN)** architecture was adopted to reduce computational cost while maintaining accuracy. Although it increases the number of stages by decomposing conventional convolution into **depthwise and pointwise** operations, it significantly lowers overall computation and memory usage to approximately **one-seventh** of a standard CNN. Python-based training confirmed about **90% recognition accuracy** under this structure.

![image.png](/images/2025-10-09/image%201.png)

### (2) BC-ResNet Model Design

To extend the number of recognizable keywords to ten, several lightweight AI architectures for KWS were examined. Among them, **BC-ResNet [2]** demonstrated high classification accuracy with low parameter count and FLOPs, making it suitable for hardware implementation.

| Model | Parameters | FLOPs | Accuracy (%) | Keywords |
| --- | --- | --- | --- | --- |
| **BC-ResNet-1** | 9.2 K | 3.1 M | **96.9** | 10 |
| DS-CNN S | 24 K | 5.4 M | 94.4 | 10 |
| TC-ResNet8 | 66 K | 3.0 M | 96.1 | 10 |
| Res8-Narrow | 20 K | 143.2 M | 90.1 | 10 |

Using **PyTorch**, the smallest BC-ResNet-1 model was fine-tuned and trained on the **Google Speech Commands v2** dataset, achieving **92.8% accuracy** on the evaluation set.

The architecture included convolutional, batch-normalization, ReLU6, dropout, and adaptive average-pooling layers, structured as modular blocks optimized for parameter efficiency.

---

## 2. RTL Design

### (1) DSCNN RTL Implementation

![image.png](/images/2025-10-09/image%202.png)

For hardware realization of keyword spotting, the system was designed at **Register-Transfer Level (RTL)** using **Verilog/SystemVerilog**. The architecture comprised:

- **Weight Memory** – stores pre-trained CNN weights
- **Input Memory** – receives features from the front-end
- **Internal Memory** – buffers intermediate results
- **Controller** – orchestrates operation sequencing
- **PE Array** – performs MAC operations (32 PEs, each 8-bit)
- **Adder Tree**, **Output Buffer**, and **Comparator** modules

Weights were trained offline and loaded via **SPI communication**. Each PE executed **8-bit integer MAC operations**, and verification through testbench simulations confirmed correctness.

The **Adder Tree** module aggregated partial sums from PEs during pointwise or fully connected layers, implemented as a single-cycle combinational logic block to minimize latency.

The **Output Buffer** temporarily stored completed channel outputs and applied the **ReLU activation function** with saturation.

The **Comparator** performed a tournament-style search to identify the maximum activation value, corresponding to the detected keyword.

The **Controller Module** coordinated all submodules across **five layers**, managing memory addresses, enable/reset signals, and inter-layer synchronization.

Latency analysis showed approximately **560 clock cycles** total (182 + 24 + 258 + 7 + 85 + inter-layer transitions).

### (2) BC-ResNet RTL Implementation

![image.png](/images/2025-10-09/image%203.png)

A more advanced circuit was designed to execute **BC-ResNet inference** in real time. The top-level architecture included:

- **PE Array** (16 PEs) for convolutional MAC operations
- **Normalization Module** for batch normalization
- **ReLU + Quantization Module** for activation and 8-bit precision adjustment
- **Classifier Module** performing average pooling and output aggregation
- **Max-Finder Module** selecting the highest-scoring output label

Feature and weight data were stored in **FSRAM** and **WSRAM**, respectively, with intermediate **feature/weight buffers** feeding the PE array.

Control was handled by a **top controller**, a **layer-state controller**, and **23 layer controllers** that managed each computational stage.

![image.png](/images/2025-10-09/image%204.png)

![image.png](/images/2025-10-09/image%205.png)

![image.png](/images/2025-10-09/image%206.png)

A detailed **state-machine design** ensured correct sequencing of weight loading, feature buffering, convolution, normalization, and activation per layer.

A **frame-wise computation scheme** was adopted to eliminate redundant operations between overlapping spectrogram windows (20 × 30 frequency × time). This **incremental computation method [1]** achieved equivalent results to full-window convolution with reduced redundancy, verified through timing analysis.

---

## 3. Logic Synthesis

After RTL verification, the design was synthesized using **Synopsys Design Compiler** for fabrication under **Samsung 28 nm CMOS** through the **IDEC MPW Program**.

During synthesis, non-synthesizable coding patterns (e.g., mixing blocking and non-blocking assignments, reserved identifiers, multi-signal resets) were corrected to ensure structural logic generation.

![image.png](/images/2025-10-09/image%207.png)

![image.png](/images/2025-10-09/image%208.png)

A **memory wrapper** cell was designed for each SRAM instance to:

1. prevent direct internal routing to memory cells, and
2. tie off unused I/O pins for electrical stability.

Post-synthesis, the **top-level hierarchy** was analyzed for **power** and **timing**, confirming functionality within expected constraints.

![image.png](/images/2025-10-09/image%209.png)

![image.png](/images/2025-10-09/image%2010.png)

---

## 4. Place and Route (P&R)

The physical implementation was carried out using **Synopsys IC Compiler II**.

The flow consisted of **placement**, **clock-tree synthesis (CTS)**, and **routing** stages.

### Memory Wrapper P&R

- Imported physical libraries for memory cells provided by the foundry.
- Defined **cell dimensions** and **placement coordinates**.
- Added **blockage boundaries** to prevent routing through memory regions.
- Implemented **power-plan connections (VDD/VSS)** and **guard-ring structures** for stable power delivery.

### Top Module P&R

- Instantiated completed memory wrappers.
- Created a **mesh-style power grid** across the layout.
- Performed **Design Rule Check (DRC)** and **Layout vs Schematic (LVS)** verification.

![image.png](/images/2025-10-09/image%2011.png)

- Added **I/O pad configurations** and finalized **GDSII** files for tape-out submission.

All DRC/LVS checks were passed, and the design was submitted to the foundry for final fabrication.

---

## References

[1] W. Shan *et al.*, “A 510-nW Wake-Up Keyword-Spotting Chip Using Serial-FFT-Based MFCC and Binarized Depthwise Separable CNN in 28-nm CMOS,” *IEEE J. Solid-State Circuits*, 56(1):151–164, Jan. 2021.

[2] B. Kim *et al.*, “Broadcasted Residual Learning for Efficient Keyword Spotting,” *arXiv:2106.04140*, 2023.

[3] Y. Zhang *et al.*, “Hello Edge: Keyword Spotting on Microcontrollers,” *arXiv:1711.07128*, 2018.

[4] S. Choi *et al.*, “Temporal Convolution for Real-Time Keyword Spotting on Mobile Devices,” *arXiv:1904.03814*, 2019.

[5] R. Tang and J. Lin, “Deep Residual Learning for Small-Footprint Keyword Spotting,” *Proc. ICASSP 2018*, pp. 5484–5488.

---

## Acknowledgment

This research was conducted as part of the **Undergraduate Research Program (URP)** and supported by the **IDEC Multi-Project Wafer (MPW)** initiative using **Samsung 28 nm CMOS process**.