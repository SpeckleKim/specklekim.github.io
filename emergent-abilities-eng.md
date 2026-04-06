---
layout: default
title: Emergent Abilities in LLMs
lang: en
---

# Emergent Abilities in LLMs

One of the most fascinating discoveries in [large language models](/llm-eng.html) is the emergence of unexpected abilities at certain scale thresholds. These "emergent abilities" appear suddenly during scaling—seemingly absent in smaller models but robustly present in larger ones. This phenomenon challenges our understanding of how language models acquire capabilities and raises profound questions about model development.

## Defining Emergent Abilities

An emergent ability is formally defined as:

1. **Absent in small models**: Performance near random chance in models below a threshold
2. **Present in large models**: Substantial performance above random in models above threshold
3. **Sharp transition**: Performance jumps non-linearly at a critical scale
4. **Unpredicted**: The ability wasn't explicitly trained for; the model "figured it out"

Key distinction:
- **Smooth scaling**: Ability improves gradually ([scaling law](/scaling-laws-eng.html))
- **Emergence**: Ability appears suddenly (phase transition)

## Classic Examples

### Chain-of-Thought Prompting

The most famous example involves chain-of-thought reasoning:

```
Prompt: "A store has 5 apples. 3 are red, 2 are green.
How many ways can you pick 2 red apples?"

Smaller models (< 10B params):
- Random guessing
- No structured reasoning visible

Larger models (> 50B params):
- "First, I need to pick 2 from 3 red apples"
- "That's C(3,2) = 3 ways"
- Correct reasoning steps
```

Wei et al. (2022) showed chain-of-thought reasoning emerges around 50-100B parameters:
- Below 10B: ~5-10% accuracy on complex reasoning
- 10-50B: ~30-50% accuracy
- Above 100B: ~70-90% accuracy (with chain-of-thought prompting)

### Arithmetic and Symbolic Reasoning

Models demonstrate sudden competence in:
- **Multi-digit arithmetic**: Exact operations on large numbers
- **Algorithmic tasks**: Sorting, searching, pattern completion
- **Code generation**: Writing compilable code from descriptions

These abilities appear at different thresholds:
- Addition: emerges ~2-5B params
- Multiplication: emerges ~10-20B params
- General programming: emerges ~50B+ params

### In-Context Learning

Even more striking: the ability to learn new tasks from examples in the prompt:

```
Few-shot example in prompt:
"fox -> fox_box, dog -> dog_box, cat -> ?"

Smaller models: Can't identify the pattern
Larger models (10B+): Correctly answers "cat_box"

Even more subtle: Models learn task structure without explicit instruction
```

### Translation and Multilingual Tasks

Models suddenly become competent translators at scale:

```
Model size 0.5B: Mostly gibberish translation
Model size 2B: Some correct words, broken syntax
Model size 10B+: Fluent, accurate translation between distant languages

This emerges WITHOUT explicit parallel corpus training (for all language pairs)
```

## The Debate: Real or Artifact?

Not everyone agrees emergent abilities are truly "emergent." This is a heated debate in AI research.

### The "Real Emergence" Camp

Proponents argue:
1. **Discontinuous capability jumps**: Sharp transitions are empirically observable
2. **Unexpected by architects**: Abilities weren't explicitly designed for
3. **Threshold phenomena**: Clear cutoff points in capability
4. **Multi-step reasoning**: Requires compositional understanding not present in smaller models
5. **Fine evidence**: Detailed studies show emergence across many domains

**Key papers**: Wei et al. (2022), Suzgun et al. (2022)

### The "Artifact" Camp

Skeptics argue:
1. **Metric artifacts**: Emergence is partly an artifact of how we measure abilities
   - Accuracy metrics are all-or-nothing (right/wrong)
   - Underlying capability may scale smoothly, but measurement is discrete
   - Example: If passing requires 70% accuracy, improvement from 69%→71% looks like sudden emergence

2. **Task definition**: How we define tasks influences whether emergence appears
   - Same underlying ability may or may not "emerge" depending on measurement
   - Switching metrics (perplexity vs. accuracy) changes emergence observation

3. **Benchmark saturation**: Early [benchmarks](/benchmarks-eng.html) are too easy for large models
   - Smaller models plateau at ceiling
   - Larger models show more granular performance
   - Looks like emergence but may be artifacts

**Key paper**: Schaeffer et al. (2023) - "Are Emergent Abilities of [Large Language Models](/llm-eng.html) a Mirage?"

### Current Consensus

Research suggests **both are true**:

1. **Some emergence is real**: Capabilities like multi-step reasoning likely require scale
2. **Some emergence is measurement artifact**: [Evaluation metrics](/evaluation-metrics-eng.html) hide gradual improvement
3. **Nuance matters**: Whether emergence is "real" depends on definition and measurement
4. **Better metrics needed**: We need finer-grained evaluation to distinguish

Most researchers now focus on:
- Using continuous metrics (loss, perplexity) alongside accuracy
- Studying learning dynamics, not just final performance
- Designing [benchmarks](/benchmarks-eng.html) that reveal gradual scaling, not just thresholds

## Mechanisms Behind Emergence

Several theories explain why abilities emerge at scale:

### Theory 1: Grokking
The model first memorizes training data, then "grok" to abstract patterns:

```
Stage 1: Fit training data (overfitting)
Stage 2: Idle period (generalization learning)
Stage 3: Sudden improvement on test set (emergent ability)
```

This explains delayed emergence even after training loss is zero.

### Theory 2: Compositional Reasoning
Larger models can combine simpler learned patterns:

```
Learned patterns:
- "follow instructions"
- "do math operations"
- "combine steps"

Small model: Can't reliably chain operations
Large model: Composes patterns → multi-step reasoning
```

### Theory 3: Distributed Representation
Scale enables richer, more distinct representations:

```
Small model: Features are tangled, difficult to isolate
Large model: Clear separation of concepts
            → Enables precise reasoning
```

### Theory 4: Phase Transitions
Physical analogy: like phase transitions in physics:

```
Ice ←→ Water ←→ Steam (at temperature thresholds)

Model capabilities similarly may have critical points where:
- Ability to do X suddenly appears
- Correlates with parameter count
- Similar to universality concepts in physics
```

## Implications for Model Development

Understanding emergence has practical consequences:

### 1. Compute Allocation
If abilities emerge at specific thresholds:
- No point training 7B model if 50B is needed for chain-of-thought
- But 50B might have other wasteful parameters
- Suggests targeted development toward capability thresholds

### 2. Task Design
Emergence suggests we should:
- Design [benchmarks](/benchmarks-eng.html) to reveal gradual scaling (not just pass/fail)
- Separate memorization from generalization
- Use multiple [evaluation metrics](/evaluation-metrics-eng.html)

### 3. Scaling Strategy
Teams can plan scaling with emergence in mind:
- Identify critical thresholds for required capabilities
- Budget compute toward those thresholds
- Less focus on marginal gains beyond thresholds

### 4. Training Dynamics
Emergence suggests:
- Don't stop training early (hidden progress during "grokking")
- Model behavior may suddenly change with more scale
- Emergent abilities are unpredictable (research direction!)

## Emergent Abilities vs. Learned Behaviors

Important distinction:

**Emergent ability:**
- Not in training objective
- Not explicitly programmed
- Arises from scale and optimization
- Example: Multi-digit arithmetic without arithmetic operations in training

**Learned behavior:**
- Trained directly from data
- Follows from task specification
- Scales smoothly
- Example: Classification accuracy on labeled data

Most "emergent abilities" are actually capabilities that scale suddenly because:
- Training signal is indirect
- Requires significant capacity to manifest
- Measurement is discrete (pass/fail) rather than continuous

## Open Questions

Despite progress, we don't fully understand emergence:

1. **Predictability**: Can we predict which abilities will emerge at which scales?
2. **Necessity**: Are there fundamental reasons certain abilities require large scale?
3. **Generality**: Do emergence principles transfer across domains?
4. **Controllability**: Can we intentionally induce emergence of desired abilities?
5. **Efficiency**: Can we achieve emerged abilities without massive scaling?

## Conclusion

Emergent abilities in [LLMs](/llm-eng.html)—from chain-of-thought reasoning to in-context learning to translation—represent one of AI's most intriguing phenomena. Whether emergence is truly discontinuous or partially a measurement artifact remains debated, but the phenomenon itself is real and important.

Understanding emergence reshapes how we think about model development: capabilities aren't necessarily smooth functions of scale, unexpected abilities can suddenly appear, and our evaluation methods significantly influence what we observe.

As models continue scaling, studying emergence will remain crucial for predicting capabilities, allocating compute efficiently, and ultimately building more capable, aligned, and useful language models.
