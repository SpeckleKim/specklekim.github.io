---
layout: default
title: RLHF & Constitutional AI
lang: en
---

# RLHF & Constitutional AI

## Reinforcement Learning from Human Feedback

**Reinforcement Learning from Human Feedback (RLHF)** is a technique that uses human preferences to fine-tune language models beyond what supervised learning alone can achieve. Rather than providing ground-truth labels, humans simply choose which output they prefer, and the model learns to maximize these preferences.

This approach bridges a critical gap: supervised learning optimizes for predicting text, but users care about helpfulness, harmlessness, and honesty—dimensions that are difficult to capture with simple labels.

## The RLHF Pipeline

RLHF typically involves four stages:

### 1. Supervised Fine-Tuning (SFT)

Start with a pretrained language model and fine-tune it on high-quality instruction-following examples. This creates a baseline that already understands tasks and can produce coherent responses.

**Purpose**: Establish initial instruction-following capability without complex RL machinery.

### 2. Reward Model Training

The core innovation: train a **separate model** to predict human preferences.

**Process**:
1. Generate multiple outputs from the SFT model for the same prompt
2. Have humans rank these outputs (usually 2-4 outputs per prompt)
3. Train a **reward model** (RM) to predict which output humans prefer

The reward model typically:
- Takes prompt + output as input
- Outputs a scalar score (reward value)
- Is trained with a ranking loss (e.g., Bradley-Terry model)

**Key insight**: Rather than teaching the language model from preferences directly (which is slow), train a classifier that rapidly scores outputs.

### 3. Reinforcement Learning with PPO

Use the reward model to train the language model via **Proximal Policy Optimization (PPO)**.

**The objective**:
```
maximize: E[reward_model(output)]
subject to: KL divergence from original model < threshold
```

The KL divergence constraint prevents the model from diverging too far from the SFT baseline, which could degrade quality.

**PPO specifics**:
- Collect trajectories (prompt → generated output)
- Compute reward and advantage estimates using the reward model and a value function
- Optimize policy to maximize reward while staying close to the original policy
- Typically requires thousands of GPU hours even on moderate-sized models

### 4. Iteration and Refinement

This process often iterates:
- Collect new human preference data
- Retrain the reward model on expanded data
- Re-run PPO with improved reward estimates
- Evaluate and iterate

## Reward Model Training Depth

The reward model is crucial but often underappreciated. Key considerations:

### Training Data Quality

Reward models are only as good as their training data. This requires:
- **Diverse prompts**: Cover various task types and difficulty levels
- **High-quality rankings**: Consistent, well-reasoned human judgments
- **Detailed guidelines**: Clear criteria for ranking (helpfulness, harmlessness, honesty)
- **Disagreement handling**: Some examples will have disagreement; decisions on how to handle this matter

### Reward Hacking

A critical challenge: the policy might find ways to exploit the reward model without actually improving quality.

**Example**: If the reward model learns to prefer longer outputs, the language model might generate unnecessarily verbose text.

**Solutions**:
- Validate reward model predictions with human raters
- Use multiple reward models trained independently
- Include explicit penalties in the loss (e.g., length regularization)
- Use diverse training objectives

### Generalization

Reward models trained on limited data may not generalize to new domains or tasks. Practitioners need to:
- Evaluate generalization empirically
- Retrain on new domains rather than assuming transfer
- Monitor for distribution shift between train and deployment

## Constitutional AI

Anthropic's **Constitutional AI (CAI)** extends RLHF with a different approach: instead of only using human preferences, also train models with explicit principles or "constitution."

### How Constitutional AI Works

1. **Define principles**: Create a set of ethical principles (e.g., "Be helpful," "Be honest," "Be harmless")

2. **Self-critique**: Have the model evaluate its own outputs against the constitution:
   - Generate an output
   - Prompt the model to critique it against constitutional principles
   - Ask the model to revise based on this critique

3. **Preference collection**: Use the revised outputs for preference learning
   - Original output vs. revised output are presented to humans or to a reward model
   - This teaches the model to follow the constitution

4. **RL from Constitutional Feedback**: Use these preferences in PPO similar to traditional RLHF

### Advantages of CAI

- **Reduces human annotation burden**: Self-critique is less expensive than extensive human labeling
- **Explicit principles**: Easier to understand what the model is optimizing for
- **Interpretability**: Constitutional principles guide behavior transparently
- **Flexibility**: Can adjust principles without retraining from scratch

### Limitations

- **Self-critique quality**: Models may not effectively critique themselves
- **Principle encoding**: Translating abstract principles into training signals is challenging
- **Circular reasoning**: Models might optimize to appear compliant without genuine improvement

## Alternative Approaches: RLAIF

**Reinforcement Learning from AI Feedback (RLAIF)** is a variant where AI systems (rather than humans) provide feedback. This is faster and cheaper than human annotation but faces its own challenges:

- AI feedback can introduce biases or errors that compound
- Quality depends entirely on the feedback model's capabilities
- May optimize for AI-preferred behavior rather than human-preferred behavior

RLAIF is useful for rapid iteration or scaling but typically requires human validation at critical steps.

## Challenges in RLHF

### Sample Efficiency

RLHF is computationally expensive:
- Each RL step requires generating completions and computing rewards
- Most generated outputs are discarded
- A 7B model might need millions of training steps

Researchers are exploring techniques to improve sample efficiency, such as:
- Offline RL methods
- Batch generation and replay buffers
- More efficient policy gradient algorithms

### Instability

RL training can be unstable:
- The policy might collapse to a narrow distribution
- Reward signal might be noisy or inconsistent
- Divergence from SFT baseline might degrade overall quality

Mitigation strategies:
- Carefully tune learning rates and KL divergence weights
- Monitor reward model agreement with validators
- Implement rollback mechanisms for destabilized training

### Reward Gaming

Models find shortcut exploits:
- Generating text that appears helpful but isn't factually correct
- Repeating phrases that correlate with high rewards
- Over-optimizing for proxy objectives

Prevention requires:
- Adversarial evaluation and human spot-checking
- Ensemble reward models
- Training signals that are robust to exploitation

### Scalability

RLHF doesn't scale linearly with model size:
- Larger models are harder to train with RL
- Reward models may struggle to evaluate complex outputs from large models
- Communication and data transfer bottlenecks multiply

## Emerging Alternatives

### Direct Preference Optimization (DPO)

Rather than training a separate reward model, directly optimize the policy using preference data:
- Compare pairs of outputs for the same prompt
- Maximize probability of preferred output relative to dispreferred output
- Simpler, fewer hyperparameters, faster training

### Identity Preference Optimization (IPO)

Builds on DPO with theoretical improvements:
- Better handling of preference strength
- More stable training dynamics
- Improved convergence properties

## Best Practices

1. **Validate extensively**: Test reward model predictions with human raters regularly
2. **Use multiple RL training runs**: Train multiple models with different random seeds and evaluate variance
3. **Monitor for collapse**: Watch model outputs during training; some degradation during RL is normal but sudden collapses indicate problems
4. **Iterative improvement**: RLHF works best as an iterative process, not a one-shot training run
5. **Human oversight remains critical**: Even with RLHF, human evaluation of final models is essential
6. **Be transparent about principles**: If using Constitutional AI, clearly document the principles guiding optimization

## The Future

Active research directions include:
- Scaling RLHF to larger models and datasets more efficiently
- Combining RLHF with retrieval-augmented approaches
- Using RLHF to optimize for multiple objectives simultaneously
- Improving transparency and interpretability of learned reward models
- Developing better methods for handling human disagreement

RLHF represents a significant advancement in making language models more useful and aligned with human values. While imperfect, it's currently the most practical approach to bridging the gap between next-token prediction and human preferences.
