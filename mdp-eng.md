---
layout: default
title: Markov Decision Process
lang: en
---

# Markov Decision Process: The Foundation of Reinforcement Learning

## Introduction

The Markov Decision Process (MDP) provides the mathematical framework underlying reinforcement learning. It formalizes the sequential decision-making problem where an agent interacts with an environment, receiving observations and rewards. Understanding MDPs is essential for grasping RL algorithms.

## Key Components

**States (S):** Complete descriptions of environment configurations. Agents observe current state and must make decisions. State space can be discrete or continuous, finite or infinite.

**Actions (A):** Possible decisions available to the agent. From each state, specific actions may be legal. Action space defines the agent's control capabilities.

**Transitions:** Deterministic or stochastic dynamics. Transition function P(s'|s,a) gives probability of next state s' given current state s and action a. Environment dynamics often unknown; agent learns from experience.

**Rewards (R):** Scalar feedback signals. Reward function r(s,a) specifies immediate payoff for action a in state s. Agent objective: maximize cumulative reward over time.

**Discount Factor (γ):** Weighting future rewards. Parameter 0 ≤ γ ≤ 1 balances immediate vs long-term rewards. γ near 1 prioritizes future; γ near 0 prioritizes immediate rewards.

## The Markov Property

"Future is independent of past given present":

```
P(S_{t+1}|S_t, A_t, S_{t-1}, ...) = P(S_{t+1}|S_t, A_t)
```

This property enables tractable computation. State must contain all relevant historical information; if not, the problem isn't truly Markovian.

## Value Functions

**State Value Function V(s):** Expected cumulative discounted reward starting from state s:

```
V(s) = E[G_t | S_t = s] where G_t = R_{t+1} + γR_{t+2} + γ²R_{t+3} + ...
```

**Action Value Function Q(s,a):** Expected cumulative reward from taking action a in state s:

```
Q(s,a) = E[G_t | S_t = s, A_t = a]
```

Both functions satisfy recursive relationships enabling efficient computation.

## Bellman Equations

Fundamental recursive equations relating value functions:

**Bellman Expectation Equation:**
```
V(s) = E[R_{t+1} + γV(S_{t+1}) | S_t = s]
V(s) = Σ_a π(a|s) Σ_{s',r} P(s',r|s,a)[r + γV(s')]
```

Where π is the policy (action selection strategy).

**Bellman Optimality Equation:**
```
V*(s) = max_a E[R_{t+1} + γV*(S_{t+1}) | S_t = s, A_t = a]
V*(s) = max_a Σ_{s',r} P(s',r|s,a)[r + γV*(s')]
```

These equations decompose long-term rewards into immediate reward plus discounted future reward.

## Policies

Policy π specifies action selection: π(a|s) = probability of action a in state s.

**Deterministic Policy:** π(a|s) ∈ {0,1}—always selects specific action

**Stochastic Policy:** π(a|s) ∈ [0,1]—probabilistic action selection

**Policy Evaluation:** Computing V^π for given policy π

**Policy Improvement:** Finding better policy using value function estimates

## Value Iteration

Computing optimal value function:

**Algorithm:**
```
Initialize V(s) = 0 for all s
Repeat until convergence:
  For each state s:
    V(s) = max_a Σ_{s',r} P(s',r|s,a)[r + γV(s')]
```

Directly solves Bellman optimality equation. Guaranteed convergence to V*. Computational cost: O(|S|²|A|) per iteration.

## Policy Iteration

Computing optimal policy:

**Algorithm:**
```
Initialize policy π randomly
Repeat until convergence:
  Policy Evaluation: Compute V^π
  Policy Improvement: π' = greedy(V^π)
  If π' = π: done
  Else: π = π'
```

Alternates between evaluating current policy and improving it. Typically faster convergence than value iteration.

## Generalized Policy Iteration

Framework unifying value and policy iteration:

1. **Evaluation:** Make V consistent with current policy
2. **Improvement:** Make policy greedy w.r.t. V

Both approaches drive toward optimal policy and value function.

## Practical Considerations

**Finite vs Infinite Horizon:**
- Finite: Episodes terminate after fixed steps
- Infinite: Discount factor ensures finite sums

**Continuing vs Episodic:**
- Episodic: Natural episode boundaries (game ends, task completes)
- Continuing: No natural termination

**Model-Based vs Model-Free:**
- Model-Based: Knows/learns P(s'|s,a) and R(s,a)
- Model-Free: Learns directly from experience

## Extensions

**Partially Observable MDPs (POMDPs):** Agent observes noisy signals instead of true state

**Continuous State/Action:** Infinite state/action spaces require function approximation

**Multi-Agent:** Multiple agents with competing or aligned objectives

## Conclusion

MDPs formalize sequential decision-making through states, actions, transitions, and rewards. Bellman equations reveal value functions' recursive structure. Value and policy iteration provide algorithms for optimal control. These foundations enable understanding modern RL algorithms that build on or extend these core concepts.

