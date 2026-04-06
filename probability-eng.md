---
layout: default
title: Probability & Statistics for AI
lang: en
---

# Probability & Statistics for AI

Probability and statistics form the mathematical backbone of machine learning. From decision-making under uncertainty to training probabilistic models, understanding probability distributions, Bayesian inference, and statistical hypothesis testing is crucial for effective AI development.

## Probability Basics

**Probability** measures the likelihood of an event occurring. It ranges from 0 (impossible) to 1 (certain). Probability theory provides the foundation for reasoning about uncertainty in data and models.

**Key concepts:**
- **Sample space (Ω)**: all possible outcomes
- **Event**: a subset of the sample space
- **Probability of event A**: P(A) = (favorable outcomes) / (total outcomes)

In AI, we often deal with **continuous probability distributions** where probability is defined over intervals using probability density functions (PDFs).

## Conditional Probability and Independence

**Conditional probability** is the probability of event A given that event B has occurred: **P(A|B) = P(A∩B) / P(B)**

This is fundamental to reasoning about cause and effect. For example, in spam detection, we care about P(spam | email content).

**Independence**: Events A and B are independent if **P(A|B) = P(A)**, meaning B provides no information about A.

**Chain rule**: **P(A,B,C) = P(A)·P(B|A)·P(C|A,B)**

Breaking down joint probabilities enables computing complex probabilities from simpler conditional ones.

## Bayes' Theorem

Bayes' Theorem is central to probabilistic inference: **P(A|B) = P(B|A)·P(A) / P(B)**

In machine learning terms:
- **P(A|B)**: posterior probability (what we want to compute)
- **P(B|A)**: likelihood (how well does the model explain the data?)
- **P(A)**: prior probability (what we believed before seeing data)
- **P(B)**: evidence (probability of the data)

Bayes' Theorem enables **Bayesian inference**: updating beliefs given new evidence. As we collect more data, posteriors become more confident, concentrating around the true value.

## Common Probability Distributions

**Gaussian (Normal) Distribution**: The most important distribution in AI and statistics. Defined by mean μ and variance σ². Bell-shaped curve describes phenomena like heights, measurement errors, and neural network activations.

**Bernoulli Distribution**: Models binary outcomes (heads/tails, true/false). Parameterized by p (probability of success). Used in binary classification.

**Categorical Distribution**: Generalization of Bernoulli to multiple outcomes. Each category has probability p_i. Used in multiclass classification.

**Poisson Distribution**: Models count data (number of events in fixed time). Defined by parameter λ (rate). Used for event counting, arrival processes.

**Exponential Distribution**: Models time until next event (waiting times). Used in survival analysis and duration modeling.

**Uniform Distribution**: Equal probability for all outcomes in a range. Used as a default prior when lacking domain knowledge.

Many AI algorithms assume data comes from specific distributions (e.g., Gaussian Naive Bayes assumes features follow Gaussian distributions).

## Expectation and Variance

**Expectation (Mean)**: The average value a random variable takes: **E[X] = Σ x·P(x)** (discrete) or **E[X] = ∫ x·p(x)dx** (continuous)

Expectation is linear: **E[aX + b] = a·E[X] + b**

**Variance**: Measures spread around the mean: **Var(X) = E[(X - E[X])²]**

High variance means values are spread out; low variance means they cluster around the mean.

**Standard deviation**: σ = √Var(X), in same units as X.

**Covariance**: **Cov(X,Y) = E[(X - E[X])·(Y - E[Y])]** measures how two variables move together.

**Correlation**: Normalized covariance: **ρ = Cov(X,Y) / (σ_X · σ_Y)**, ranges from -1 to +1.

## Maximum Likelihood Estimation (MLE)

MLE finds parameter values that make observed data most probable. Given data D and parameters θ:

**L(θ|D) = P(D|θ)** (likelihood function)

MLE selects: **θ_MLE = argmax_θ P(D|θ)**

Often we maximize log-likelihood (numerically stable): **θ_MLE = argmax_θ Σ log P(d_i|θ)**

**Why use MLE?**
- Principled approach to parameter estimation
- Under certain conditions, MLE estimators are consistent and asymptotically normal
- Powers logistic regression, Gaussian Naive Bayes, and many other models

## Maximum A Posteriori (MAP)

MAP combines likelihood with prior knowledge: **θ_MAP = argmax_θ P(θ|D) = argmax_θ P(D|θ)·P(θ)**

This is Bayesian inference: balancing what the data says (likelihood) with what we believed before (prior).

When prior P(θ) is uniform, MAP equals MLE. Strong priors can prevent overfitting by pulling estimates toward expected values.

## Hypothesis Testing

Hypothesis testing provides a framework for making statistical decisions:

1. **Null hypothesis (H₀)**: baseline assumption (e.g., "no effect")
2. **Alternative hypothesis (H₁)**: what we suspect (e.g., "there is an effect")
3. **Test statistic**: computed from data
4. **P-value**: probability of observing data as extreme or more extreme than observed, assuming H₀
5. **Significance level (α)**: threshold (commonly 0.05)

If p-value < α, we reject H₀. Small p-values provide evidence against the null hypothesis.

**Type I error**: rejecting H₀ when it's true (false positive)
**Type II error**: failing to reject H₀ when it's false (false negative)

## Applications in Machine Learning

**Classification**: Use Bayes' Theorem for probabilistic classifiers. Naive Bayes assumes feature independence given the class.

**Regression**: MLE estimation fits parameters of linear and logistic regression models.

**Uncertainty quantification**: Bayesian models output probability distributions over predictions, not just point estimates.

**Model selection**: Cross-validation and information criteria (AIC, BIC) use likelihood-based reasoning.

**Generative models**: VAEs and GANs explicitly model probability distributions of data.

**Reinforcement learning**: MDP theory is fundamentally probabilistic; agents reason about state transitions and rewards probabilistically.

Probability and statistics transform uncertain, incomplete information into principled decisions. They enable machines to learn from data while quantifying confidence in their predictions.
