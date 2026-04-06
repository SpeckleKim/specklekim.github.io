---
layout: default
title: CNN Deep Dive
lang: en
---

# CNN Deep Dive: Understanding Convolutional Neural Networks

## Introduction

Convolutional [Neural Networks](/neural-eng.html) (CNNs) revolutionized [computer vision](/cv-eng.html) by introducing parameter sharing and hierarchical feature extraction. Unlike fully connected networks, CNNs leverage the spatial structure of images through convolution operations, making them highly efficient for visual tasks.

## The Convolution Operation

At its core, convolution is a mathematical operation between an input and a kernel (filter). The kernel slides across the input, computing element-wise products and summing the results. This operation extracts local features—edges, textures, patterns—from different spatial locations.

Mathematically, the 2D convolution is:
```
(I * K)[i,j] = ∑∑ I[i+m, j+n] × K[m,n]
```

Where I is the input and K is the kernel.

## Filters and Feature Maps

Filters are learnable kernels that detect specific patterns. Early layers learn low-level features (edges, corners), while deeper layers combine these into high-level semantic features. Each filter produces a feature map—a 2D representation of where patterns occur in the input.

## Stride and Padding

**Stride** determines how many pixels the filter moves at each step. A stride of 1 produces dense feature maps; stride of 2 creates downsampling. This reduces computation and increases receptive field.

**Padding** adds border pixels (usually zeros) to the input. Valid padding discards boundary information, while same padding preserves spatial dimensions. Padding prevents information loss at edges and controls output dimensions.

## Pooling Operations

Pooling layers downsample feature maps by taking aggregate [statistics](/probability-eng.html) from small regions:

- **Max Pooling**: Extracts maximum value (most common)
- **Average Pooling**: Computes mean value
- **Stochastic Pooling**: Randomly samples based on [probability](/probability-eng.html)

Pooling reduces computation, prevents overfitting, and increases robustness to small spatial translations.

## Receptive Field

The receptive field measures how much of the original input influences an output neuron. It grows with network depth and filter size. Understanding receptive field is crucial for architecture design—larger fields capture context, while smaller fields focus on fine details.

Receptive field formula:
```
RF = previous_RF + (filter_size - 1) × stride_product
```

## Feature Map Visualization

Feature maps at different depths reveal hierarchical learning. Shallow layers detect edges and corners; middle layers combine these into textures and parts; deep layers recognize objects and scenes. This hierarchical representation explains CNN's effectiveness.

## Famous Architectures Overview

**AlexNet (2012)**: Pioneering deep CNN with 8 layers. Demonstrated that deep networks with ReLU and [dropout](/dropout-eng.html) outperformed shallow approaches.

**VGGNet (2014)**: Emphasized depth through small 3×3 filters. Simpler, more uniform architecture than AlexNet.

**ResNet (2015)**: Introduced [residual connections](/residual-connections-eng.html) enabling very deep networks (50-152 layers). [Skip connections](/residual-connections-eng.html) solve [vanishing gradient](/vanishing-gradients-eng.html) problem.

**Inception/GoogLeNet (2014)**: Multi-scale feature extraction using parallel pathways of different filter sizes.

**MobileNet (2017)**: Efficient architecture using depthwise separable convolutions for mobile/embedded devices.

**DenseNet (2016)**: Dense connections between all layers, improving gradient flow and reducing parameters.

## Modern Trends

Contemporary architectures focus on efficiency (MobileNets, ShuffleNets), hybrid approaches (combining CNNs with attention), and [Vision Transformers](/vision-transformers-eng.html). Yet convolutions remain fundamental—even [transformer](/llm-eng.html)-based vision models often incorporate convolutional components.

## Conclusion

CNNs succeed because they incorporate domain knowledge: spatial locality, translation invariance, and hierarchical composition. Understanding convolution, filters, padding, pooling, and receptive fields provides the foundation for modern [computer vision](/cv-eng.html). These concepts remain relevant even as architectures evolve.

