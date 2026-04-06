---
layout: default
title: Semi-Supervised Learning
lang: en
---

# Semi-Supervised Learning: Leveraging Unlabeled Data

Semi-supervised learning bridges supervised and unsupervised learning, combining small amounts of labeled data with large amounts of unlabeled data. In practice, this is one of the most relevant learning paradigms because obtaining labels is expensive while unlabeled data is abundant.

## The Label Scarcity Problem

Real-world applications face a critical challenge: **label scarcity**. Medical imaging, autonomous driving, and content moderation require expensive expert annotation. Meanwhile, we have access to massive amounts of unlabeled data—images, text, sensor data—that could inform our models if we knew how to leverage it.

Semi-supervised learning addresses this bottleneck by using unlabeled data to improve model performance beyond what supervised learning alone can achieve. The key assumption: the structure of unlabeled data reveals patterns that help classification even without explicit labels.

## Self-Training

Self-training iteratively labels unlabeled data using the model's own predictions.

**Process:**
1. Train a model on small labeled dataset
2. Use model to predict labels for unlabeled data
3. Select confident predictions (high confidence threshold)
4. Add pseudo-labeled examples to training set
5. Retrain model on expanded dataset
6. Repeat steps 2-5 until convergence or enough pseudo-labels obtained

**Strength:** Simple, intuitive, works with any base classifier.

**Weakness:** Confirmation bias—if initial model is wrong, it reinforces errors on pseudo-labeled examples. Error accumulation over iterations can degrade performance.

**Mitigation:** Use threshold confidence filtering, periodic manual validation, or ensemble predictions.

## Co-Training

Co-training uses multiple learners trained on different feature subsets or views of the data.

**Process:**
1. Split features into two conditionally independent subsets
2. Train two classifiers, one on each view
3. Each classifier makes high-confidence predictions on unlabeled data
4. Exchange these pseudo-labeled examples between classifiers
5. Retrain both classifiers with pseudo-labeled data
6. Repeat until convergence

**Key Idea:** Different views provide diverse information. Disagreement between classifiers flags uncertain examples; agreement provides confidence.

**Example:** For text classification, one classifier might use word features while another uses syntactic structure.

**Advantage:** Reduces bias from single model; leverages data redundancy.

**Limitation:** Assumes feature independence; requires finding meaningful feature splits.

## Graph-Based Methods

Graph-based approaches treat data as nodes in a graph, where edges represent similarity. Labels propagate through the graph from labeled to unlabeled nodes.

### Label Propagation
Label propagation assigns unlabeled examples the class of their most influential neighbors, weighted by edge strength.

**Algorithm:** Initialize graph with labeled node values, iteratively propagate labels through edges, stop when labels converge.

**Intuition:** Similar nodes should have similar labels. Propagation enforces consistency.

**Advantage:** Simple, mathematically elegant, respects data geometry.

**Challenge:** Sensitive to graph construction; computationally expensive for large graphs.

### Harmonic Fields and Potential Functions
Model label propagation as solving Laplace's equation on the graph—unlabeled nodes' labels are harmonic functions of labeled nodes' labels.

**Mathematical Foundation:** Minimize energy functional encoding smoothness and label fitting simultaneously.

## Pseudo-Labeling

Pseudo-labeling directly assigns predicted labels to unlabeled examples and retrains the model on the combined set.

**Approach:**
1. Train on labeled data
2. Predict labels for all unlabeled data
3. Combine original labeled data with all pseudo-labeled data
4. Retrain on combined dataset

**Difference from Self-Training:** Self-training filters by confidence; pseudo-labeling includes all predictions.

**Benefit:** Maximum data utilization; can significantly boost performance.

**Risk:** Low-confidence errors propagate; requires careful threshold selection.

**Modern Enhancement:** Use model uncertainty estimates or temperature-based filtering.

## Consistency Regularization

Consistency regularization enforces that predictions remain stable under data perturbations. If we slightly perturb an unlabeled example (add noise, apply augmentation), the model should make the same prediction.

**Mathematical Form:** Minimize supervised loss on labeled data plus consistency loss on unlabeled data:

Loss = L_labeled + λ * L_consistency

where L_consistency measures prediction differences under perturbations.

**Intuition:** Unlabeled data constrains decision boundaries. Predictions should be robust to small input variations.

**Practical Implementation:**
- Apply augmentations to unlabeled examples
- Make predictions on both original and augmented versions
- Minimize divergence (KL divergence, mean squared difference) between predictions

**Power:** Leverages manifold assumption—decision boundary should lie in low-density regions.

## MixMatch

MixMatch is a state-of-the-art semi-supervised algorithm combining multiple techniques.

**Components:**
1. **Consistency Regularization:** Augment unlabeled examples, enforce prediction consistency
2. **Entropy Minimization:** Encourage model to make confident predictions
3. **MixUp:** Mix labeled and unlabeled examples in feature space, linearly interpolate targets

**MixUp Operation:** For examples x and x', mix with parameter λ:
x_mixed = λx + (1-λ)x'
y_mixed = λy + (1-λ)y'

**Advantage:** Combines complementary techniques; achieves excellent semi-supervised results.

**Computational Cost:** Requires multiple augmentations and forward passes.

## FixMatch

FixMatch simplifies semi-supervised learning with a single intuitive idea: model predictions under weak augmentation should match pseudo-labels from strong augmentation.

**Algorithm:**
1. Weak augmentation (flip, shift)—preserves true label
2. Strong augmentation (RandAugment, CutOut, etc.)—may change true label
3. Model predicts on weak augmentation
4. Pseudo-label from strong augmentation prediction
5. Loss = supervised loss on labeled data + consistency loss between predictions

**Insight:** Strong augmentation removes superficial cues, forcing the model to rely on semantic features.

**Elegance:** Simple, interpretable, highly effective.

**Scalability:** Efficient, outperforms complex methods on benchmark datasets.

## Applications

**Medical Imaging:** Limited expert-annotated images paired with hospital archives. Semi-supervised learning enables diagnosis systems with modest labeled data.

**Content Moderation:** Few manually-reviewed cases guide models trained on millions of automatic reviews.

**Autonomous Driving:** Limited annotated driving sequences combined with abundant raw sensor data.

**Natural Language Processing:** Modest labeled datasets combined with vast web text.

**Speech Recognition:** Small transcribed audio combined with recorded but untranscribed speech.

## Why Semi-Supervised Learning Matters

As datasets grow and annotation becomes expensive, semi-supervised learning becomes essential. It reflects realistic constraints: we often have signal but not ground truth. By leveraging unlabeled data structure, semi-supervised methods unlock performance gains previously requiring complete annotation.

The evolution from simple self-training to sophisticated methods like MixMatch and FixMatch shows the field's maturation. These techniques now form the backbone of practical ML systems handling real-world label scarcity.

