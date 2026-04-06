---
layout: default
title: LoRA & Parameter-Efficient Fine-Tuning
lang: en
---

# LoRA & Parameter-Efficient Fine-Tuning

Parameter-Efficient Fine-Tuning (PEFT) methods address a critical practical challenge: full fine-tuning of [large language models](/llm-eng.html) is computationally prohibitive. LoRA (Low-Rank Adaptation) has emerged as the dominant approach, enabling cost-effective adaptation of massive models.

## The Full Fine-Tuning Bottleneck

Full fine-tuning updates all model parameters, requiring:
- **Memory**: Storing [optimizer](/optimizers-eng.html) states and gradients for all parameters. A 7B model needs 70+ GB RAM
- **Computation**: [Backpropagation](/backpropagation-eng.html) through all layers for every training sample
- **Time**: Days or weeks of training even with expensive hardware
- **Infrastructure**: Requires enterprise-grade GPUs (H100s, A100s)

This creates a paradox: while fine-tuning is essential for adaptation, only well-funded organizations can afford it. PEFT methods democratize model customization.

## LoRA: Low-Rank Adaptation

LoRA introduces a simple but effective insight: adaptation requires updating only a small fraction of parameters effectively. Rather than updating weight matrices W directly, LoRA adds learnable low-rank decompositions:

```
W' = W + ΔW = W + BA
```

Where:
- W is the original pretrained weight matrix
- B and A are low-rank matrices with much fewer parameters
- For a weight matrix of size (d_in × d_out), using rank r:
  - Original parameters: d_in × d_out
  - LoRA parameters: d_in × r + r × d_out (typically r << d)

For a 7B model with typical LoRA rank=8:
- Full fine-tuning: ~28GB parameters
- LoRA fine-tuning: ~12MB trainable parameters

This 2000x reduction in trainable parameters dramatically reduces memory and computation requirements while maintaining competitive performance.

## LoRA Training Process

1. **Initialization**: Initialize B as zeros and A randomly (or vice versa), so ΔW = 0 initially
2. **Frozen Base**: The pretrained weights W remain frozen throughout training
3. **Compute Adaptation**: During forward pass, compute W' = W + BA
4. **Update Only LoRA**: Gradients only update B and A parameters
5. **Deployment**: Merge ΔW back into W: W_new = W + BA, or keep separate for inference

This approach maintains compatibility with existing inference code while drastically reducing training costs.

## QLoRA: Efficient Fine-Tuning of Quantized Models

QLoRA extends LoRA to quantized models, enabling even more aggressive compression:

**Key innovation**: Combine LoRA with 4-bit [quantization](/quantization-eng.html)
- Quantize the base model to 4-bit (reducing size from 28GB to 3.5GB)
- Add LoRA adapters for fine-tuning
- Fine-tune using 8-bit [optimizer](/optimizers-eng.html) state

**Results**: A 7B model requires only 7GB [GPU](/gpu-hardware-eng.html) memory for QLoRA fine-tuning, making it accessible on consumer GPUs.

QLoRA achieves performance comparable to full fine-tuning with less than 5% of the memory required, making large-scale model adaptation practical for individual researchers.

## Adapters: Modular Fine-Tuning

Adapters introduce learnable modules between [transformer](/llm-eng.html) layers:

```
x' = Adapter(x) + x  # Residual connection
```

**Architecture**:
- Small bottleneck layers inserted into [transformer](/llm-eng.html) blocks
- Down-project to lower dimension (e.g., 64), process, then up-project back
- Typically 0.5-3% additional parameters

**Advantages**:
- Modular: Different adapters can be composed for different tasks
- Composable: Combine multiple adapters for multi-task learning
- Flexible: Can be applied to different layer types

Adapters are particularly useful for multi-task scenarios where one adapter per task is more flexible than a single LoRA.

## Prefix Tuning

Prefix tuning adds learnable tokens to the beginning of each sequence:

**Key idea**: Only the prompt prefix is learnable, not the model weights
- Prepend k learnable tokens to the input
- These "soft prompts" are optimized through [gradient descent](/calculus-eng.html)
- Model weights remain frozen

**Characteristics**:
- Extremely parameter-efficient (k × hidden_dim parameters)
- Works like in-context learning but optimized
- Can be task-specific or instruction-conditioned

Prefix tuning is especially effective for prompt-based learning and works well with [prompt engineering](/prompt-eng.html) techniques.

## IA3: Infused Adapter by Inhibition

IA3 applies element-wise multiplication to inject task-specific scaling:

**Mechanism**:
- Learn scaling factors for query, key, and value vectors in attention
- Learn scaling factor for the feed-forward network
- Minimal parameter overhead (only scaling vectors)

**Characteristics**:
- Extreme parameter efficiency: fewer parameters than LoRA
- Less flexible than LoRA for complex adaptations
- Specialized for attention-based architectures

IA3 is useful when maximum parameter efficiency is critical, though it's less commonly used than LoRA.

## Comparison of PEFT Methods

| Method | Params (7B) | Memory (QLoRA) | Flexibility | Composability | Use Case |
|--------|-------------|----------------|-------------|---------------|----------|
| Full Fine-tune | 28GB | 70GB+ | Highest | No | Research/unlimited budget |
| LoRA | 12MB | 7GB | High | Good | General purpose, single task |
| QLoRA | 12MB | 7GB | High | Good | Limited [GPU](/gpu-hardware-eng.html), consumer devices |
| Adapters | 100MB | 10GB | High | Excellent | Multi-task learning |
| Prefix Tuning | <10MB | 7GB | Medium | Good | Prompt-based learning |
| IA3 | <5MB | 7GB | Low | Poor | Extreme efficiency needed |

## Practical Implementation with PEFT

**Hugging Face PEFT library** provides easy-to-use implementations:

```python
from peft import LoraConfig, get_peft_model

config = LoraConfig(
    r=8,  # Low rank
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05
)

model = get_peft_model(base_model, config)
# Now fine-tune with standard training loop
```

**Practical tips**:
- Start with rank=8 and adjust based on task complexity
- Higher rank = more expressiveness but more parameters
- Target key modules: query/value projections capture most adaptation
- Combine with [quantization](/quantization-eng.html) for memory-constrained scenarios

## When to Use Each Method

**LoRA**: Default choice for most fine-tuning scenarios; excellent balance of efficiency and effectiveness.

**QLoRA**: When [GPU](/gpu-hardware-eng.html) memory is severely limited; maintains performance with 10-20x less memory.

**Adapters**: Multi-task or multi-domain scenarios; modularity is more important than parameter efficiency.

**Prefix Tuning**: Prompt-based learning or when compatibility with in-context learning is desired.

**IA3**: Extreme parameter efficiency is critical and the task is well-suited to multiplicative scaling.

## Impact on Production Systems

PEFT methods have fundamentally changed the economics of language model deployment:

- **Individual researchers** can now fine-tune 7B-70B models on consumer GPUs
- **Rapid experimentation** replaces slow, expensive training pipelines
- **Specialized models** become practical for niche domains and tasks
- **Multi-task systems** can use shared base models with task-specific adapters

LoRA and related methods have become industry standard, enabling the proliferation of specialized language models across diverse applications.
