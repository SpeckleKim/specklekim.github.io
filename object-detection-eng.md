---
layout: default
title: Object Detection
lang: en
---

# Object Detection: Localizing and Classifying Objects

## Introduction

Object detection extends image classification by identifying multiple objects and their locations. Modern approaches divide into two categories: two-stage detectors that propose regions first, and one-stage detectors that directly predict bounding boxes. Each strategy offers different trade-offs between accuracy and speed.

## Two-Stage Detectors

Two-stage detectors separate detection into region proposal and classification stages, typically achieving higher accuracy.

**R-CNN Family Evolution:**
- **R-CNN (2014)**: Uses selective search for region proposals, feeds each to CNN for features, then classifies with SVM
- **Fast R-CNN (2015)**: RoI pooling layer enables sharing computation across proposals
- **Faster R-CNN (2015)**: Replaces selective search with Region Proposal Network (RPN), dramatically improving speed
- **Cascade R-CNN (2017)**: Cascaded refinement of detections across multiple quality thresholds
- **Mask R-CNN (2017)**: Extends Faster R-CNN with instance segmentation branch

## One-Stage Detectors

One-stage detectors predict bounding boxes directly from feature maps, trading some accuracy for speed.

**YOLO Series:**
- **YOLOv1 (2016)**: Pioneering approach treating detection as regression problem on spatial grid
- **YOLOv2 (2016)**: Batch normalization, multi-scale predictions, anchor boxes
- **YOLOv3-v8 (2018-2023)**: Progressively improved with better backbones and training strategies
- **YOLOv9**: Advanced scaling and efficiency innovations

**SSD (Single Shot MultiBox Detector, 2015):** Uses multi-scale feature maps for detecting objects at different scales.

## Anchor-Based vs Anchor-Free

**Anchor-Based:** Pre-defined bounding box templates at each location. Requires careful anchor design but provides useful priors.

**Anchor-Free:** FCOS, CenterNet, keypoint-based approaches directly predict object centers and dimensions, reducing design choices and hyperparameters.

## DETR and Vision Transformers

**DETR (2020):** Detection Transformer treats detection as set prediction problem. Encoder-decoder architecture with learnable queries for objects. Eliminates hand-designed components like NMS, enabling end-to-end training.

**Advantages:** Conceptually clean, set-based loss avoiding NMS
**Disadvantages:** Slower convergence, requires more data than CNN-based detectors

## RT-DETR

Efficient DETR variant optimized for real-time detection. Combines transformer strengths with CNN efficiency, achieving competitive speed-accuracy trade-offs for practical deployment.

## Evaluation Metrics

**mAP (mean Average Precision):**
- Average Precision (AP) computed for each class
- IoU (Intersection over Union) threshold determines positive predictions (typically 0.5)
- mAP@0.5 uses single threshold; mAP@[0.5:0.95] averages across IoU thresholds
- Standard metric for comparing detector performance

**Other Metrics:**
- **Precision/Recall curves:** Trade-off between false positives and negatives
- **FPS (Frames Per Second):** Speed measurement
- **Latency:** Critical for real-time applications

## Modern Trends

Contemporary detectors increasingly adopt transformer architectures, hybrid approaches combining CNNs and attention, and efficient designs for deployment. Recent work focuses on:

- Few-shot and zero-shot detection
- Cross-domain generalization
- Efficient detectors for edge devices
- Self-supervised pre-training

## Practical Considerations

When choosing detectors: Consider target accuracy (mAP), speed requirements (FPS/latency), computational resources, and deployment platform. Two-stage detectors excel in accuracy; one-stage dominate speed. Transformers offer conceptual clarity at computational cost.

## Conclusion

Object detection remains a cornerstone of computer vision. Understanding architectural choices—two-stage vs one-stage, anchor-based vs anchor-free, CNN vs transformer—enables informed decisions for specific applications. The field continues evolving toward more efficient and generalizable approaches.

