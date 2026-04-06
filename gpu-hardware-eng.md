---
layout: default
title: "GPU & Hardware for AI"
lang: en
---

# GPU & Hardware for AI

## Introduction

Deep learning and AI workloads demand specialized hardware that goes far beyond traditional CPUs. Graphics Processing Units (GPUs) have become the backbone of modern AI infrastructure, enabling the massive parallelism required for training and inference at scale. Understanding GPU architecture, memory systems, and hardware selection is crucial for building efficient AI systems.

## GPU Architecture Fundamentals

### CUDA Cores and Tensor Cores

NVIDIA's GPU architecture revolves around two key computational units:

**CUDA Cores** are general-purpose processing units designed for parallel computation. Each CUDA core can execute simple arithmetic operations independently, making them ideal for massively parallel workloads. A modern GPU contains thousands of CUDA cores organized into streaming multiprocessors (SMs).

**Tensor Cores** are specialized processors designed specifically for matrix operations, the fundamental building block of [neural networks](/neural-eng.html). A tensor core can perform fused multiply-accumulate (FMA) operations on small matrices in a single clock cycle. For example, an NVIDIA A100 has 108 tensor cores per SM with support for various precision levels (FP32, FP16, TF32, INT8).

The ratio of tensor cores to CUDA cores has been increasing with each GPU generation, reflecting the AI-centric evolution of hardware design.

### Memory Hierarchy

GPU memory follows a strict hierarchy:

- **Registers**: Ultra-fast, per-thread private memory (nanoscale latency)
- **Shared Memory**: Fast, per-SM cache shared among threads (microsecond latency)
- **L1/L2 Cache**: Automatic caching of global memory accesses
- **Global Memory (VRAM)**: Largest, slowest GPU memory (microsecond latency, but high bandwidth)
- **Host Memory (RAM)**: System RAM accessible via PCIe

Memory bandwidth is critical for AI workloads. The A100 provides up to 2TB/s bandwidth (with NVLink), while older GPUs like V100 offer 900GB/s via PCIe.

## NVIDIA Architectures

### A100 (Ampere, 2020)

The NVIDIA A100 set the standard for AI computing with:
- 40GB or 80GB HBM2 memory
- 108 tensor cores per SM, 108 SMs total
- Native support for TF32, BF16, FP16, INT8
- Built-in sparsity acceleration (50% higher throughput for sparse models)
- Strong for both training and inference

### H100 (Hopper, 2022)

The H100 pushed boundaries further:
- 80GB HBM3 memory with 3.2TB/s bandwidth
- 4x matrix engines per SM for higher tensor throughput
- DPX (Distributed Processing eXecution) for sparse tensor operations
- [Transformer](/llm-eng.html) Engine with automatic [mixed precision](/mixed-precision-eng.html)
- Significantly faster for [LLM](/llm-eng.html) training and inference

### Other Architectures

- **V100 (Volta, 2017)**: Pioneer tensor architecture, still used in older deployments
- **L40S (Ada, 2023)**: Optimized for inference workloads with mixed-precision support
- **RTX 4090 (Ada Consumer)**: Affordable alternative for small-scale training (24GB VRAM)

## TPUs and Alternative Accelerators

### Google TPUs

Tensor Processing Units are application-specific integrated circuits (ASICs) designed for TensorFlow workloads:
- TPUv4: Up to 275 teraFLOPS for matrix multiplication
- Excels at dense, regular computations (especially [transformer](/llm-eng.html) models)
- Integrated with GCP for seamless scaling
- More power-efficient than GPUs for specific workloads

### NPUs and Edge Accelerators

Neural Processing Units target edge devices:
- Apple Neural Engine: 16-core NPU in M-series chips
- Qualcomm Hexagon NPU: Found in Snapdragon processors
- Optimized for INT8/INT4 inference with low power consumption

## Key Performance Metrics

### TFLOPS (Teraflops)

Measures floating-point operations per second:
- A100: 312 TFLOPS (FP32), 1,248 TFLOPS (TF32)
- H100: 989 TFLOPS (FP32), 1,456 TFLOPS (TF32)
- Important baseline but doesn't account for memory bandwidth bottlenecks

### Memory Bandwidth

Measures data throughput between memory and compute:
- Bandwidth = Data Width × Clock Speed × 2 (for DDR)
- Critical for models with poor arithmetic intensity (low compute-to-memory ratio)
- H100: 3,200 GB/s with HBM3
- Increasingly important for inference workloads

## Hardware Selection Guidelines

### For Training

- **Budget unconstrained**: H100 clusters with NVLink/InfiniBand
- **Budget conscious**: A100 or RTX 4090 depending on model size
- **Multi-node scaling**: Prioritize GPU interconnect bandwidth
- **Memory requirements**: Larger models require 40GB+ VRAM

### For Inference

- **Throughput priority**: L40S or RTX 4090
- **Latency priority**: Lower batch sizes, consider CPU inference for small models
- **Cost per inference**: TPUs for dense, regular workloads
- **Edge deployment**: NPUs or INT8 quantized models

## Cost Considerations

GPU cost comprises:
- **Capital**: $10k-$40k per high-end GPU
- **Power consumption**: A100 (250W) vs H100 (700W)
- **Cooling infrastructure**: GPUs generate significant heat
- **Total Cost of Ownership**: Must factor multi-year amortization
- **Spot pricing**: Cloud providers offer 70-80% discounts for preemptible instances

## Future Trends

- **Increased chip density**: 5nm and 3nm processes enabling more cores
- **Specialized precision**: BF16 becoming standard for training
- **On-chip photonics**: Optical interconnects replacing electrical for inter-GPU communication
- **ASIC specialization**: Custom silicon for specific model architectures

## Conclusion

Choosing the right hardware is as critical as choosing the right algorithm. Understanding your workload's compute-to-memory ratio, precision requirements, and scalability needs guides optimal hardware selection. The GPU landscape evolves rapidly, but fundamental principles around parallelism, memory hierarchy, and bandwidth remain central to AI acceleration.
