---
layout: default
title: Image Segmentation
lang: en
---

# Image Segmentation: Pixel-Level Classification

## Introduction

Image segmentation assigns class labels to individual pixels, enabling dense predictions. Three main types exist: semantic segmentation (class per pixel), instance segmentation (distinguishing individual objects), and panoptic segmentation (combining both). This task is fundamental for scene understanding in robotics, medical imaging, and autonomous driving.

## Semantic Segmentation

Assigns a class label to each pixel. Multiple pixels of the same class merge into single regions. Cannot distinguish between separate objects of the same class.

**FCN (Fully Convolutional Networks, 2014):**
- Replaces fully connected layers with convolutional layers
- Uses [skip connections](/residual-connections-eng.html) from earlier layers to combine coarse and fine features
- Output resolution matches input through upsampling
- Pioneering architecture enabling end-to-end pixel-wise prediction

**DeepLab Series:**
- **DeepLab v1-v3+:** Progressive improvements through atrous convolution and spatial pyramid pooling
- Atrous (dilated) convolution increases receptive field without losing resolution
- ASPP (Atrous Spatial Pyramid Pooling) captures multi-scale context
- Excellent performance on standard [benchmarks](/benchmarks-eng.html)

## Instance Segmentation

Distinguishes individual object instances while providing precise pixel-level masks. More challenging than semantic segmentation.

**Mask R-CNN (2017):**
- Extends Faster R-[CNN](/cnn-deep-dive-eng.html) with segmentation branch
- For each region proposal, generates binary mask predicting pixel membership
- Combines bounding box detection with instance masks
- Became standard approach, widely adopted in industry
- Fast Mask R-[CNN](/cnn-deep-dive-eng.html) variants available for real-time applications

**Other Approaches:**
- YOLACT: Speed-focused instance segmentation
- Panoptic Feature Pyramid Networks: Handling both thing and stuff classes

## Panoptic Segmentation

Unifies semantic and instance segmentation. Things (countable objects like people, cars) get instance IDs; stuff (background regions like sky, grass) get semantic labels only.

**Key Challenge:** Handling the heterogeneous nature of different object categories simultaneously.

## U-Net (2015)

Encoder-decoder architecture with [skip connections](/residual-connections-eng.html):
- Encoder: Convolutional layers progressively downsample
- Decoder: Upsampling layers with [skip connections](/residual-connections-eng.html) restore spatial resolution
- Widely adopted for medical image segmentation
- Small number of parameters despite good performance
- Highly influential in biomedical imaging community

## SAM (Segment Anything Model)

Recent foundation model enabling interactive segmentation:
- Takes prompts (points, boxes, text) to produce masks
- Trained on diverse dataset covering 1.1 billion masks
- Zero-shot generalization to new domains without fine-tuning
- Democratizes segmentation—users need not be ML experts
- Potential revolution in segmentation workflows

## Loss Functions

**Cross-Entropy:** Standard pixel-wise classification loss

**Dice Loss:** Directly optimizes IoU-like metric, robust to class imbalance

**Focal Loss:** Focuses on hard pixels, useful for imbalanced datasets

**Lovász-Hinge:** Surrogate for mIoU, differentiable relaxation

## Practical Considerations

- **Class Imbalance:** Background typically dominates; weighted losses and hard negative mining help
- **Multi-Scale Context:** Must capture information at multiple scales
- **Boundary Precision:** Post-processing and refinement networks improve edge accuracy
- **Computational Efficiency:** Real-time segmentation requires model optimization

## Evaluation Metrics

**mIoU (mean Intersection over Union):** Average IoU across classes, primary metric

**Pixel Accuracy:** Percentage of correctly classified pixels (can be misleading with imbalance)

**Boundary IoU:** Measures edge accuracy specifically

## Conclusion

Image segmentation progresses from semantic to instance to panoptic approaches, each adding complexity and value. Architectures like FCN, U-Net, and Mask R-[CNN](/cnn-deep-dive-eng.html) established foundations; newer models like SAM and diffusion-based approaches promise further advances. Understanding these paradigms enables informed decisions for dense prediction tasks.

