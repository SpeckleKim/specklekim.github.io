---
layout: default
title: Instruction Tuning
lang: en
---

# Instruction Tuning

## From Pretraining to Instruction Following

Large language models trained on next-token prediction alone demonstrate remarkable abilities, but they often fail at basic task-following. A model might complete Shakespeare passages beautifully but struggle to answer "What is the capital of France?" when phrased as an instruction.

**Instruction tuning** bridges this gap. It's a fine-tuning phase where models learn to follow explicit instructions and produce task-specific outputs. Rather than just predicting the next likely token, models learn to understand tasks, follow constraints, and deliver useful responses.

### Why Pretraining Alone Isn't Enough

Pretraining on raw internet data optimizes for compression—predicting likely continuations. This builds broad knowledge but doesn't directly optimize for instruction following. A model trained purely on web text might:
- Fail to recognize questions as requiring answers
- Continue text rather than summarize it
- Miss implicit task constraints
- Ignore special formatting instructions

Instruction tuning explicitly teaches these behaviors.

## The Instruction Tuning Dataset

The foundation of instruction tuning is a **carefully constructed dataset of (instruction, input, output) triples**.

### Data Format

A typical instruction-tuning example:
```
Instruction: Summarize this text in one sentence.
Input: The evolution of artificial intelligence has progressed
       through multiple phases...
Output: AI has evolved through several phases, advancing from
        narrow specialist systems to more general capabilities.
```

Or more concisely (for input-less tasks):
```
Instruction: What is photosynthesis?
Output: Photosynthesis is the process by which plants convert
        light energy into chemical energy...
```

### Dataset Scale and Diversity

Effective instruction tuning requires:
- **Quantity**: Thousands to hundreds of thousands of examples (typically 1K-100K)
- **Diversity**: Examples spanning multiple task types (summarization, QA, reasoning, translation, etc.)
- **Quality**: High-quality outputs with minimal errors
- **Task variation**: Different phrasings of the same task to teach robustness

## Notable Instruction-Tuning Approaches

### FLAN (Finetuning Language Models with Annotated Natural Languages)

Google's FLAN dataset provides diverse instruction examples across dozens of task categories. FLAN combines:
- Original datasets (hundreds of existing benchmarks)
- Templates converting datasets to instruction format
- Task-specific variations

**Key insight**: Instead of training on raw Wikipedia, fine-tune on formatted instructions derived from established task collections. This teaches models to follow instructions while maintaining performance on original tasks.

### InstructGPT

OpenAI's InstructGPT demonstrated that instruction tuning with human feedback significantly improves usefulness and safety:

1. **Supervised Fine-Tuning (SFT)**: Fine-tune base model on high-quality instruction examples
2. **Reward Model Training**: Train a separate model to predict which outputs humans prefer
3. **Reinforcement Learning**: Use PPO to update the model to maximize rewards

This three-stage process, combined with human feedback, produced more aligned and capable models.

## Creating Instruction-Tuning Datasets

### Crowdsourcing and Human Annotation

Hire annotators to write instructions and provide high-quality outputs:
1. Annotators write diverse instructions
2. Annotators provide gold-standard outputs
3. Quality control removes low-quality examples
4. Validation ensures diversity and coverage

This is labor-intensive but produces high-quality data.

### Conversion from Existing Datasets

Many benchmark datasets (SQuAD, GLUE, SuperGLUE) can be converted to instruction format:

**SQuAD to instruction format:**
```
Instruction: Answer the question based on the passage.
Input: Passage: [...] Question: [...]
Output: [Answer]
```

**GLUE sentiment task to instruction:**
```
Instruction: Classify the sentiment as positive or negative.
Input: This movie was amazing!
Output: positive
```

### Self-Instruct

Rather than relying solely on human annotation, **self-instruct** uses the model itself to generate instructions and outputs:

1. Start with a small seed set of human-written instructions
2. Use the model to generate new instructions in similar styles
3. The model answers these self-generated instructions
4. Filter low-quality outputs; add to training set
5. Iterate to expand the dataset

This reduces human annotation burden while maintaining diversity.

## Template Formatting

### Structuring Inputs and Outputs

A well-designed template helps models understand task structure:

**Standard format:**
```
### Instruction:
{instruction}

### Input:
{input}

### Response:
{output}
```

**Chat format:**
```
Human: {instruction}
{input}
Assistant: {output}
```

**Prompt format:**
```
Q: {instruction}
A: {output}
```

Different templates teach different behaviors. Chat format naturally teaches conversational behavior, while Q&A format is suited for factual tasks.

## Multi-Task Instruction Tuning

Instruction tuning is most effective when combining **diverse task types** in a single training run.

### Why Multitask Matters

Training on a variety of tasks:
- Prevents overfitting to a single task distribution
- Teaches the model to adapt across domains
- Improves generalization to unseen tasks
- Creates more robust, flexible models

### Task Distribution

Practitioners typically:
- Include 10-50+ distinct task types
- Weight tasks by importance or frequency
- Oversample tasks where performance lags
- Balance between breadth and depth

A well-designed curriculum mixes foundational tasks (QA, summarization) with specialized ones (reasoning, code generation).

## Instruction Tuning Principles

### Clear and Specific Instructions

Models learn from instruction quality. Effective instructions:
- State the task explicitly
- Provide constraints clearly
- Give examples when helpful
- Avoid ambiguity

**Poor:** "Write about AI"
**Better:** "Write a 3-paragraph introduction to machine learning, suitable for beginners, with no technical jargon."

### Output Quality Matters

The quality of training outputs directly impacts model performance. Typical practices:
- Use human experts for high-stakes domains
- Implement quality control processes
- Verify factuality for knowledge-based tasks
- Maintain consistency in style and formatting

### Scale Efficiency

Instruction tuning is surprisingly efficient:
- Modest amounts (1K-10K examples) can meaningfully improve performance
- Adding more task diversity often helps more than adding more examples per task
- A well-curated 5K example dataset beats a poorly curated 50K dataset

## Challenges and Considerations

### Task Leakage

If instruction-tuning data contains examples from evaluation benchmarks, performance numbers are inflated. This is a common pitfall.

### Instruction Following vs. Knowledge

Instruction tuning teaches *task-following*, not knowledge. It can't teach facts a model never learned during pretraining. This is why models sometimes follow instructions to produce false information.

### Generalization to Unseen Tasks

A key research goal: can instruction tuning on diverse tasks enable zero-shot performance on entirely new tasks? Early results (InstructGPT, FLAN) suggest yes, but generalization has limits.

### Computational Cost

Instruction tuning requires:
- High-quality dataset creation (expensive)
- Significant compute for full-model fine-tuning
- Careful hyperparameter tuning per model/domain

Smaller models may need more instruction tuning data to match larger models' instruction-following ability.

## Emerging Techniques

**Prompt tuning**: Rather than updating model weights, learn task-specific prompt embeddings. More efficient but less powerful than full instruction tuning.

**Instruction inversion**: Instead of writing instructions for outputs, learn to generate instructions that should produce given outputs. Useful for understanding what models learn.

**Conditional training**: Weight loss differently for different tasks or instruction types, optimizing for specific performance goals.

Instruction tuning has become essential for building practical, useful language models. By explicitly teaching task-following, practitioners transform raw language models into capable assistants.
