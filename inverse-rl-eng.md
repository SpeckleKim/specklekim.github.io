---
layout: default
title: Inverse Reinforcement Learning
lang: en
---

# Inverse Reinforcement Learning: Learning Rewards from Demonstrations

## Introduction

Inverse [Reinforcement Learning](/rl-eng.html) (IRL) reverses the typical RL problem: given demonstrations of expert behavior, infer the reward function. Rather than hand-engineering reward functions (often difficult, error-prone), IRL learns from human expertise. Applications span autonomous driving, robotic manipulation, and imitation learning.

## Problem Formulation

**Standard RL:** Find policy π maximizing reward function R.

**IRL:** Given expert demonstrations D, infer reward function R such that expert policy optimizes it.

**Challenge:** Multiple reward functions could explain observed behavior. IRL seeks reward consistent with demonstrations, ideally generalizing to new situations.

## Maximum Entropy IRL

**Core Principle:** Among all reward functions consistent with demonstrations, choose maximum [entropy](/information-theory-eng.html) policy.

**Intuition:** Don't assume expert is optimizing any single reward function; assume observed actions are randomized among good-enough policies.

**Mathematical Framework:**
```
max_R H(π_R) subject to:
E_π_R[f(s,a)] = E_π_E[f(s,a)]
where f represents feature expectations
```

**Feature Matching:**
Expert policy and learned policy have same expected feature counts.

**Algorithm:**
```
1. Initialize reward function
2. Compute optimal policy under current reward
3. Compute feature expectations for both policies
4. Update reward to increase feature matching
5. Iterate until convergence
```

## Apprenticeship Learning

**Idea:** Learn reward function assuming it's linear combination of features:
```
R(s,a) = w^T φ(s,a)
```

**Process:**
1. Extract feature representations φ from demonstrations
2. Find weight vector w such that expert policy maximizes E[w^T φ]
3. Use linear program to find optimal w
4. If policy deviates, add more examples, iterate

**Connection to Maximum Margin Learning:** Similar to support vector methods.

## GAIL (Generative Adversarial Imitation Learning)

Modern approach using adversarial training:

**Core Idea:** Train policy to fool discriminator distinguishing expert vs learned trajectories.

**Setup:**
- **Generator:** Policy π_θ generating trajectories
- **Discriminator:** Classifies trajectories as expert or learned

**Objective:**
```
min_θ max_D E_expert[log D(τ)] + E_π_θ[log(1-D(τ))]
```

**Advantages:**
- No explicit reward function needed
- Learns directly from raw observations
- Stable training with adversarial framework
- Sample efficient compared to behavioral cloning

**Key Insight:** Discriminator implicitly learns reward function; policy learns to match expert behavior.

## Behavioral Cloning Baseline

**Simplest Approach:** [Supervised learning](/supervised-learning-eng.html) from demonstrations.
```
min_θ ||π_θ(a|s) - π_expert(a|s)||²
```

**Limitations:**
- Distribution shift: Only learns from expert's state distribution
- Compounding errors: Small mistakes accumulate
- Lack of exploration
- Poor generalization to new states

**When It Works:** Simple tasks with clear action patterns.

## Comparative Analysis

**IRL vs Behavioral Cloning:**
- IRL infers underlying reward structure
- BC directly mimics actions
- IRL more robust to distribution shift
- BC faster for simple tasks

**IRL vs Reinforcement Learning:**
- IRL learns from demonstrations
- RL learns from sparse/shaped reward signals
- IRL useful when reward design difficult
- RL effective with clear objectives

## Applications

**Autonomous Driving:**
- Learn from expert human drivers
- Infer safety considerations, comfort preferences
- Generalize to new roads/conditions

**Robotic Manipulation:**
- Learn from teleoperation demonstrations
- Understand task objectives beyond explicit rewards
- Transfer to new scenarios

**Preference Learning:**
- Infer user preferences from behavior
- Personalization without explicit instructions

**Safety-Critical Systems:**
- Learn human values/preferences
- Build autonomous systems respecting human concerns

## Practical Challenges

**Demonstration Quality:** Expert demonstrations must be good. Poor demos mislead learning.

**Feature Engineering:** Good feature representations essential for IRL success.

**Scalability:** Computing optimal policies for high-dimensional problems expensive.

**Ambiguity:** Many reward functions explain same behavior; different weights could be learned.

## Advanced Topics

**Bayesian IRL:** Posterior distribution over rewards. Captures ambiguity.

**Deep IRL:** [Neural networks](/neural-eng.html) learn reward functions directly from raw observations.

**Multi-Objective IRL:** Learn multiple competing objectives from demonstrations.

**Interactive IRL:** Agent queries human for clarification during learning.

## Deep Learning Integration

**DeepIRL:** Using [neural networks](/neural-eng.html) for reward function:
```
R_θ(s,a) = neural_net_θ(s,a)
```

Learns from high-dimensional observations directly. Enables end-to-end learning.

**Integrating with Policy Learning:**
- Simultaneously learn reward and policy
- Iterative refinement using demonstrations

## Ethical Considerations

**Values Learning:** IRL learns human values; must ensure alignment.

**Bias Amplification:** If demonstrations contain biases, learned rewards perpetuate them.

**Goal Specification:** What objective does system actually optimize?

**Transparency:** Inferred rewards should be interpretable.

## Connection to Imitation Learning

**Imitation Learning Spectrum:**
1. **Behavioral Cloning:** Direct action copying
2. **IRL → RL:** Infer reward, optimize policy
3. **GAIL:** Adversarial imitation without explicit reward
4. **Online Learning:** Query expert during learning

**Trade-offs:**
- Behavioral cloning: Fast, simple, poor generalization
- GAIL: Flexible, stable, more complex
- IRL: Interpretable rewards, computationally expensive

## Recent Advances

**Offline IRL:** Learning from fixed demonstration datasets without environment interaction.

**Preference Learning:** Learning from pairwise comparisons rather than full demonstrations.

**Meta-IRL:** Learning how to infer rewards across task distributions.

**Uncertainty-Aware IRL:** Quantifying ambiguity in inferred rewards.

## Conclusion

Inverse RL provides framework for learning reward functions from demonstrations, addressing the reward specification problem. Maximum [Entropy](/information-theory-eng.html) IRL provides principled approach; GAIL offers practical, efficient learning. These methods enable agents to learn from human expertise, crucial for safety-critical and preference-sensitive applications. As AI systems become more autonomous, learning human values through IRL becomes increasingly important.

