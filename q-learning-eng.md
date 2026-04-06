---
layout: default
title: Q-Learning & DQN
lang: en
---

# Q-Learning and Deep Q-Networks: Model-Free Value Learning

## Introduction

Q-Learning is a foundational model-free reinforcement learning algorithm learning optimal action values without requiring environment model. Deep Q-Networks extend this to high-dimensional state spaces through neural networks. Both remain highly influential in practical RL applications.

## Q-Values and Q-Tables

Q(s,a) represents expected cumulative discounted reward from taking action a in state s and following optimal policy afterward.

**Bellman Optimality for Q:**
```
Q*(s,a) = E[R_{t+1} + γ max_{a'} Q*(s',a') | S_t = s, A_t = a]
```

**Q-Table:** For discrete state/action spaces, store Q(s,a) in lookup table. Updates become simple table operations.

## Temporal Difference Learning

TD methods bootstrap—estimate future value using current value estimates:

```
NewEstimate = OldEstimate + α[Target - OldEstimate]
```

Where Target incorporates next state's current estimate.

**Advantages over Monte Carlo:**
- Learn before episode ends
- Lower variance than full trajectory returns
- Single step feedback

## Q-Learning Algorithm

**Off-Policy Update Rule:**
```
Q(S_t, A_t) ← Q(S_t, A_t) + α[R_{t+1} + γ max_{a'} Q(S_{t+1}, a') - Q(S_t, A_t)]
```

**Key Characteristics:**
- Off-policy: Can learn from any behavior policy
- Uses greedy policy (max Q) for bootstrapping
- Actual actions can follow different policy (exploration)
- Guaranteed convergence under mild conditions

**Why Off-Policy Matters:**
Separate behavior policy (exploration) from target policy (greedy). Enables learning from suboptimal trajectories, improves sample efficiency.

## Experience Replay

**Problem:** Sequential experiences are highly correlated, violating i.i.d. assumption of supervised learning.

**Solution:** Store transitions in replay buffer, sample minibatches for updates:

```
Store (s, a, r, s') in buffer
Sample random minibatch
Update Q using minibatch
```

**Benefits:**
- Breaks correlation between samples
- Reuses past experience multiple times
- Stabilizes learning
- Enables asynchronous training

## Deep Q-Networks (DQN, 2015)

Extends Q-Learning to continuous state spaces:

**Components:**
1. **Neural Network:** Maps states to Q(s,a) for all actions
2. **Experience Replay:** Random sampling from memory buffer
3. **Target Network:** Separate network for bootstrapping

**Target Network:**
Keep copy of network for computing targets:
```
Target = r + γ max_{a'} Q_target(s', a')
Loss = [Target - Q(s,a)]²
```

Update target network every N steps. Stabilizes learning by reducing divergent bootstrapping.

**Why Target Network:**
Prevents chasing moving target. Q-network and target often diverge, causing instability.

## Double DQN

**Overestimation Problem:** Max operation in Q-Learning leads to overestimated values.

**Solution:** Use current network for action selection, target network for evaluation:
```
a* = argmax_a Q(s',a) using current network
Target = r + γ Q_target(s',a*) using target network
```

Separates action selection from evaluation, reducing overestimation.

## Dueling DQN

**Architecture Innovation:** Separate value and advantage streams:

```
V(s): State value, independent of action
A(s,a): Advantage, how much better action is than average
Q(s,a) = V(s) + A(s,a) - mean(A)
```

**Benefits:**
- Shares features for state value across actions
- Learns action advantages independently
- Faster learning with improved stability
- Better learning signal for rarely-visited states

## Rainbow DQN

Combines multiple improvements:
1. Double DQN: Reduced overestimation
2. Dueling DQN: Value/advantage decomposition
3. Prioritized Replay: Sample important transitions more
4. Noisy Networks: Exploration through parameter noise
5. Distributional RL: Learn full Q-distribution
6. Multi-step TD: n-step bootstrapping

Significantly outperforms vanilla DQN.

## Exploration vs Exploitation

**ε-Greedy:** Take random action with probability ε, else greedy.
- Simple, effective for many problems
- ε-decay: Decrease ε over time for more exploitation

**Other Strategies:**
- Boltzmann Exploration: Softmax over Q-values
- Thompson Sampling: Uncertainty-based exploration
- Upper Confidence Bound (UCB): Optimism under uncertainty

## Practical Considerations

**Hyperparameters:**
- Learning rate α: Controls update step size
- Discount γ: Weights future vs immediate rewards
- Replay buffer size: Memory for past experiences
- Target network update frequency
- ε and ε-decay schedule

**Stability Issues:**
- Function approximation divergence
- High bias from overestimation
- Non-stationary targets
- Correlated samples

## Limitations

- Sample inefficiency despite replay buffer
- Limited to discrete action spaces (extensions exist)
- Discrete nature of max operation
- Struggles with very large action spaces

## Modern Extensions

- **Rainbow DQN:** Combines multiple improvements
- **Noisy Networks:** Parameterized exploration
- **Distributional RL:** Learn value distributions
- **Offline RL:** Learning from fixed datasets
- **Multi-step Learning:** n-step TD returns

## Conclusion

Q-Learning provides foundation for model-free RL. DQN successfully scaled Q-Learning to high-dimensional states through neural networks and stabilization techniques. Double DQN and Dueling DQN addressed specific issues. Rainbow integrated multiple improvements into unified architecture. These methods established deep RL and remain influential despite newer policy gradient methods.

