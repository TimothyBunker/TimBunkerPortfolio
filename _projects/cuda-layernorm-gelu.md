---
title: Optimized CUDA Kernels - 22x Faster LayerNorm+GELU
description: Hand-written CUDA kernels achieving 6-22x speedups over PyTorch native operations through shared memory optimization and kernel fusion.
date: 2025-01-10 12:00:00
label: CUDA / Performance
image: '/TimBunkerPortfolio/images/cuda-benchmark-cover.png'
---

# CUDA Kernel Optimization: Fused LayerNorm + GELU

## Performance Results

Benchmarked on RTX 4090:

| Shape | PyTorch | Optimized | Speedup |
|-------|---------|-----------|---------|
| (128, 1024) | 0.131ms | 0.006ms | **21.87x** |
| (256, 1024) | 0.050ms | 0.006ms | **8.03x** |
| (1024, 1024) | 0.054ms | 0.007ms | **7.32x** |

Maximum numerical error vs PyTorch: `4.76e-04`

---

## The Problem

Transformer models heavily rely on LayerNorm and GELU activations. In PyTorch, these are typically executed as separate operations:

```python
x = F.layer_norm(x, normalized_shape)
x = F.gelu(x)
```

Each operation requires:
1. Reading the entire tensor from global memory
2. Performing computation
3. Writing results back to global memory

This memory bandwidth becomes the bottleneck, not compute.

---

## The Solution: Kernel Fusion

By fusing LayerNorm and GELU into a single CUDA kernel, we:
- Read input data once
- Compute LayerNorm statistics
- Apply normalization AND GELU in one pass
- Write output once

**Result**: Dramatic reduction in memory traffic.

---

## Implementation Details

### The Math

```
output = GELU(LayerNorm(x))
       = GELU((x - μ) / σ)

where GELU(x) = 0.5 * x * (1 + tanh(√(2/π) * (x + 0.044715 * x³)))
```

### Baseline Kernel (Naive)

```cuda
// 1 thread per element - each thread recomputes mean/variance
__global__ void layernorm_gelu_baseline(float* input, float* output, 
                                         int batch_size, int hidden_dim) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    int row = idx / hidden_dim;
    
    // Each thread computes mean for entire row (wasteful!)
    float mean = 0.0f;
    for (int i = 0; i < hidden_dim; i++) {
        mean += input[row * hidden_dim + i];
    }
    mean /= hidden_dim;
    // ... similar for variance
}
```

**Problem**: O(hidden_dim) redundant computation per thread.

### Optimized Kernel

```cuda
__global__ void layernorm_gelu_optimized(float* input, float* output,
                                          int batch_size, int hidden_dim) {
    extern __shared__ float shared[];
    float* shared_sum = shared;
    float* shared_sum_sq = shared + blockDim.x;
    
    int row = blockIdx.x;
    int tid = threadIdx.x;
    
    // Parallel reduction for mean/variance
    float thread_sum = 0.0f, thread_sum_sq = 0.0f;
    for (int col = tid; col < hidden_dim; col += blockDim.x) {
        float val = input[row * hidden_dim + col];
        thread_sum += val;
        thread_sum_sq += val * val;
    }
    
    shared_sum[tid] = thread_sum;
    shared_sum_sq[tid] = thread_sum_sq;
    __syncthreads();
    
    // Tree reduction
    for (int stride = blockDim.x / 2; stride > 0; stride >>= 1) {
        if (tid < stride) {
            shared_sum[tid] += shared_sum[tid + stride];
            shared_sum_sq[tid] += shared_sum_sq[tid + stride];
        }
        __syncthreads();
    }
    
    // Compute mean and variance
    float mean = shared_sum[0] / hidden_dim;
    float var = (shared_sum_sq[0] / hidden_dim) - (mean * mean);
    float inv_std = rsqrtf(var + 1e-5f);  // Fast inverse sqrt
    
    // Apply LayerNorm + GELU in one pass
    for (int col = tid; col < hidden_dim; col += blockDim.x) {
        float val = input[row * hidden_dim + col];
        float normalized = (val - mean) * inv_std;
        output[row * hidden_dim + col] = gelu(normalized);
    }
}
```

### Key Optimizations

1. **Shared Memory Reduction**: Threads cooperate to compute statistics in O(log n) instead of O(n)
2. **Coalesced Memory Access**: Grid-stride loops ensure memory coalescing
3. **`rsqrtf()`**: Hardware-accelerated inverse square root
4. **Single Pass Output**: Normalization and activation computed together

---

## Kernel Configuration

```cpp
// 1 block per batch row, 256 threads per block
dim3 grid(batch_size);
dim3 block(256);
size_t shared_mem = 2 * 256 * sizeof(float);

layernorm_gelu_optimized<<<grid, block, shared_mem>>>(input, output, 
                                                       batch_size, hidden_dim);
```

---

## Full Benchmark Results

```
Shape                PyTorch (ms)    Optimized (ms)  Speedup
────────────────────────────────────────────────────────────
(128, 768)           0.0507          0.0066          7.71x
(256, 768)           0.0572          0.0071          8.10x
(1024, 768)          0.1221          0.0150          8.16x
(128, 1024)          0.0504          0.0065          7.73x
(256, 1024)          0.0748          0.0062          11.99x
(1024, 1024)         0.1122          0.0090          12.47x
(256, 2048)          0.1049          0.0082          12.86x
```

---

## What I Learned

- **GPU Architecture**: Understanding warps, shared memory banks, and memory coalescing
- **Profiling**: Using NVIDIA Nsight Compute (`ncu`) for kernel analysis
- **Numerical Stability**: Balancing speed with floating-point accuracy
- **PyTorch Extensions**: Building custom CUDA extensions with proper Python bindings

---

## Usage

```python
import torch
import layernorm_gelu_cuda

x = torch.randn(128, 768, device='cuda')
output = layernorm_gelu_cuda.optimized(x, eps=1e-5)
```

---

## Requirements

- CUDA Toolkit 11.0+
- PyTorch 2.0+
- NVIDIA GPU with compute capability 7.0+
