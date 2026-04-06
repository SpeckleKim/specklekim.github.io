---
layout: default
title: Question Answering Systems
lang: en
---

# Question Answering Systems: From Extraction to Generation

Question Answering (QA) is a core NLP task requiring systems to locate, extract, or generate answers to natural language questions. Modern QA systems power search engines, virtual assistants, and knowledge-seeking applications, advancing from simple pattern matching to sophisticated neural architectures capable of complex reasoning.

## Task Formulations

QA systems can be categorized along multiple dimensions:

**Extractive QA** locates answer spans within provided context, selecting start and end positions. This approach guarantees answers come from the source material, ensuring factuality but limiting answer expressiveness.

**Generative QA** produces novel answer text, not constrained to source material. This enables more natural, concise answers but introduces [hallucination](/hallucination-eng.html) risks and requires more careful evaluation.

**Closed-book QA** relies purely on parameters learned during training, answering without external context. This tests pure memorization and reasoning capabilities.

**Open-book QA** provides relevant documents or passages, requiring the system to locate and utilize this information. Most practical systems operate in this mode.

## Extractive QA: The SQuAD Era

The Stanford Question Answering Dataset (SQuAD) revolutionized the field by establishing extractive QA as a well-defined [benchmark](/benchmarks-eng.html). SQuAD 2.0 added unanswerable questions, requiring systems to recognize when context contains no valid answer. Extractive systems typically employ two steps: retrieving relevant passages and identifying answer spans within those passages.

## BERT for Question Answering

[BERT](/bert-eng.html) achieved state-of-the-art performance on SQuAD and similar datasets through fine-tuning on QA tasks. The approach uses special tokens ([CLS] and [SEP]) to separate questions and passages, then adds a span classification layer predicting start and end token positions. BERT's bidirectional contextual representations excel at understanding relationships between questions and context, making it particularly effective for extractive QA.

## Retriever-Reader Architecture

The retriever-reader pipeline separates QA into two stages: retrieval and reading comprehension. The **retriever** component ranks documents by relevance, typically using BM25 or dense passage retrieval. The **reader** component, often a fine-tuned [BERT](/bert-eng.html) variant, finds answers within the top-ranked passages. This modular approach scales efficiently to large document collections while maintaining accuracy.

Dense passage retrieval uses learned embeddings to compute similarity between questions and passages, enabling semantic search beyond keyword matching. Contrastive learning approaches like DPR (Dense Passage Retriever) train embeddings that bring relevant passages close to questions in embedding space while pushing irrelevant passages apart.

## Open-Domain Question Answering

Open-domain QA removes the assumption of having a small [context window](/context-window-eng.html), instead requiring systems to search through millions of documents. These systems combine retrieval and comprehension: first retrieving candidate documents using BM25 or dense retrieval, then applying reader models to find answers. The challenge involves handling noisy retrieval results and errors that propagate to downstream components.

## Multi-Hop Question Answering

Multi-hop reasoning requires combining information across multiple documents or passages to answer complex questions. For example, "Who is the spouse of the person who directed Movie X?" requires locating Movie X, identifying its director, then finding their spouse. Systems must maintain context across multiple reasoning steps and recognize relevant connections between entities.

## Conversational Question Answering

Conversational QA handles follow-up questions within a dialogue context, requiring systems to interpret pronouns, implicit references, and question reformulations. The QUAC and CoQA datasets address this challenge, requiring QA systems to track conversation history and resolve ambiguous references. Historical context becomes part of the input, and systems must maintain consistency across turns.

## Generative QA and seq2seq Models

[Seq2seq](/seq2seq-eng.html) models with attention can generate natural language answers unconstrained by source text. Encoder-decoder architectures process questions and context, generating answer tokens sequentially. These systems produce more fluent answers but require careful training to avoid [hallucination](/hallucination-eng.html) and maintain factual consistency with the provided context.

## Challenges and Future Directions

**Retrieval errors** in open-domain settings propagate to answer extraction, requiring robust mechanisms for handling noisy documents. **Cross-lingual QA** extends English-centric systems to multilingual settings, necessitating cross-lingual transfer or multilingual models. **Zero-shot and few-shot QA** reduce dependence on large task-specific labeled datasets through in-context learning.

**Knowledge integration** remains challenging, as purely parametric models may struggle with rare facts or require prohibitively large parameter counts. **Adversarial robustness** ensures systems resist carefully crafted questions designed to elicit errors.

Modern [large language models](/llm-eng.html) demonstrate impressive few-shot QA capabilities, answering complex questions without task-specific fine-tuning. However, they exhibit brittleness to adversarial inputs and may hallucinate plausible-sounding but incorrect answers. The field continues evolving toward systems that combine retrieval, reasoning, and generation while maintaining interpretability and factual grounding.
