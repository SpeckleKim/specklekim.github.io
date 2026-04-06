---
layout: default
title: Policy Gradient Methods
lang: en
---

# Policy Gradient Methods: Directly Learning Policies

## Introduction

Policy gradient methods directly optimize policies by computing gradients of expected return. Unlike value-based methods that learn Q-values, policy gradients learn parameterized policies. This approach offers advantages in continuous control, on-policy learning, and theoretical properties.

## Policy Gradient Theorem

**Core Result:** Gradient of expected return with respect to policy parameters:

```
∇_θ J(θ) = E[∇_θ log π_θ(a|s) Q^π(s,a)]
```

This elegant result shows how to compute policy gradients using log-probabilities and Q-values. Enables efficient policy optimization without model knowledge.

**Intuition:** Increase probability of high-value actions, decrease low-value ones.

## REINFORCE

Simplest policy gradient algorithm:

**Algorithm:**
```
For each episode:
  Sample trajectory τ = (s_0,a_0,r_0,...,s_T,a_T)
  Compute return G_t = Σ_{k≥t} γ^{k-t} r_k
  Update: θ ← θ + α ∇_θ log π_θ(a_t|s_t) G_t
```

**Characteristics:**
- On-policy: Must sample from current policy
- Monte Carlo returns: Full trajectory needed
- High variance: Returns highly variable
- Guaranteed convergence to local optimum

**Advantages:**
- Simple implementation
- Works with continuous and discrete actions
- Theoretically sound

**Disadvantages:**
- High variance requires many samples
- Slow convergence
- Can diverge without careful learning rates

## Baseline Functions

**Problem:** Using raw returns G_t as critic has high variance.

**Solution:** Subtract baseline b(s) from returns:

```
∇_θ J(θ) = E[∇_θ log π_θ(a|s) (Q^π(s,a) - b(s))]
```

Baseline doesn't affect expected gradient but reduces variance. Optimal baseline is value function V^π(s).

**Benefits:**
- Dramatically reduces variance
- Maintains unbiased gradient
- Same convergence guarantees
- Simple implementation

## Proximal Policy Optimization (PPO)

Modern state-of-the-art policy gradient method:

**Key Innovation:** Clipped objective prevents oversized policy changes:

```
L(θ) = E[min(r_t(θ)Â_t, clip(r_t(θ),1-ε,1+ε)Â_t)]
```

Where r_t(θ) = π_θ(a|s) / π_old(a|s) is probability ratio, Â_t is advantage.

**Why Clipping:**
Without clipping, gradient encourages large probability ratio jumps. Clipping bounds ratio to [1-ε, 1+ε], constraining policy change per update.

**Practical Benefits:**
- Stable, reliable learning
- Works across diverse domains
- Robust to hyperparameters
- Sample efficient compared to REINFORCE

**PPO Pseudocode:**
```
For each epoch:
  Collect trajectories using π_old
  Compute advantages and returns
  For each minibatch:
    Optimize clipped objective for K epochs
```

## Trust Region Policy Optimization (TRPO)

Principled approach to constrain policy changes:

**Objective:** Maximize expected return while constraining KL divergence:

```
maximize E[r_t(θ)Â_t]
subject to E[KL(π_old || π_θ)] ≤ δ
```

**Natural Gradient:** Incorporates Fisher information matrix. Updates invariant to policy parameterization.

**Connection to PPO:** PPO approximates TRPO with simpler clipped objective rather than explicit KL constraint.

## Advantage Estimation (GAE)

Generalized Advantage Estimation balances bias-variance trade-off:

```
Â_t = Σ_{l=0}^∞ (γλ)^l δ_t^{V}
```

Where δ_t^{V} = r_t + γV(s_{t+1}) - V(s_t) is TD residual.

**λ Parameter:**
- λ = 0: Use bootstrapped value (low variance, high bias)
- λ = 1: Use full trajectory (high variance, low bias)
- λ ∈ (0,1): Weighted combination

GAE enables flexible bias-variance control, improving practical performance.

## Actor-Only Policy Gradient

Without baseline (value function):

```
∇_θ J(θ) = E[∇_θ log π_θ(a|s) r_t]
```

- Very high variance
- Slow convergence
- Generally avoided in practice

## Continuous vs Discrete Actions

**Discrete Actions:**
- Policy outputs probability distribution over actions
- Natural for categorical distributions

**Continuous Actions:**
- Policy outputs mean and variance of Gaussian: π_θ(a|s) = N(μ_θ(s), σ_θ(s))
- Enables smooth control

## Practical Considerations

**Hyperparameter Sensitivity:**
- Learning rate crucial; affects convergence speed and stability
- Number of policy update steps
- Advantage estimation lambda (GAE)
- Clipping epsilon (PPO)

**Implementation Details:**
- Normalize advantages: Improves stability and generalization
- Entropy bonus: Encourages exploration, prevents premature convergence
- Value function loss: Minimize (V_θ(s) - target)²

## Limitations

- On-policy learning: Must interact with environment constantly
- Correlated trajectory samples: Samples within episode are dependent
- Exploration challenges: Might get stuck in local optima
- Sample inefficiency: Typically lower sample efficiency than value-based methods

## Modern Extensions

- **Distributed PPO:** Training across multiple workers
- **Asynchronous Methods:** A3C using parallel actors
- **Multi-task Learning:** Learning policies for multiple tasks
- **Curriculum Learning:** Progressive task difficulty
- **Meta-learning:** Learning to adapt policies quickly

## Conclusion

Policy gradient methods directly optimize policies using gradient ascent. REINFORCE pioneered the approach; baselines reduced variance. PPO introduced practical, stable training through policy clipping. TRPO provided theoretical foundations. These methods become standard for continuous control and remain competitive despite newer approaches. Their simplicity, flexibility, and theoretical grounding explain their continued prominence in RL.

