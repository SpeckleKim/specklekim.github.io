---
layout: default
title: Hallucination in LLMs
lang: en
---

# Hallucination in LLMs

## What is Hallucination?

In [large language models](/llm-eng.html), **hallucination** refers to the phenomenon where a model generates plausible-sounding but false, fabricated, or ungrounded information. The model produces text that appears coherent and well-formed but doesn't correspond to reality, violates facts, or contradicts provided context.

Unlike a human who knows the limits of their knowledge, [LLMs](/llm-eng.html) lack explicit reasoning about what they do or don't know. They optimize for prediction [probability](/probability-eng.html), not factual accuracy, making hallucination inevitable without careful mitigation.

### Why This Matters

Hallucinations undermine trust in AI systems. A model that confidently cites non-existent papers, invents [statistics](/probability-eng.html), or claims false biographical details is unreliable for critical applications. Understanding hallucination is essential for responsible deployment.

## Types of Hallucination

Researchers categorize hallucinations into two main types, though the distinction isn't always sharp.

### Intrinsic Hallucination

The model generates text that **contradicts its input context**. Despite being given information, the model ignores or contradicts it.

**Examples:**
- Given: "John is a software engineer born in 1985"
- Hallucination: "John is a doctor born in 1990"
- Given: "The company has 50 employees"
- Hallucination: "The company has 200 employees"

This is more obviously a failure—the model had relevant information but didn't use it correctly.

### Extrinsic Hallucination

The model generates text that **cannot be verified or is false** relative to world knowledge, even though it doesn't directly contradict provided input.

**Examples:**
- "Dr. Sarah Chen published a paper on quantum computing in Nature magazine in 2022" (person/paper may not exist)
- "The capital of Freedonia is New Libertyville" (Freedonia is fictional)
- "[GPT](/gpt-eng.html)-8 was released in March 2024" (false claim about AI models)

Extrinsic hallucinations are harder to detect because the model isn't explicitly violating context—it's fabricating from broader knowledge gaps or misconceptions.

## Why Hallucinations Happen

### Training Objective Mismatch

Language models optimize for **next-token prediction probability**, not factual accuracy. The model learns patterns that produce fluent text, but fluency ≠ truth. A coherent sentence about a non-existent entity achieves high [probability](/probability-eng.html) by statistical patterns without requiring real-world knowledge.

### Knowledge Gaps and Confusion

Models trained on internet data face inherent uncertainty:
- Incomplete or conflicting information in training data
- Rare entities with limited training examples
- Fine-grained distinctions that training data didn't clearly distinguish
- Confusion between similar-sounding entities or concepts

### Decoding Effects

Generating text is a sequential process. Early tokens constrain later ones. If the model makes an early commitment (e.g., "The researcher discovered..."), it may follow through with fabricated details to maintain coherence, even if those details are false.

### Domain-Specific Challenges

- **Long context**: Models may lose track of information provided earlier
- **Numerical reasoning**: Computing precise numbers isn't language modeling's strength
- **Rare facts**: Information appearing in few training examples is error-prone
- **Reasoning chains**: Multi-step reasoning amplifies errors

## Detecting Hallucinations

### Surface-Level Detection

Some hallucinations are obvious:
- **Contradictions within output**: Model says X then says not-X
- **Factual inconsistency with input**: Clearly contradicts provided facts
- **Internal logical inconsistency**: Statements violate reasoning rules

These are detectable through basic consistency checking.

### Factuality Metrics

Research has developed metrics to measure hallucination:

**ROUGE/BLEU variants**: Compare generated text to reference summaries. High similarity suggests lower hallucination, but this assumes good reference data exists.

**Semantic similarity**: Use embedding models to check if generated claims align with reference knowledge bases.

**QA-based evaluation**: For a generated statement, ask "Can a model trained on reliable sources answer this?" Models performing poorly indicate hallucination.

**Entailment detection**: Does the output logically follow from input? Textual entailment models can assess this.

### Human Evaluation

Gold standard: have humans verify generated facts. Expensive but most reliable. Often combined with metrics to prioritize which outputs to evaluate.

## Mitigation Strategies

### Retrieval-Augmented Generation (RAG)

Instead of relying solely on the model's internal knowledge, **augment generation with external retrieval**.

1. For a query, retrieve relevant documents from a knowledge base
2. Provide these documents as context to the model
3. Model generates answers grounded in retrieved information
4. Model can cite sources directly

**Benefits**: Directly addresses extrinsic hallucinations by providing factual grounding.

**Limitations**: Depends on quality of retrieval. Poor retrieval → poor grounding. Still doesn't eliminate hallucination entirely.

### Chain of Verification

Have the model verify its own outputs:
1. Generate an initial response
2. List claims made in the response
3. Verify each claim against input or knowledge
4. Revise response if needed

This forces explicit reasoning about accuracy and can catch some hallucinations, though the model may consistently fail at both generation and verification.

### Constitutional AI and Principles

Anthropic's approach: train models with principles (e.g., "be helpful, harmless, and honest") and have them reason about whether outputs violate these principles. A critique phase judges outputs before generation, reducing hallucination.

### Fine-tuning on High-Quality Data

Models trained on clean, well-annotated data hallucinate less. But scaling and data quality are perpetual challenges.

### Grounding Through Formats

Constrain generation to specific formats:
- Require citations with every claim
- Use structured output (JSON) that's easier to validate
- Force models to say "I don't know" when uncertain

Explicit grounding makes hallucination harder and mistakes more obvious.

## Faithfulness Metrics

Researchers have proposed metrics specifically for measuring **faithfulness**—how well generated text reflects facts in source material.

### Atomic Fact Extraction

Break text into atomic factual claims, then verify each:
1. "The patient has diabetes" → atomic facts include "has disease: diabetes"
2. Check each atomic fact against source
3. Hallucination rate = (false facts) / (total facts)

This is more granular than whole-text evaluation.

### Token-Level Attribution

Advanced approaches assign attribution: for each generated token, which source tokens does it depend on? Tokens without clear attribution may indicate hallucination.

### Semantic NLI

Use Natural Language Inference (NLI) models to check if claims in output are entailed by input. If output contradicts or isn't supported by input, flag it.

## Challenges and Trade-offs

### The Knowledge Problem

Models need broad knowledge to be useful, but broad knowledge includes hallucinations. Constraining knowledge (via [RAG](/rag-eng.html)) helps but makes models less flexible.

### Uncertainty Quantification

Models confidently hallucinate. They don't distinguish "I'm confident and right" from "I'm confident and wrong." Learning to express uncertainty is difficult and an open research problem.

### Measuring Hallucination

No single metric captures all aspects. Evaluation is often task-specific. A metric for [summarization](/text-summarization-eng.html) may not work for dialogue, translation, or [code generation](/code-generation-eng.html).

### The Honesty Gap

Even with fine-tuning, models may hallucinate less but still lack true honesty about knowledge boundaries. They learn patterns of saying "I don't know" without genuinely understanding uncertainty.

## Best Practices for Practitioners

1. **Choose the right tool**: If factuality is critical, use [RAG](/rag-eng.html) with trusted sources rather than relying on model knowledge alone.

2. **Set expectations**: Users should understand models can hallucinate. Appropriate disclaimers matter.

3. **Verify for high-stakes decisions**: For medical, legal, financial, or safety-critical applications, always verify generated information against authoritative sources.

4. **Use uncertainty indicators**: Prompt models to express confidence levels. Low confidence suggests verification is needed.

5. **Combine with retrieval**: Even for creative tasks, grounding in documents reduces hallucination.

6. **Monitor and iterate**: Track hallucination rates in deployment. Use feedback to fine-tune or adjust prompting strategies.

Hallucination is an active area of research. While mitigation strategies exist, eliminating it entirely remains an open challenge.
