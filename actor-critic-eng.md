---
layout: default
title: Actor-Critic Methods
lang: en
---

# Actor-Critic Methods: Combining Value and Policy Learning

## Introduction

Actor-critic methods combine policy-based and value-based approaches. The actor (policy) determines actions; the critic (value function) evaluates them. This combination balances variance reduction with policy optimization, enabling stable and efficient learning across diverse domains.

## Actor-Critic Framework

**Two Components:**

1. **Actor:** Policy π_θ(a|s) that selects actions
2. **Critic:** Value function V_ϕ(s) or Q_ϕ(s,a) that estimates value

**Advantage:** Critic provides baseline for variance reduction while actor optimizes policy using critic's estimates.

**Information Flow:**
```
State → Actor → Action → Environment
            ↓
        Critic evaluates action
            ↓
        Provides TD error as signal
```

## Policy Update

Actor uses critic's value estimates:

```
∇_θ J(θ) = E[∇_θ log π_θ(a|s) Q_ϕ(s,a)]
```

Rather than waiting for full trajectory (REINFORCE), use critic's Q-value estimate. Reduces variance, enables lower-variance updates.

## Value Function Update

Critic learns from temporal differences:

```
δ_t = r_t + γV_ϕ(s_{t+1}) - V_ϕ(s_t)
ϕ ← ϕ - β∇_ϕ [V_ϕ(s_t) - (r_t + γV_ϕ(s_{t+1}))]²
```

TD error δ_t serves dual purpose: critic update and advantage estimate for actor.

## Advantage Estimation

Using TD error as advantage:

```
Â_t = δ_t = r_t + γV_ϕ(s_{t+1}) - V_ϕ(s_t)
```

Actor update becomes:

```
θ ← θ + α ∇_θ log π_θ(a_t|s_t) δ_t
```

Low variance compared to REINFORCE yet efficient.

## A2C (Advantage Actor-Critic)

Synchronous version using batched updates:

**Algorithm:**
```
For each batch:
  Collect transitions using current policy
  Compute value targets: V_target = r + γV_ϕ(s')
  Compute advantages: Â = V_target - V_ϕ(s)

  Actor loss: -log π_θ(a|s) × Â
  Critic loss: (V_ϕ(s) - V_target)²

  Update both networks
```

**Characteristics:**
- Synchronous: Parallel environments, synchronized updates
- Efficient: Leverages parallel data collection
- Stable: Advantage clipping and value loss stabilize training
- Practical: Good sample efficiency and wall-clock time

## A3C (Asynchronous Advantage Actor-Critic)

Asynchronous parallel version:

**Key Innovation:** Multiple parallel workers collecting experience asynchronously.

**Architecture:**
- Global policy and value networks
- Worker threads with local networks
- Asynchronous gradient accumulation to global networks

**Advantages:**
- Natural parallelization: Multiple workers explore independently
- On-policy: Each worker acts with most recent policy
- Reduced memory: No experience replay needed
- Exploration: Diverse worker experiences improve coverage

**Disadvantages:**
- Complex implementation
- Asynchrony can hurt convergence
- Off-policy corrections difficult

## SAC (Soft Actor-Critic)

Entropy-regularized approach for robustness:

**Key Insight:** Maximize expected return AND entropy to encourage exploration:

```
J(π) = E[r(s,a) + αH(π(·|s))]
```

Where H is policy entropy, α is temperature controlling exploration intensity.

**Mechanism:**
1. Soft value function incorporates entropy: V_soft = Q - α log π
2. Policy trained to maximize soft Q-value
3. Q-networks updated toward soft Bellman target

**Benefits:**
- More stable learning through entropy regularization
- Better exploration-exploitation balance
- Robust to hyperparameters
- Effective for continuous control

## TD3 (Twin Delayed Deep Deterministic Policy Gradient)

Addressing overestimation in deterministic policy gradient:

**Three Key Improvements:**

1. **Twin Q-networks:** Use two Q-networks, take minimum for target
   ```
   Target = r + γ min(Q_ϕ1(s',π_θ(s')), Q_ϕ2(s',π_θ(s')))
   ```

2. **Delayed Policy Updates:** Update policy less frequently than value networks
   ```
   Update Q every step; update π and target networks every d steps
   ```

3. **Target Policy Smoothing:** Add noise to target policy actions
   ```
   Target action = π_θ(s') + ε, where ε ~ clip(N(0,σ), -c, c)
   ```

**Results:** Significantly reduced overestimation, more stable learning.

## Off-Policy Actor-Critic

While A2C is on-policy, many actor-critic methods use off-policy:

**DDPG (Deep Deterministic Policy Gradient):**
- Deterministic policy for continuous actions
- Experience replay buffer
- Target networks for stability
- Can learn from suboptimal experience

**Off-Policy Advantages:**
- Better sample efficiency
- Can learn from previously collected data
- Supports more efficient exploration

## Stability Considerations

**Critic Convergence:** Critic must converge faster than actor for reliable gradient estimates.

**Learning Rate Ratio:** Typically α_critic > α_actor to ensure critic keeps up.

**Variance Sources:**
1. Function approximation error in critic
2. Bias from bootstrapping
3. Non-stationary targets (moving critic)

**Mitigation:**
- Target networks: Separate networks for computing targets
- Advantage clipping: Bound advantage estimates
- Value normalization: Normalize returns before Q-learning

## Practical Considerations

**Hyperparameter Tuning:**
- Actor and critic learning rates
- Discount factor γ
- Advantage clipping range
- Network architectures and sizes

**Network Initialization:**
- Proper scaling of initial weights
- Bias initialization affects learning
- Output ranges matter (especially for deterministic policies)

**Data Collection:**
- Batch size effects convergence and stability
- Parallel workers improve sample efficiency
- Exploration noise crucial for continuous action spaces

## Comparison to Other Methods

**vs Value-Based (DQN):**
- Better for continuous actions
- More stable training
- Better exploration

**vs Pure Policy Gradient:**
- Reduced variance from value baseline
- Faster convergence
- More data efficient

**vs Model-Based:**
- Model-free: No environment model needed
- Simpler implementation
- Often competitive with model-based in practice

## Modern Applications

- **Robotics:** Continuous control, real robot learning
- **Game Playing:** Continuous action spaces (StarCraft with macro-actions)
- **Autonomous Systems:** Real-time decision making
- **Resource Allocation:** Economic dispatch, task scheduling

## Conclusion

Actor-critic methods unify policy optimization with value-based learning. A2C and A3C demonstrated practical effectiveness; SAC introduced entropy regularization; TD3 addressed overestimation. These methods dominate modern deep RL, particularly for continuous control. Their flexibility, sample efficiency, and theoretical grounding make them go-to approaches for many applications.

