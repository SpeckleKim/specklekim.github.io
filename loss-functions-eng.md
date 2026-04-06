---
layout: default
title: Loss Functions
lang: en
---

# Loss Functions: Measuring Model Error

Loss functions quantify the disagreement between model predictions and ground truth labels. Choosing the right loss function is crucial—it directly shapes what the model optimizes for and ultimately determines which solutions are learned.

## Mean Squared Error (MSE)

MSE is the most common loss for regression:

`MSE = (1/m) * Σ(y_true - y_pred)²`

**Characteristics**: Penalizes large errors more than small ones (quadratic penalty). Smooth gradient everywhere.

**Advantages**: Simple, differentiable, convex.

**Disadvantages**: Sensitive to outliers. Predictions far from truth are heavily penalized.

**Use case**: Standard regression tasks. When outliers are rare or should be heavily weighted.

## Mean Absolute Error (MAE)

MAE computes average absolute differences:

`MAE = (1/m) * Σ|y_true - y_pred|`

**Characteristics**: Linear penalty for all errors. Robust to outliers.

**Advantages**: Outlier-resistant. Interpretable in original data units.

**Disadvantages**: Non-smooth at zero (gradient undefined). Less sensitive to very small errors.

**Use case**: Regression with outliers. When errors of any size matter equally.

## Huber Loss

Huber combines MSE and MAE, being quadratic for small errors and linear for large ones:

`Huber = (1/m) * Σ L_δ(y - y_pred)` where L depends on residual magnitude and δ threshold

**Characteristics**: Hybrid approach. Smooth everywhere. Robust yet sensitive to small errors.

**Advantages**: Best of both worlds—robust to outliers yet penalizes small errors.

**Disadvantages**: Hyperparameter δ to tune.

**Use case**: Regression with occasional outliers. Robust learning preferred.

## Binary Cross-Entropy

For binary classification:

`BCE = -(1/m) * Σ[y*log(ŷ) + (1-y)*log(1-ŷ)]`

**Characteristics**: Penalizes confident wrong predictions heavily. Natural for [probability](/probability-eng.html)-based outputs.

**Advantages**: Aligns with probabilistic interpretation. Strong gradients far from target.

**Disadvantages**: Numerically unstable if not using log-sum-exp trick. Undefined for ŷ=0 or 1.

**Use case**: Binary classification with sigmoid output.

## Categorical Cross-Entropy

For multi-class classification:

`CCE = -(1/m) * Σ Σ y_k*log(ŷ_k)`

**Characteristics**: Generalization of BCE. Works with softmax output.

**Advantages**: Natural for multi-class problems. Standard choice.

**Disadvantages**: Requires one-hot encoded labels. Numerically sensitive.

**Use case**: Multi-class classification. Standard approach for most classification tasks.

## Focal Loss

Focal loss addresses class imbalance by down-weighting easy examples:

`FL = -(1/m) * Σ (1 - ŷ_t)^γ * log(ŷ_t)`

where γ is a focusing parameter.

**Characteristics**: Hard-example mining via multiplicative loss weighting. Adaptive emphasis.

**Advantages**: Handles severe class imbalance. Better than BCE when classes are imbalanced.

**Disadvantages**: Hyperparameter γ to tune. More complex.

**Use case**: Imbalanced datasets. [Object detection](/object-detection-eng.html) where background dominates.

## Hinge Loss

Hinge loss is standard for [support vector machines](/svm-eng.html) and margin-based learning:

`Hinge = (1/m) * Σ max(0, 1 - y_true * y_pred)`

**Characteristics**: Margin-based. Treats correctly classified examples beyond margin as zero loss.

**Advantages**: Natural margin interpretation. Sparse solutions.

**Disadvantages**: Non-smooth at margin. Not directly probabilistic.

**Use case**: [SVM](/svm-eng.html)-like problems. Learning with explicit margins.

## Contrastive Loss

Contrastive loss learns embeddings by comparing pairs of examples:

`Contrastive = (1/m) * Σ [y*d² + (1-y)*max(0, m-d)²]`

where d is distance and y indicates if pair is similar.

**Characteristics**: Learns similarity metric. Useful for metric learning.

**Advantages**: Interpretable similarity. Flexible for various tasks.

**Disadvantages**: Requires paired data. Computationally expensive.

**Use case**: Siamese networks. [Face recognition](/face-recognition-eng.html). Similarity learning.

## Triplet Loss

Triplet loss compares anchor, positive, and negative examples:

`Triplet = max(0, d(a,p) - d(a,n) + margin)`

**Characteristics**: Encourages separation between similar and dissimilar examples.

**Advantages**: More expressive than contrastive. Better for ranking.

**Disadvantages**: Requires triplet mining. Computationally intensive.

**Use case**: Metric learning at scale. Face verification. Person re-identification.

## CTC Loss (Connectionist Temporal Classification)

CTC loss handles [sequence-to-sequence](/seq2seq-eng.html) problems where alignment is unknown:

`CTC = -log(Σ_paths P(path | inputs))`

**Characteristics**: Marginalizes over all possible alignments.

**Advantages**: No need for aligned labels. Handles variable-length sequences.

**Disadvantages**: Complex to implement. Slower than aligned loss.

**Use case**: Speech recognition. [OCR](/ocr-eng.html). Tasks where label alignment is unknown.

## Custom Losses

Domain-specific losses can be designed for specialized objectives. Examples include:

- **Dice Loss**: Useful in [semantic segmentation](/image-segmentation-eng.html) with class imbalance
- **IoU Loss**: Matches evaluation metric in [object detection](/object-detection-eng.html)
- **Perceptual Loss**: Uses pre-trained networks to measure semantic similarity
- **Adversarial Loss**: Used in [GANs](/gan-eng.html) to measure discriminator vs. generator

**Design principles**: Match loss to evaluation metric when possible. Ensure gradients are informative.

## Loss Selection Guide

**Regression**:
- Default: MSE
- With outliers: Huber or MAE
- Custom range: Adjusted MSE/MAE

**Classification**:
- Binary: Binary cross-[entropy](/information-theory-eng.html)
- Multi-class: Categorical cross-[entropy](/information-theory-eng.html)
- Imbalanced: Focal loss
- Metric learning: Contrastive or triplet

**Sequence tasks**:
- Alignment unknown: CTC loss
- Word-level: Cross-[entropy](/information-theory-eng.html)
- Custom alignment: Custom CTC variants

## Summary

Loss functions are the objective that models optimize. Selecting the right loss is as important as architecture choice. Modern practice often involves loss combinations (multi-task learning) and careful weighting to balance competing objectives. Understanding loss properties—smoothness, sensitivity, computational cost—is essential for effective model training.
