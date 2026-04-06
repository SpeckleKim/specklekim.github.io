---
layout: default
title: Model-Based RL
lang: en
---

# Model-Based Reinforcement Learning: Planning with Learned Models

## Introduction

Model-based RL learns environment models enabling agents to plan. Unlike model-free methods that learn directly from experience, model-based approaches use planning algorithms (search, dynamic programming) with learned models. This paradigm offers sample efficiency, interpretability, and transfer learning opportunities at the cost of model learning complexity.

## World Models

**Core Concept:** Learn a dynamics model—how environment evolves given current state and action.

**Representation:** Could be:
- **Latent Models:** Compress observations into learned latent space
- **Pixel Models:** Predict next frame directly
- **Stochastic Models:** Capture action/environment stochasticity
- **Deterministic Models:** Assume deterministic dynamics

## Model Learning

**Components:**

1. **State Encoder:** Compress observations to state representation
2. **Dynamics Predictor:** Model p(s_{t+1}|s_t, a_t)
3. **Reward Model:** Predict r(s_t, a_t)
4. **Terminal Model:** Predict episode termination

**Training Objective:**
```
L = ||s_{t+1} - f(s_t, a_t)||² + ||r - r_model(s_t, a_t)||²
```

**Challenges:**
- Compounding error: Model errors propagate over long horizons
- Off-distribution: Planning may select states rarely seen during training
- Stochasticity: Uncertainty over futures challenging to model

## Planning Methods

**Model Predictive Control (MPC):**
```
For each timestep:
  For each action trajectory:
    Simulate forward H steps using model
    Evaluate cumulative reward
  Execute best action trajectory
```

Myopic but computationally feasible for short horizons.

**Cross-Entropy Method:** Iteratively refine action distribution.

**Trajectory Optimization:** Gradient-based optimization of action sequences.

## Dreamer Architecture

Recent state-of-the-art model-based approach:

**Components:**
1. **World Model:** VAE-based latent dynamics with stochastic variables
2. **Actor:** Policy network maximizing imagined returns
3. **Critic:** Value network in imagination space
4. **Imagination Rollouts:** Plan trajectories in learned latent space

**Key Insight:** Train policy within imagination (latent space), bypass pixel-level model errors.

**Training Loop:**
```
1. Collect real world experience
2. Train world model on experience
3. Imagine trajectories using world model
4. Train actor/critic on imagined trajectories
5. Execute policy in real environment
6. Repeat
```

**Advantages:**
- Imagination loop decouples policy learning from model errors
- Sample efficient: One real trajectory generates many imagined ones
- Scalable: Latent space reduces dimensionality
- Stable: Purely learned objectives without explicit MPC

## MuZero Architecture

AlphaZero-style planning without explicit environment model:

**Key Innovation:** Learn value-equivalent model—model rewards and values, not states.

**Components:**
1. **Representation:** Encode observation into latent state
2. **Prediction:** Predict value, reward, policy from latent
3. **Dynamics:** Transition in latent space without reconstructing states

**Advantage:** Model only what's relevant for planning (value, rewards), ignoring irrelevant details.

**Applications:**
- Atari games: State-of-the-art performance
- Board games: Superhuman performance on Go, Chess
- General domains: Effective across diverse environments

## Imagination-Based Training

**Core Loop:**
1. **Real Experience:** Collect trajectories in actual environment
2. **Model Training:** Update dynamics, reward, value models
3. **Imagination:** Plan trajectories using learned models
4. **Policy Learning:** Train policy on imagined trajectories

**Benefits:**
- Massive sample efficiency: Few real transitions, many imagined
- Stability: Value targets from imagination more stable than TD
- Transfer: Learned models enable transfer between tasks

## Uncertainty Quantification

Handling model uncertainty improves robustness:

**Ensemble Methods:**
- Train multiple models
- Use disagreement as uncertainty estimate
- Encourage exploration toward uncertain regions

**Bayesian Approaches:**
- Probabilistic models with uncertainty
- Posterior distributions over weights
- Thompson sampling from uncertainty

## Model Learning Challenges

**Compounding Error:**
```
Error at step t+1 depends on error at step t
Long-horizon error grows exponentially
```

Mitigation: Short imagination horizons, frequent model retraining.

**Off-Distribution Shift:**
```
Planning may select states not in training data
Model accuracy degrades off-training distribution
```

Mitigation: Ensemble uncertainty, constrained optimization.

**High-Dimensional Observations:**
```
Pixel-level modeling difficult
Latent models required for scalability
```

## Model-Based + Model-Free Hybrid

Combine strengths of both paradigms:

**MBPO (Model-Based Policy Optimization):**
- Train dynamics model from real experience
- Use model to generate imaginary transitions
- Train policy with model-free algorithm (SAC) on mixed real/imaginary data
- Keeps model-free stability, gains sample efficiency from model

**Benefits:**
- Better sample efficiency than pure model-free
- More stable than pure model-based
- Practical trade-off between paradigms

## Advantages of Model-Based RL

**Sample Efficiency:** Learn from fewer real interactions.

**Planning Capability:** Deliberate about futures beyond immediate action.

**Transfer Learning:** Models learned on one task transfer to related tasks.

**Interpretability:** Predictions provide insights into learned dynamics.

**Off-Policy Learning:** Plan with historical data without policy constraint.

## Limitations

**Model Complexity:** Learning accurate models challenging, especially in high-dimensional spaces.

**Computational Cost:** Planning often expensive compared to model-free inference.

**Imagination Divergence:** Model errors accumulate during planning.

**Limited Exploration:** Pure planning may miss important exploration regions.

## Modern Approaches

**Diffusion-Based Models:** Using diffusion for sequence modeling in RL.

**Successor Models:** Learning abstract successor representations.

**Latent World Models:** Learning in compressed observation spaces.

**Hybrid Approaches:** Seamlessly combining model-based and model-free.

## Practical Considerations

**When to Use Model-Based:**
- Sample efficiency critical (robotics, real-world tasks)
- Environment dynamics learnable (physics simulations)
- Need for planning and interpretability
- Transfer to related tasks important

**When to Use Model-Free:**
- Abundant data available
- Environment highly stochastic/complex
- Rapid training needed (large-scale domains)
- State representation easily learned from data

## Conclusion

Model-based RL enables planning with learned models, offering sample efficiency and interpretability advantages. Dreamer and MuZero represent modern approaches that effectively balance model learning with policy optimization. Hybrid methods combining model-based and model-free approaches are increasingly attractive. As sample efficiency and real-world applicability gain importance, model-based RL becomes increasingly relevant for practical applications.

