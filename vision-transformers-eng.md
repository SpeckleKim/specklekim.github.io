---
layout: default
title: Vision Transformers (ViT)
lang: en
---

# Vision Transformers: Applying Transformers to Images

## Introduction

Vision Transformers (ViT) adapted the transformer architecture from NLP to computer vision, achieving remarkable results on image classification and related tasks. Unlike CNNs with inductive biases for spatial structure, ViT learns relationships purely from data, offering a different perspective on visual reasoning.

## Patch Embedding

ViT begins by dividing images into non-overlapping patches, typically 16×16 or 32×32 pixels. Each patch is flattened and linearly embedded into a fixed-dimensional vector. This creates a sequence of patch embeddings, enabling transformer processing.

**Key Insight:** Patches become tokens, analogous to words in NLP. The transformer processes this sequence without explicit spatial structure knowledge.

**Positional Encoding:** Patch position information is added via learnable position embeddings, allowing the model to understand spatial relationships.

## ViT Architecture

The standard ViT follows transformer design:
1. **Input:** Image patches + special [CLS] token
2. **Embedding:** Linear projection + positional encoding
3. **Transformer Encoder:** Multiple layers of multi-head self-attention and feed-forward networks
4. **Classification:** MLP head applied to [CLS] token output

Unlike CNNs that hierarchically downsample, ViT maintains resolution throughout, processing all patches at the same scale.

## DeiT (Data-Efficient Image Transformers)

ViT initially required massive datasets (JFT-300M) for strong performance. DeiT improved data efficiency through:

- **Knowledge Distillation:** Learning from a CNN teacher, reducing data needs
- **Regularization Techniques:** Stochastic depth, label smoothing, augmentation strategies
- **Achieves Competitive Results:** With ImageNet-only training, ViT becomes practical

## Swin Transformer

Introduces hierarchical structure to ViT through shifted window attention:

- **Hierarchical Design:** Progressive downsampling like CNNs, creating feature pyramid
- **Shifted Windows:** Attention computed within local windows, then shifted for cross-window connections
- **Efficiency:** Dramatically reduces computational complexity compared to global attention
- **Versatility:** Effective for detection, segmentation, and classification

## Hybrid Approaches

Combine transformer strengths with CNN inductive biases:

- **CNNs → ViT:** Use CNN as backbone for patch generation
- **ViT → CNN:** Early transformer layers followed by convolutional heads
- **Intermixed:** Attention and convolution layers alternately arranged

Hybrids often achieve better performance than pure approaches with less data.

## ViT vs CNN Debate

**ViT Advantages:**
- Better scalability with data and model size
- Learns global context without explicit architectural constraints
- More interpretable attention patterns
- Flexible for diverse input types (images, video, 3D)

**CNN Advantages:**
- Superior efficiency (fewer parameters, faster inference)
- Stronger inductive bias for spatial locality
- Better performance with limited data
- Established optimization and deployment infrastructure

**Convergence:** Modern practice adopts both—ViT dominates in large-scale settings; CNNs remain optimal for efficiency-constrained scenarios. Hybrids often provide best trade-offs.

## Recent Developments

- **Efficient ViT Variants:** Distillation, pruning, quantization reducing model size
- **Masked Image Modeling:** Self-supervised pre-training analogous to BERT
- **Video ViT:** Extending ViT to temporal dimension for video understanding
- **Multimodal ViT:** CLIP-like models learning from image-text pairs

## Practical Considerations

Choose ViT when:
- Large training datasets available
- Computational resources (GPUs/TPUs) accessible
- Need for state-of-the-art accuracy important
- Model size/inference speed less critical

Choose CNN when:
- Limited training data
- Edge deployment or real-time requirements
- Computational efficiency essential
- Established tools and libraries preferred

## Conclusion

Vision Transformers represent a paradigm shift from CNNs, showing that explicit inductive biases are unnecessary for vision tasks. While CNNs remain relevant for efficiency, ViT's scalability and flexibility make it the dominant approach for large-scale vision models. The field continues exploring hybrid architectures and efficient variants to broaden ViT applicability.

