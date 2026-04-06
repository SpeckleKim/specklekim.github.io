---
layout: default
title: Multi-Agent RL
lang: en
---

# Multi-Agent Reinforcement Learning: Beyond Single Agents

## Introduction

[Multi-Agent](/multi-agent-eng.html) [Reinforcement Learning](/rl-eng.html) (MARL) extends single-agent RL to environments with multiple interacting agents. Agents can be cooperative (shared objectives), competitive (opposing goals), or mixed. MARL poses challenges: non-stationary environments, coordination difficulties, scalability issues. Applications include robot swarms, traffic management, and game playing.

## Fundamental Differences

**Single-Agent RL:** Environment's response depends only on single agent's actions.

**Multi-Agent RL:** Environment's response depends on all agents' actions; each agent observes incomplete state information.

**Key Challenges:**
- Environment becomes non-stationary (other agents' policies change)
- Credit assignment: Who caused success/failure with multiple agents?
- Scalability: Exponential state/action space growth
- Convergence: No guarantees when multiple agents adapt simultaneously

## Game-Theoretic Foundations

**Normal Form Games:**
- Simultaneous action selection
- Payoff matrices determine outcomes
- Nash Equilibrium: No agent benefits from unilateral deviation

**Extensive Form Games:**
- Sequential action selection
- Game trees represent decision structures
- Subgame perfect equilibrium more appropriate

**Zero-Sum vs General-Sum:**
- Zero-sum: One agent's gain is another's loss (poker, chess)
- General-sum: Possible mutual benefit/loss (traffic, cooperation)

## Nash Equilibrium

Fundamental game-theoretic solution concept:

```
σ* is Nash Equilibrium if:
E[u_i(σ*_i, σ*_{-i})] ≥ E[u_i(σ'_i, σ*_{-i})] for all σ'_i
```

Each agent's strategy is best response to others' strategies.

**Multi-Agent Learning Goal:** Find Nash Equilibrium policy profile.

**Challenges:**
- Multiple equilibria may exist
- Not all equilibria are desirable
- Convergence to equilibrium uncertain

## Cooperative vs Competitive

**Fully Cooperative:**
- Shared reward: All agents maximize same objective
- Communication essential
- Centralized training beneficial

**Fully Competitive:**
- Zero-sum games
- Opposing objectives
- Self-play training effective

**Mixed Settings:**
- Team competition (multiple teams)
- Mixed-motive games (some shared, some opposing)
- Complex strategic reasoning required

## Centralized Training, Decentralized Execution (CTDE)

**Key Framework:** Train with joint state/action information; execute with local observations.

**Training Phase:**
- Central critic observes all agents, all actions
- Training uses full information

**Execution Phase:**
- Each agent acts independently
- Uses only local observations
- Learned policies must work without communication

**Benefits:**
- Training stability from global information
- Execution scalability without central coordinator
- Addresses credit assignment through global critic

**Canonical Approach: MADDPG**
- [Multi-agent](/multi-agent-eng.html) extension of DDPG
- Centralized critics during training
- Decentralized actors for execution

## Communication and Coordination

**Implicit Communication:** Learned through emergent behaviors.

**Explicit Communication:** Agents send messages; learn communication protocol.

**Coordination Mechanisms:**
- Handshake protocols
- Shared goals/targets
- Reward shaping toward coordination

**CommNet, QMIX:** Architectures enabling inter-agent communication.

## MAPPO (Multi-Agent PPO)

Extension of PPO to [multi-agent](/multi-agent-eng.html) settings:

**Key Components:**
1. **Shared Policy Network:** All agents share policy parameters
2. **Individual Value Networks:** Each agent has separate value function
3. **Centralized Value Function:** Central critic for stability (optional)
4. **Clipped Surrogate:** PPO clipping applied to [multi-agent](/multi-agent-eng.html) case

**Training:**
```
For each episode:
  All agents collect transitions
  Compute advantages using local value functions
  Update shared policy and individual values
  Optionally update centralized critic
```

**Characteristics:**
- Simple, practical algorithm
- Effective in many cooperative domains
- Scalable to many agents
- Maintains [policy gradient](/policy-gradient-eng.html) benefits

## Learning in Non-Stationary Environments

**Core Problem:** As other agents' policies change, environment appears non-stationary.

**Solutions:**

**Policy Lag:** Keep opponent models fixed temporarily.
- Train against fixed policies
- Periodically update opponent models
- Reduces non-stationarity effects

**Opponent Modeling:** Explicitly track other agents' behaviors.
- Learn models of opponent policies
- Adapt own policy accordingly
- Computationally expensive

**Self-Play:** Train against copies of own policy.
- Effective for zero-sum games
- Natural curriculum: Increasingly strong opponents
- Enables superhuman performance (AlphaGo, AlphaStar)

## Self-Play Training

**Process:**
```
For each iteration:
  Current policy v_t plays against previous versions
  Train new policy v_{t+1} to beat v_t
  Accumulate experience from all generations
  v_{t+1} becomes new best, possibly rollback if weaker
```

**Advantages:**
- Natural curriculum: Opponents grow stronger with learner
- Exploration: Previous policies provide exploration diversity
- Complexity management: Progressive difficulty increase

**Success Stories:**
- AlphaGo defeating world champion Go players
- AlphaStar reaching superhuman StarCraft performance
- Emergent behaviors in [multi-agent](/multi-agent-eng.html) environments

## Emergent Behaviors

**Remarkable Phenomenon:** Complex, coordinated behaviors emerge from simple reward functions without explicit programming.

**Examples:**
- Predator-prey dynamics without preprogrammed behaviors
- Formation control without explicit coordination rules
- Language-like communication in [multi-agent](/multi-agent-eng.html) systems
- Task specialization in teams

**Factors Enabling Emergence:**
- Sufficient agent population
- Persistent training
- Appropriate reward structure
- Environmental complexity

## Scalability Challenges

**Exponential Growth:**
- State space: O(|S|^n) for n agents
- Action space: O(|A|^n) for n agents
- Complexity grows dramatically with agent count

**Solutions:**

**Mean-Field Approximation:** Assume single agent interacts with average of other agents.
```
Value ≈ f(own_state, mean_field_of_others)
```
Reduces dimensionality, enables many-agent learning.

**Graph Neural Networks:** Use agent interactions as graph structure.
- Share representations across agents
- Leverage permutation invariance
- Scale to large agent populations

**Hierarchical Learning:** Different scales of coordination.
- Low level: Individual agent control
- High level: Team/swarm coordination

## Applications

**Robot Swarms:** Coordinated motion, collective decision-making.

**Traffic Control:** Autonomous vehicles coordinating on roads; reducing congestion.

**Game Playing:** Dota 2 (OpenAI Five), StarCraft (AlphaStar), Chess variants.

**Supply Chain:** Multiple warehouses, distributors coordinating.

**Spectrum Sharing:** Wireless devices learning to coexist.

## Benchmarks and Environments

**SMAC (StarCraft Multi-Agent Challenge):** Cooperative combat scenarios in StarCraft.

**OpenAI Gym Environments:** [Multi-agent](/multi-agent-eng.html) variants of classic environments.

**OpenSpiel:** Board games, game-theoretic environments.

**Custom Simulators:** Tailored for specific domains (robotics, traffic, etc.).

## Theoretical Considerations

**Convergence Guarantees:** Limited in [multi-agent](/multi-agent-eng.html) settings.
- Single-agent convergence proofs don't apply
- Cyclic behaviors possible
- Non-convergent dynamics common

**Communication Complexity:** How many messages needed?
- Information-theoretic limits
- Trade-off between efficiency and coordination

**Hardness Results:** Some [multi-agent](/multi-agent-eng.html) problems NP-hard.

## Modern Trends

**Attention Mechanisms:** Focus on relevant agents for communication/coordination.

**Transformer Architectures:** Handle variable agent populations.

**Learning Communication Protocols:** Agents discover optimal messaging strategies.

**Value Decomposition:** Break joint value into individual contributions (QMIX, QTRAN).

## Conclusion

[Multi-agent](/multi-agent-eng.html) RL extends single-agent learning to complex, interactive environments. Cooperative and competitive settings require different approaches. CTDE framework enables scalable training and execution. Self-play drives superhuman performance in zero-sum games. Emergent behaviors demonstrate complex coordination from simple objectives. As multi-agent systems proliferate (autonomous vehicles, drone swarms, game AI), MARL becomes increasingly crucial for practical applications.

