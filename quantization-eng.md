---
layout: default
title: Model Quantization
lang: en
---

# Model Quantization

Model quantization is a technique that reduces the numerical precision of model weights and activations, drastically decreasing model size and computational requirements while maintaining performance. This is essential for deploying [large language models](/llm-eng.html) on resource-constrained devices.

## The Need for Quantization

Modern language models contain billions to trillions of parameters. A 7 billion parameter model stored in 32-bit floating point (FP32) requires approximately 28 GB of memory. This makes deployment expensive, slow, and impractical for many applications. Quantization addresses this fundamental challenge.

Beyond memory savings, quantization provides:
- **Reduced bandwidth**: Smaller models require less data transfer
- **Faster computation**: Lower precision operations execute faster on specialized hardware
- **Lower energy consumption**: Inference becomes more power-efficient
- **Edge deployment**: Models can run on mobile devices and embedded systems

## INT8 and INT4 Quantization

**INT8 (8-bit integer)** is the most common post-training quantization approach. Each floating point value is mapped to an 8-bit integer, reducing memory by 4x while preserving model behavior surprisingly well:

```
quantized_value = round((original_value - zero_point) / scale)
```

Where scale and zero_point are calibration parameters determined from the range of values in the model.

**INT4 (4-bit integer)** quantization provides extreme compression (8x reduction) but requires more careful handling. With only 16 possible values, INT4 quantization is inherently lossy and can degrade performance significantly if not implemented properly.

## Post-Training Quantization (PTQ)

Post-training quantization applies quantization after the model is fully trained, requiring no retraining:

1. **Calibration**: Run the model on a representative dataset to collect [statistics](/probability-eng.html) about weight and activation distributions
2. **Scale Computation**: Calculate optimal scale and zero_point for each layer or tensor
3. **Quantization**: Convert weights and activations to lower precision
4. **Evaluation**: Test quantized model on validation data to verify performance

PTQ is fast and practical but often results in accuracy loss, especially with extreme quantization like INT4. Typical accuracy drops range from 1-5% depending on model size and quantization bit-width.

## Quantization-Aware Training (QAT)

Quantization-aware training incorporates quantization into the training process:

1. **Fake Quantization**: During training, weights are quantized and dequantized at each step, simulating the effect of quantization
2. **Gradient Computation**: Gradients are computed through the quantization operation
3. **Learning**: The model adapts to the quantization constraint during training
4. **Fine-tuning**: The model learns to compensate for quantization loss

QAT achieves better accuracy than PTQ because the model learns to work effectively under quantization constraints. However, it requires training data and computational resources to retrain the model.

## GPTQ (Generative Pre-trained Transformer Quantization)

GPTQ is an efficient INT4 quantization method based on the Optimal Brain Quantization framework. It works layer-by-layer:

- **Per-layer approach**: Quantizes one layer at a time using information from previously quantized layers
- **Hessian-based scaling**: Uses second-order information (Hessian matrix) to determine optimal quantization scales
- **Speed**: Quantizes a 7B parameter model in minutes, not hours
- **Quality**: Achieves reasonable performance at INT4 despite extreme compression

GPTQ has become the de facto standard for INT4 quantization in open-source models, enabling community model sharing and deployment.

## AWQ (Activation-aware Weight Quantization)

AWQ improves upon GPTQ by considering activation magnitudes:

- **Activation importance**: Not all weights are equally important; weights that handle larger activations are more critical
- **Selective precision**: Protects important weights by keeping them at higher precision or allowing more error in less important weights
- **Better accuracy**: AWQ achieves 1-2% better accuracy than GPTQ at INT4
- **Similar speed**: Retains GPTQ's efficiency advantages

AWQ represents the current state-of-the-art for INT4 quantization, balancing efficiency and accuracy.

## GGUF Format and Quantization Variants

GGUF ([GPT](/gpt-eng.html)-Generated Unified Format) is a file format designed for efficient inference:

- **Multiple quantization levels**: GGUF supports various bit-widths (2-bit, 3-bit, 4-bit, 5-bit, 6-bit, 8-bit)
- **CPU inference**: GGUF models can run efficiently on CPU without [GPU](/gpu-hardware-eng.html)
- **Portability**: Standardized format allows models to run across different frameworks
- **Wider adoption**: Enabled by popular tools like llama.cpp and Ollama

GGUF quantization is less aggressive than GPTQ (typically focuses on 4-5 bit) but provides better overall inference efficiency across different hardware.

## bitsandbytes: GPU Quantization

Bitsandbytes library enables quantization for [GPU](/gpu-hardware-eng.html) inference:

- **8-bit quantization**: Reduced memory for [LLM](/llm-eng.html).int8() quantization during inference
- **4-bit quantization**: Further reduced memory using nf4 (nested float 4) format
- **No training required**: Works on already-trained models
- **QLoRA integration**: Enables parameter-efficient fine-tuning of quantized models
- **Seamless integration**: Works directly with Hugging Face [transformers](/llm-eng.html) library

Bitsandbytes is widely used in practice for fine-tuning and inference on [GPU](/gpu-hardware-eng.html) hardware.

## Accuracy vs. Efficiency Tradeoff

The fundamental challenge in quantization is balancing compression and accuracy:

| Bit-width | Memory Savings | Typical Accuracy Drop | Speed | Use Case |
|-----------|-----------------|----------------------|-------|----------|
| FP32      | 1x              | 0%                   | Baseline | Reference |
| INT8      | 4x              | 1-2%                 | 2-3x  | Server/[GPU](/gpu-hardware-eng.html) |
| INT4      | 8x              | 3-8%                 | 4-8x  | Mobile/CPU |
| INT2      | 16x             | 10-20%               | 10-15x | Extreme edge |

Larger models (13B+) tolerate quantization better than smaller models. Task-specific requirements determine the acceptable accuracy loss.

## Practical Deployment Considerations

**Model Selection**: Start with models known to quantize well; some architectures are more quantization-friendly than others.

**Batch Quantization**: INT8 typically requires only calibration; INT4 may need fine-tuning for acceptable performance.

**Mixed Precision**: Consider keeping critical layers (embeddings, output layer) at higher precision while quantizing others.

**Hardware Compatibility**: Ensure your deployment hardware supports the quantization format and can achieve the promised speed improvements.

**Benchmarking**: Always test quantized models on your specific task before production deployment to measure actual accuracy impact and latency improvements.

Quantization has become essential infrastructure for practical [LLM](/llm-eng.html) deployment, enabling powerful models to run on diverse hardware from GPUs to mobile devices.
