---
layout: default
title: Self-Supervised Learning
lang: en
---

# Self-Supervised Learning: Creating Targets from Data Itself

Self-[supervised learning](/supervised-learning-eng.html) represents a paradigm shift in machine learning. Instead of relying on expensive human annotations, it automatically generates training signals from the data structure itself. This enables models to learn powerful representations from vast unlabeled datasets, forming the foundation of modern foundation models.

## What is Self-Supervised Learning?

Self-[supervised learning](/supervised-learning-eng.html) creates artificial prediction tasks (pretext tasks) from unlabeled data, where the target is derived directly from the data itself. By solving these pretext tasks, models learn rich feature representations useful for downstream applications.

### The Core Idea

Split data artificially into input and target components. If a model can learn to predict one part from another, it must capture meaningful patterns. The learned representations generalize to other tasks.

**Examples:**
- Predict masked words from context ([BERT](/bert-eng.html))
- Predict image regions from surrounding patches (MAE)
- Match augmented views of the same image (SimCLR)

## Pretext Tasks

Pretext tasks are artificial prediction problems designed to train useful representations.

### Context Prediction
Predict missing elements from context. Language models predict next tokens; image models predict missing patches.

**Language:** [BERT](/bert-eng.html) masks 15% of input tokens and trains models to predict them from context. This forces the model to understand semantic relationships and linguistic structure.

**Vision:** Masked Autoencoders (MAE) mask large portions (75%) of image patches and reconstruct from visible patches, learning local and global structure.

### Rotation Prediction
Predict image rotation (0°, 90°, 180°, 270°). While seemingly simple, this task forces the model to understand object orientation and spatial structure.

### Colorization
Convert grayscale images to color. This task requires understanding object categories and semantic properties.

### Jigsaw Puzzle
Scramble image patches and predict their original arrangement. This forces understanding of spatial relationships and object structure.

## Contrastive Learning: The Powerhouse

Contrastive learning represents a breakthrough approach where representations are learned by comparing similar and dissimilar examples.

### Core Principle
Maximize agreement between different augmented views of the same image while minimizing agreement between views of different images.

**Mathematical Goal:** For positive pairs (two augmentations of same image), maximize similarity. For negative pairs (augmentations from different images), minimize similarity.

### SimCLR (Simple Contrastive Learning of Representations)

SimCLR is elegant in its simplicity yet remarkably effective.

**Algorithm:**
1. Apply two stochastic augmentations (crop, color jitter, rotation) to each image → x₁, x₂
2. Encode both through shared encoder → z₁ = f(x₁), z₂ = f(x₂)
3. Project to lower dimension → h₁ = g(z₁), h₂ = g(z₂)
4. Compute contrastive loss (NT-Xent: normalized temperature-scaled cross-[entropy](/information-theory-eng.html))
5. Maximize similarity between h₁ and h₂; minimize with others in batch

**Why It Works:** Two augmentations of the same image share semantic content but differ in low-level details. The model learns to ignore superficial differences and focus on semantic features.

**Key Components:**
- **Strong augmentations:** Essential; without augmentation diversity, models ignore image content
- **Projection head:** Projecting to lower dimension before loss improves learned representations
- **Large batch sizes:** Need many negatives for effective contrastive learning
- **Temperature parameter:** Controls loss concentration

### Momentum Contrast (MoCo)

MoCo improves upon SimCLR by maintaining a momentum-updated queue of encoded examples.

**Innovation:** Instead of requiring huge batch sizes, MoCo uses a queue and momentum encoder:
- Encoder weights updated by [gradient descent](/calculus-eng.html)
- Momentum encoder updated as slow-moving average of main encoder
- Queue stores encoded examples from recent batches

**Advantage:** Works well with smaller batch sizes; more practical.

**Refinement:** MoCo v2 combines insights from SimCLR, achieving state-of-the-art results.

### BYOL (Bootstrap Your Own Latent)

BYOL removes the need for negative pairs—a surprising finding that challenged conventional wisdom.

**Key Insight:** Don't need explicit negatives; instead, use target network (EMA of main network) and prevent representation collapse through predictor networks.

**Algorithm:**
1. Encode augmented image through main network
2. Encode different augmentation through target network (EMA update)
3. Maximize agreement between main prediction and target representation
4. Predictor prevents trivial solution of all embeddings being identical

**Revolutionary:** Questions whether negatives are necessary; suggests the learning objective itself provides sufficient constraint.

## Masked Prediction Methods

Masked prediction continues the tradition of context prediction at scale.

### BERT (Bidirectional Encoder Representations from Transformers)

[BERT](/bert-eng.html) masks 15% of input tokens and trains [transformers](/llm-eng.html) to predict masked tokens from bidirectional context. This self-supervised pre-training transfers remarkably well to downstream NLP tasks.

**Masking Strategy:**
- 80% of time: replace with [MASK] token
- 10% of time: replace with random token
- 10% of time: keep original

This randomization forces the model to learn robust representations.

### Masked Autoencoders (MAE)

MAE extends [BERT](/bert-eng.html)'s masking to vision. Critical innovation: mask 75% of patches (vs. BERT's 15% tokens).

**Asymmetric Design:**
- Encoder sees only visible patches (25%)
- Decoder reconstructs all patches from encoder output
- Asymmetry is key—encoding only visible patches forces the decoder to learn meaningful features

**Result:** Teaches rich visual understanding; excellent transfer to downstream tasks.

### Generative Pretraining

[Large language models](/llm-eng.html) like [GPT](/gpt-eng.html) learn by predicting next tokens. This simple objective, applied at massive scale, produces models with remarkable emergent capabilities—few-shot learning, reasoning, [code generation](/code-generation-eng.html).

## DINO: Emerging without Negatives or Momentum

DINO (self-supervised [Vision Transformer](/vision-transformers-eng.html) from a Teacher model) removes several architectural components while maintaining or improving performance.

**Key Components:**
1. [Vision Transformer](/vision-transformers-eng.html) backbone
2. No negative examples
3. No momentum encoder
4. Student and teacher networks (teacher is EMA of student)
5. Sharpened softmax (small temperature)

**Process:** Student tries to predict teacher's output on different augmentations. Temperature in softmax creates sharp distributions, forcing concentrated predictions.

**Remarkable Finding:** [Self-attention](/self-attention-eng.html) maps in [ViT](/vision-transformers-eng.html) trained with DINO naturally emerge with semantic meaning—no explicit segmentation labels.

## Relationship to Foundation Models

Foundation models are large models trained on vast unlabeled data with self-[supervised learning](/supervised-learning-eng.html), then fine-tuned for diverse downstream tasks.

**Why Self-Supervised Learning Enables Foundation Models:**
1. **Scale:** No labeling bottleneck; can use all available data
2. **Transfer:** Learned representations generalize across domains and tasks
3. **Efficiency:** Fine-tuning on modest labeled data achieves strong performance
4. **Emergence:** Large-scale self-[supervised learning](/supervised-learning-eng.html) reveals unexpected capabilities

**Examples:**
- **Vision:** CLIP (contrastive text-image), [ViT](/vision-transformers-eng.html) with DINO
- **Language:** [BERT](/bert-eng.html), [GPT](/gpt-eng.html) family, T5
- **Multimodal:** Foundation models combining text, vision, audio

## Why Self-Supervised Learning Powers Modern AI

Self-[supervised learning](/supervised-learning-eng.html) solved a critical bottleneck: annotation scarcity. It enables leveraging the vast majority of internet-scale data that lacks explicit labels.

### Advantages:
1. **Data Efficiency:** Learns from unlabeled data at scale
2. **Generalization:** Representations learned unsupervised often outperform supervised baselines when transferred
3. **No Label Bias:** Avoids systematic biases in human annotation
4. **Emergent Properties:** Large models show unexpected capabilities

### Challenges:
1. **Hyperparameter Sensitivity:** Many design choices (augmentation, temperature, batch size)
2. **Computational Cost:** Requires large computational resources and data
3. **Theoretical Understanding:** Why certain pretext tasks work well remains partially mysterious
4. **Domain Specificity:** Task design may need customization for different domains

## The Foundation Model Era

Self-[supervised learning](/supervised-learning-eng.html) powers the current foundation model era:
- [Transformers](/llm-eng.html) pre-trained on massive unlabeled data
- Fine-tuned for numerous downstream applications
- Remarkably few parameters adapted per task
- Strong zero-shot and few-shot performance

This paradigm has transformed AI, enabling systems like [GPT](/gpt-eng.html)-4, Claude, DALL-E, and Gemini to learn powerful world models with minimal task-specific supervision.

Self-[supervised learning](/supervised-learning-eng.html) isn't just a technique—it's a fundamental shift in how we approach machine learning in the era of abundant data and scarce labels.

