---
layout: default
title: Information Theory for AI
lang: en
---

# Information Theory for AI

Information theory is the mathematical foundation for understanding data, uncertainty, and learning. Claude Shannon revolutionized this field in 1948, providing tools essential to modern machine learning. This guide explores key concepts that bridge information theory and AI systems.

## Entropy: Measuring Uncertainty

Entropy quantifies the expected amount of information in a random variable. For a discrete variable X with probability distribution P:

**H(X) = -∑ P(x) log₂ P(x)**

Higher entropy means more unpredictability. A fair coin has maximum entropy (1 bit), while a certain outcome has zero entropy. In machine learning, entropy guides decision trees: we split features that reduce entropy most, capturing the most informative decisions.

Consider a biased coin landing heads with probability 0.9. Its entropy is:
- H = -0.9 log₂(0.9) - 0.1 log₂(0.1) ≈ 0.47 bits

This is much lower than a fair coin (1 bit), reflecting our greater certainty about outcomes.

## Cross-Entropy: Measuring Distribution Distance

Cross-entropy measures the average length of messages needed to encode data from one distribution (true) using a code optimized for another (predicted):

**H(P, Q) = -∑ P(x) log Q(x)**

Where P is the true distribution and Q is our model's predicted distribution. When P = Q, cross-entropy equals entropy (optimal encoding). When they differ, we need extra bits.

In classification, cross-entropy loss compares true class labels (P) with predicted probabilities (Q). A model predicting 0.9 probability for the correct class achieves -log(0.9) ≈ 0.105 loss, while predicting 0.1 incurs -log(0.1) ≈ 2.30 loss—heavily penalizing confident wrong predictions.

## KL Divergence: Asymmetric Distance

The Kullback-Leibler (KL) divergence measures how one probability distribution diverges from another:

**D_KL(P||Q) = ∑ P(x) log(P(x)/Q(x))**

This is **asymmetric**: D_KL(P||Q) ≠ D_KL(Q||P). Forward KL (used in variational inference) penalizes Q placing probability where P has none. Reverse KL penalizes Q missing probability mass where P concentrates.

Key insight: KL divergence decomposes as H(P, Q) - H(P). Minimizing cross-entropy is equivalent to minimizing KL divergence plus a constant term. This explains why cross-entropy loss is ubiquitous: it naturally aligns model predictions with true distributions.

## Mutual Information: Dependence Measure

Mutual information I(X; Y) quantifies how much knowing one variable reduces uncertainty about another:

**I(X; Y) = ∑∑ P(x,y) log(P(x,y)/(P(x)P(y)))**

Also expressible as: I(X; Y) = H(X) - H(X|Y) = H(Y) - H(Y|X)

High mutual information means X and Y are strongly dependent. Zero mutual information indicates independence. For AI, mutual information guides feature selection—features with high mutual information with the target carry maximum predictive power.

## Information Gain: The Heart of Decision Trees

Information gain measures how much a feature reduces entropy when splitting data:

**IG(S, A) = H(S) - ∑(|S_v|/|S|) * H(S_v)**

Where S is the dataset, A is the attribute we split on, and S_v are subsets after splitting. Decision trees greedily select splits maximizing information gain, recursively partitioning data into increasingly pure classes.

For example, splitting a balanced dataset of 100 samples (50 positive, 50 negative) with H(S) = 1 bit on a feature creating two pure subsets (all positive, all negative) yields:
- IG = 1 - 0 = 1 bit

Maximum information gain occurs with perfect splits. In practice, we choose splits among imperfect options, always preferring features providing the greatest entropy reduction.

## Cross-Entropy Loss in Deep Learning

Neural networks for classification minimize cross-entropy loss:

**Loss = -∑ y_i log(ŷ_i)**

Where y_i are true one-hot encoded labels and ŷ_i are predicted probabilities. This formulation directly implements the information-theoretic principle: align predicted distributions (from softmax) with true distributions (labels).

For multi-class problems with K classes:
- Loss = -log(ŷ_correct_class)

A model approaching 100% confidence on correct class drives loss to zero. Conversely, low confidence incurs high penalty. This asymmetry provides strong learning signal—the gradient ∂Loss/∂ŷ is most dramatic when predictions are most wrong.

Binary cross-entropy for logistic regression uses the same principle with one output neuron:
- Loss = -[y*log(ŷ) + (1-y)*log(1-ŷ)]

## Relationship to ML Loss Functions

Information theory elegantly unifies diverse loss functions:

1. **Regression (MSE)**: Equivalent to maximum likelihood under Gaussian noise with fixed variance
2. **Classification (Cross-entropy)**: Assumes categorical distribution, directly implementing Shannon's compression principle
3. **Divergence-based losses** (JS divergence, Wasserstein): Measure distributional mismatch
4. **Contrastive losses**: Implicitly maximize mutual information between related samples

Understanding losses through information theory reveals why they work. Cross-entropy incentivizes models to learn true data distributions. KL divergence explains regularization—constraining model distributions toward priors.

## Data Compression and Optimal Codes

Shannon's source coding theorem states: the expected message length cannot be shorter than entropy H(X), but can approach it arbitrarily closely with optimal encoding.

This connects directly to deep learning. A model learning data distribution effectively compresses it. By minimizing cross-entropy loss, we're finding codes requiring fewest bits. Regularization prevents overfitting by limiting code complexity—a model using fewer parameters is like enforcing longer message lengths.

The compression principle explains why neural networks generalize: they must capture essential patterns in data to compress it. Noise-specific patterns require more bits to encode and are naturally discarded during training.

## Rate-Distortion Theory and Representation Learning

Rate-distortion theory extends compression: at what information rate can we represent data with acceptable distortion?

Information bottleneck framework applies this to deep learning: hidden layers balance information preservation about inputs (rate) against information about targets (relevance). Early layers compress input information while preserving target information—a fundamental principle explaining representation learning.

## Practical Implications for AI Engineers

1. **Feature Engineering**: Select features maximizing mutual information with targets
2. **Model Selection**: Compare models via their cross-entropy loss on test sets
3. **Regularization**: Penalize model complexity (higher entropy) to prevent overfitting
4. **Calibration**: Post-training probability calibration uses KL divergence to align predictions with empirical frequencies
5. **Dataset Design**: Maximize entropy diversity in training data for better generalization

## Conclusion

Information theory provides the principled language for machine learning. Entropy measures uncertainty we face, cross-entropy guides learning objectives, KL divergence quantifies distributional misalignment, and mutual information identifies relevant features. Modern AI systems optimize information-theoretic objectives, making deep understanding of these concepts invaluable for designing better algorithms and understanding why they work.
