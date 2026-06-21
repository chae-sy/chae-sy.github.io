---
title: "[Project] CUDA Matrix Multiplication Implementation and Optimization "
date: 2025-12-17
teaser: /images/2025-12-17/tiling.png
---
Implemented and optimized CUDA kernels for scaled dot-product attention, attention linear projection, and Q/K/V projection using shared-memory tiling, WMMA Tensor Cores, PTX-level optimizations, vectorization, and asynchronous pipelining, achieving performance comparable to or exceeding PyTorch and cuBLAS baselines on NVIDIA Ada GPUs.
# 1. Implementation of Scaled Dot-Product Attention Using CUDA Kernels

The scaled dot-product attention operation was implemented on NVIDIA RTX 4060 Ti, RTX 5080, and L40S GPUs. Various optimization techniques previously applied to matrix multiplication on the Ada architecture were adapted to the attention computation.

The computation pipeline consisted of three kernels. The (QK^T) matrix multiplication and attention mask addition were fused into a single kernel, followed by a softmax kernel and a matrix multiplication kernel for (PV). Through this design, the entire scaled dot-product attention could be executed using only three CUDA kernels.

## 1.1 Shared Memory Tiling
![Fig.1 Memory Tiling Optimization](/images/2025-12-17/tiling.png)
![Fig.2 Memory Tiling Optimization](/images/2025-12-17/tiling2.png)

A tiled matrix multiplication strategy was first implemented by loading partial matrices into shared memory, computing partial products, and accumulating the results.

Since the (QK^T) operation requires multiplication with a transposed (K) matrix, data were loaded from global memory in a non-transposed format to maximize memory coalescing. The transposition was then performed when storing the data into shared memory.

To further improve performance, 1D and 2D tiling techniques previously used in matrix multiplication were applied. These approaches reduce the frequency of accesses to both global memory and shared memory, thereby increasing data reuse and improving execution speed compared to a naïve shared-memory implementation.

![Fig.3 Vectorization](/images/2025-12-17/tiling2.png)
Furthermore, although CUDA compilers do not automatically vectorize accesses to contiguous global memory locations, vectorized memory access functions were explicitly utilized, resulting in additional performance improvements.

## 1.2 Warp-Level Optimization Using WMMA

Profiling results revealed that global memory loading was the primary bottleneck for FP32 computation, whereas computation itself became the dominant bottleneck in FP16. Since most modern deep learning models operate in low-precision formats such as FP16 and BF16, optimization efforts focused on FP16 execution.

GPUs follow a SIMT execution model. Threads within an SM execute the same instruction stream, and instructions are scheduled at the warp level. On modern NVIDIA GPUs, a warp consists of 32 threads.

Leveraging this execution model, the matrix multiplication operations within scaled dot-product attention were scheduled at the warp level. Since warps can execute independently and in parallel, this approach increases the overall degree of parallelism.

In addition, modern GPUs contain Tensor Cores designed specifically to accelerate matrix-matrix multiplication. Tensor Cores operate at the warp level, and NVIDIA provides WMMA APIs for accessing them. These WMMA operations were integrated into the matrix multiplication stage of scaled dot-product attention.

As a result, on the RTX 5080 GPU, the optimized implementation achieved a runtime of 0.87 ms for a sequence length of 512, matching the performance of PyTorch. However, for longer sequence lengths such as 1024 and 2048, the implementation remained approximately 1–4 ms slower than the PyTorch baseline.

## 1.3 Instruction-Level Optimization Using PTX
![Fig.4 PTX](/images/2025-12-17/ptx.png)

Despite multiple optimization stages, the implementation remained slower than the PyTorch baseline. Further profiling revealed a significant number of shared memory bank conflicts.

The performance gap became increasingly pronounced as matrix dimensions increased, suggesting that bank conflicts were a major bottleneck.

With WMMA, thread access patterns inside a warp cannot be directly controlled, making it difficult to avoid multiple threads accessing the same shared memory bank.

To address this issue, PTX instructions provided by NVIDIA were employed. NVIDIA provides instructions that permute addresses when storing data into shared memory. By utilizing these instructions, threads within the same warp were forced to access different shared memory banks, effectively eliminating bank conflicts.

As a result, the optimized implementation outperformed the PyTorch baseline across all sequence lengths that could fit within the memory capacity of the RTX 5080 GPU, up to a sequence length of 4096.

In particular, for sequence length 4096, PyTorch required 50.18 ms, whereas the PTX-optimized custom kernel completed in 39.92 ms, achieving approximately a 10 ms speedup.

## 1.4 Softmax Optimization
![Fig.5 Softmax Optimization](/images/2025-12-17/softmax.png)
While optimizing matrix multiplication, kernel-level profiling indicated that the softmax operation constituted a major performance bottleneck.

An online softmax algorithm was initially implemented to simultaneously compute the running maximum, perform numerical stabilization, and accumulate the normalization term. However, this approach required storing intermediate values in shared memory and introduced multiple synchronization barriers through repeated `__syncthreads()` calls.

To eliminate these overheads, the warp-level execution model used in matrix multiplication was applied to softmax computation as well. Each warp independently processed one softmax operation, removing the need for shared memory communication and synchronization.

Through this design, a softmax kernel with performance comparable to PyTorch's implementation was achieved.

# 2. Implementation of Attention Linear Projection Using CUDA Kernels

Matrix multiplication kernels were implemented to perform the Q/K/V projections and the output linear projection in the attention mechanism.

## 2.1 Shared Memory Tiling

A cache-blocked tiled GEMM kernel was implemented using shared memory. Input and weight matrices were loaded into shared memory tile-by-tile, and partial products were accumulated locally.

The primary objective was to reduce redundant global memory accesses and improve arithmetic intensity through tile reuse.

During implementation, considerations such as shared memory bank conflicts and boundary handling for incomplete tiles were required.

Performance evaluation showed that the implementation remained slower than cuBLAS, indicating the need for further optimization. Profiling suggested that memory access overhead rather than computational workload was the dominant bottleneck, motivating the adoption of register tiling as the next optimization stage.

In addition, accuracy verification revealed errors significantly exceeding acceptable tolerance levels, indicating issues in the computation process that required further investigation.

## 2.2 Accuracy Issues

For correctness verification, the outputs of the custom kernel were compared against cuBLAS results using the maximum absolute difference metric.

It was discovered that cuBLAS assumes column-major matrix layouts by default, whereas the custom kernel interpreted matrices in row-major format. This layout mismatch effectively introduced an implicit transpose operation and resulted in substantial numerical discrepancies.

After aligning the matrix layouts, the accuracy issue was resolved. Boundary conditions were also carefully examined to ensure that no out-of-bound accesses or missing computations occurred near tile boundaries.

## 2.3 Register Block Tiling: 1D and 2D

Register block tiling was introduced by allowing each thread to accumulate a subset of output elements directly in registers.

Both 1D and 2D block tiling strategies were implemented sequentially, increasing per-thread workload and improving data reuse.

Compared with the shared-memory-only implementation, register tiling improved performance by approximately 40%.

While 2D tiling further increases data reuse relative to 1D tiling, its effectiveness depends on the underlying bottleneck. If shared memory bandwidth or bank conflicts dominate execution time, additional tiling may provide limited benefits. Furthermore, increased register usage can reduce achieved occupancy due to higher register pressure.

These observations highlight the importance of balancing tile dimensions, shared memory utilization, and register consumption through careful parameter tuning.

## 2.4 Vectorization

To improve memory efficiency, vectorized memory accesses were introduced.

Contiguous memory locations were loaded as vectorized data structures both during global-to-shared memory transfers and shared-to-register transfers. This reduced the number of memory transactions and improved memory throughput.

Performance measurements showed approximately a 30% improvement after vectorization.

Additional optimization opportunities were identified through shared memory indexing refinements and tuning of tile parameters such as BK.

# 3. Implementation of Q/K/V Projection Using CUDA Kernels

A GEMM kernel was implemented for generating Q, K, and V projections in the attention mechanism.

The target problem consisted of FP16 input matrices (A) and (B), FP32 accumulation matrices (C) and (D), and matrix dimensions (M = N = K = 4096). The ultimate objective was to achieve performance comparable to cuBLAS.

## 3.1 Naïve MMA Kernel

### 3.1.1 Kernel Design

The simplest approach directly invokes the PTX instruction

```text
mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
```

to execute Tensor Core operations.

On Ada GPUs, each SM contains four warp schedulers and Tensor Cores, requiring at least four warps per thread block to fully utilize the hardware.

A thread block consisting of (16 \times 16) threads was used, yielding a total of eight warps. These warps were arranged as a (2 \times 4) grid, collectively computing a (32 \times 32) output tile.

Each warp computes a (16 \times 8) output tile, and all Tensor Core operations are issued through inline PTX assembly rather than CUDA high-level APIs.

### 3.1.2 Data Movement and Computation

The naïve kernel employs a straightforward data movement strategy.

Each thread independently loads FP16 elements from global memory into shared memory. These accesses are neither contiguous nor aligned, resulting in poor global memory utilization.

Subsequently, data are loaded from shared memory into registers. Since multiple threads within the same warp frequently access the same shared memory bank, substantial bank conflicts occur.

Because each loaded element participates in only a single MMA operation, data reuse is extremely limited. Consequently, memory access costs dominate execution time and Tensor Core resources remain underutilized.

### 3.1.3 Performance Analysis

Kernel 1.0 achieved an execution time of 4680 µs and a throughput of 29.4 TFLOP/s.

This corresponds to approximately 19.1% of cuBLAS performance and 17.8% of the theoretical peak performance of the RTX 4090.

Nsight Compute analysis identified shared memory throttling, synchronization barriers, and long-scoreboard stalls caused by global memory loads as the primary bottlenecks.

On average, each MMA instruction required approximately 180 cycles, substantially higher than the theoretical minimum of 32 cycles.

### 3.1.4 Basic Tiling Improvement: Kernel 1.1

Applying 2× tiling along both the M and N dimensions allows each warp to execute four MMA operations per iteration.

As a result, the output tile size expands from (32 \times 32) to (64 \times 64).

This simple modification reduces execution time to 2400 µs and increases throughput to 57.3 TFLOP/s, achieving approximately 37.3% of cuBLAS performance.

However, global memory inefficiencies and shared memory bank conflicts remain unresolved, limiting further performance gains.

## 3.2 Vectorized Loads and Permuted Shared Memory Layout

### 3.2.1 Motivation

The primary bottlenecks of Kernel 1 were inefficient global memory accesses, shared memory bank conflicts, and low data reuse.

To address these issues, techniques inspired by CUTLASS were adopted, including 128-bit vectorized loads using `uint4`, permuted shared memory layouts, and warp-level loading via `ldmatrix`.

### 3.2.2 Vectorized Global Memory Loads

Tensor Cores operate on 128-bit vector fragments. Therefore, eight consecutive FP16 values along the K dimension were packed into a single `uint4` load.

Matrices A and B were stored in row-major and column-major formats, respectively. Warp-level loading ensured fully coalesced global memory accesses and significantly improved bandwidth utilization.

### 3.2.3 Permuted Shared Memory Layout

Shared memory consists of 32 banks with a width of 4 bytes.

Since a `uint4` occupies 16 bytes, naïve row-major storage would generate bank conflicts during warp-wide accesses.

To avoid this, an XOR-based permutation was applied:

```text
store_column = (laneID mod 8) XOR floor(laneID / 8)
```

This permutation eliminates bank conflicts both during global-to-shared stores and subsequent `ldmatrix` loads.

### 3.2.4 Register Loading Using `ldmatrix`

The instruction `ldmatrix.sync.aligned` loads (8 \times 128)-bit blocks from shared memory and transforms them into Tensor Core register fragments.

A tiles are loaded using `ldmatrix.x4`, while B tiles are loaded using `ldmatrix.x2`.

Because of the permuted shared memory layout, all threads access distinct banks, eliminating loading stalls.

### 3.2.5 Performance Improvements

Kernel 2.0 reduced execution time to 1080 µs and achieved 127.3 TFLOP/s, corresponding to 82.9% of cuBLAS performance.

Kernel 2.1 further improved register reuse by loading A tiles only once, reducing execution time to 1030 µs and increasing throughput to 133.4 TFLOP/s, or 86.9% of cuBLAS performance.

Average MMA latency decreased to 38 cycles, and the dominant bottleneck shifted from memory stalls to Tensor Core pipeline utilization.

## 3.3 N-Stage Global-to-Shared Pipeline

### 3.3.1 Motivation

Although Kernel 2 significantly improved performance, long-scoreboard stalls caused by global memory loads remained.

To further hide memory latency, an N-stage global-to-shared pipeline based on `cp.async` was introduced.

### 3.3.2 `cp.async` Pipeline Design

The `cp.async` instruction enables asynchronous data transfers from global memory to shared memory.

A circular buffer consisting of `N_STAGES` shared-memory buffers was allocated, and synchronization was controlled through `cp.async.commit_group` and `cp.async.wait_group`.

During execution, MMA operations on the current stage overlap with global-to-shared prefetching for the next stage, effectively hiding memory latency.

### 3.3.3 Performance Results and Extended Tiling

Kernel 3.0 achieved an execution time of 1000 µs and a throughput of 137.4 TFLOP/s, corresponding to 89.5% of cuBLAS performance.

Kernel 3.1 further expanded tiling along both M and N dimensions by 4×, increasing MMA density per warp.

As a result, execution time decreased to 895 µs, matching the performance of cuBLAS.

Warp-state analysis showed that most stalls were due to Tensor Core pipeline limitations, and average MMA latency reached 34.2 cycles, approaching the theoretical minimum of 32 cycles.

### 3.3.4 Summary

By combining an N-stage global-to-shared pipeline with large-scale tiling, global memory latency was effectively hidden and Tensor Core utilization was maximized.

These results demonstrate that achieving near-theoretical Tensor Core performance requires careful memory-layout design, warp-level data movement optimization, and asynchronous pipeline scheduling.
