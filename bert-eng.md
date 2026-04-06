---
layout: default
title: BERT & Masked Language Modeling
lang: en
---

# BERT & Masked Language Modeling: A Paradigm Shift

BERT (Bidirectional Encoder Representations from [Transformers](/llm-eng.html)) fundamentally changed how we approach NLP. Rather than training models for specific tasks, BERT showed that massive pretraining on unlabeled data, followed by simple fine-tuning, could achieve state-of-the-art results across virtually every downstream task. This paradigm—pretrain, then fine-tune—became the foundation of modern NLP.

## The Bidirectionality Insight

Previous language models were unidirectional:

```
Unidirectional (left-to-right):
"The bank manager approved..."
 ↑   ↑               ↑
When predicting "manager", only see "The bank"
Missing right-context: "approved"
```

BERT uses bidirectional context:

```
Bidirectional:
"The bank manager approved..."
 ↑   ↑     ↑            ↑
When encoding "manager":
- See left context: "The bank"
- See right context: "approved..."
- Much richer representation!
```

**The challenge:** Standard language modeling (predicting next token) is inherently unidirectional. How do you train bidirectional models?

BERT's breakthrough: masked language modeling.

## Masked Language Modeling (MLM)

Rather than predicting the next token, BERT randomly masks tokens and predicts them:

```
Original: "The quick brown fox jumped over the lazy dog"

Masked:  "The quick [MASK] fox jumped over the [MASK] dog"
         (15% of tokens randomly masked)

Task: Predict the masked tokens ("brown", "lazy")
```

**Masking strategy:**
- 15% of tokens selected for masking
- 80% of selected tokens replaced with [MASK]
- 10% replaced with random token
- 10% left unchanged

**Why not 100% [MASK]?**
- The model needs to see real tokens to learn meaningful representations
- 80% [MASK], 10% random, 10% unchanged provides robust training signal
- This prevents the model from just memorizing the [MASK] token

**Training objective:**
```
Loss = -log P(masked_tokens | context)
```

Predict the original tokens given the surrounding context (both left and right).

## NSP: Next Sentence Prediction

BERT adds a second pretraining task to learn sentence-level semantics:

```
Example 1 (IsNext):
Sentence A: "The man went to the store."
Sentence B: "He bought a loaf of bread."
Label: IsNext (B follows A in document)

Example 2 (NotNext):
Sentence A: "The man went to the store."
Sentence B: "Penguins are flightless birds."
Label: NotNext (random sentence)
```

**Architecture:**
```
Input:  [CLS] sent_A_tokens [SEP] sent_B_tokens [SEP]
         ↓         ↓          ↓           ↓
[CLS] is a special classification token
[SEP] marks sentence boundaries

Output: [CLS] representation fed to binary classifier
        IsNext or NotNext
```

**Contribution to downstream tasks:**
NSP helps with tasks requiring sentence relationships:
- [Question answering](/question-answering-eng.html) (question relates to context)
- Natural language inference (premise relates to hypothesis)
- Paraphrase detection (sentences semantically similar)

Later research (RoBERTa) showed NSP's contribution is modest, and many models dropped this task.

## BERT Architecture

BERT is a [transformer](/llm-eng.html) encoder stack:

```
Input Layer (Tokenization + Embedding)
    ↓
Transformer Encoder Block 1 (Multi-head attention + FFN)
    ↓
Transformer Encoder Block 2
    ↓
... (12-24 blocks for different BERT sizes)
    ↓
Output Layer (Embeddings for each token position)
```

**Key components:**

1. **WordPiece Tokenization**: Subword [tokenization](/tokenization-eng.html) with [vocabulary](/tokenizer-design-eng.html) of ~30K
2. **Token Embeddings**: Each token → dense vector
3. **Segment Embeddings**: Token-level embedding indicating which sentence (A or B)
4. **Position Embeddings**: Absolute position (1 to max_length)
5. **Transformer Layers**: [Self-attention](/self-attention-eng.html) and feed-forward networks
6. **Layer Normalization**: Stabilizes training

**BERT sizes:**
- BERT-Base: 12 layers, 768 hidden dimensions, 12 attention heads (~110M parameters)
- BERT-Large: 24 layers, 1024 hidden dimensions, 16 attention heads (~340M parameters)

## Fine-tuning for Downstream Tasks

BERT's pretraining learns general language understanding. Task-specific fine-tuning adapts this to downstream applications.

**Text Classification:**
```
[CLS] sentence tokens [SEP]
        ↓
     BERT encoding
        ↓
   [CLS] representation
        ↓
   Linear classifier
        ↓
   Class probabilities
```

Simple: just add a linear layer on top of [CLS] token representation.

**Named Entity Recognition:**
```
[CLS] The cat sat [SEP]
        ↓
     BERT encoding
        ↓
Per-token outputs (one label per token)
```

Use output at each token position for tagging.

**Question Answering:**
```
[CLS] question [SEP] passage [SEP]
        ↓
     BERT encoding
        ↓
Predict start and end positions of answer
```

BERT learns to span the answer within the passage.

**Fine-tuning approach:**
1. Initialize with pretrained BERT weights
2. Add task-specific layer(s)
3. Train on labeled task data (typically 5-10 epochs)
4. [Learning rate](/learning-rate-scheduling-eng.html): small (2e-5 to 5e-5) to preserve pretrained knowledge

## BERT Variants and Improvements

### RoBERTa (Facebook/Meta)

RoBERTa improved BERT through:
- **Longer training**: More pretraining steps
- **Larger batches**: Better gradient estimates
- **Removed NSP**: Task provides minimal benefit
- **Improved masking**: Dynamic masking (mask pattern changes per epoch)
- **Cleaned data**: Higher quality pretraining corpus

**Result:** Consistently outperforms original BERT.

### ALBERT (Google)

ALBERT introduced parameter sharing for efficiency:
- **Factorized embeddings**: Separate embedding and hidden dimensions (smaller embedding, full-size hidden)
- **Cross-layer parameter sharing**: All [transformer](/llm-eng.html) layers share parameters
- **Result:** 10x fewer parameters than BERT-Large, similar performance

### DeBERTa (Microsoft)

DeBERTa improved attention:
- **Disentangled attention**: Separate position and content attention
- **Enhanced mask decoder**: Better handling of masked tokens
- **Result:** State-of-the-art on GLUE [benchmark](/benchmarks-eng.html) (beating RoBERTa)

### ELECTRA (Google)

ELECTRA uses discriminative rather than generative pretraining:
- **Generator**: Small language model creates corrupted text
- **Discriminator**: Larger model detects which tokens were replaced
- **Result:** More efficient pretraining; better performance with less compute

Other variants address domain specificity:
- **SciBERT**: Scientific papers
- **ClinicalBERT**: Medical text
- **FinBERT**: Financial documents
- **LegalBERT**: Legal documents

## Why BERT Revolutionized NLP

### 1. **Universal Representation**

Single pretrained model works across tasks. No need for task-specific architectures.

### 2. **Unprecedented Scale**

Pretraining on massive unlabeled text (Wikipedia + Books corpus, 3.3B tokens) learns deep linguistic structure.

### 3. **Simple Fine-tuning**

Minimal task-specific architecture. Often just a linear layer on pretrained representations.

### 4. **Exceptional Performance**

BERT achieved state-of-the-art on:
- GLUE (General Language Understanding Evaluation) [benchmark](/benchmarks-eng.html)
- SQuAD ([question answering](/question-answering-eng.html))
- [NER](/ner-eng.html) ([named entity recognition](/ner-eng.html))
- [Sentiment analysis](/sentiment-analysis-eng.html)
- And many more

### 5. **Accessibility**

BERT's open-source release democratized NLP research. Anyone could fine-tune on their task.

## Limitations and Evolution

### BERT Limitations:

1. **Unidirectional generation**: Cannot generate text left-to-right efficiently
2. **Fixed sequence length**: Max 512 tokens (though can extend)
3. **Independence assumption**: Masked positions predicted independently (not jointly)
4. **Static embeddings**: Per-token representations don't change within sequence

### Subsequent Improvements:

These limitations motivated development of:
- **GPT models**: Generative (unidirectional) pretraining for generation
- **T5**: Treats all NLP as text-to-text, handling generation better
- **XLNet**: Permutation language modeling (more flexible context)
- **ELECTRA, DeBERTa**: More efficient and effective pretraining

## Modern Impact

In 2024, BERT's direct influence has diminished as newer models ([GPT](/gpt-eng.html)-4, Claude, Llama) dominate. However:

1. **Conceptual foundation**: Pretrain-then-fine-tune is still dominant paradigm
2. **Encoder-only models**: Still used for efficiency-critical applications
3. **Multilingual variants**: mBERT, XLM-R still valuable for non-English
4. **Educational value**: Understanding BERT is essential for modern NLP

## Conclusion

BERT fundamentally changed NLP by demonstrating that pretraining on masked language modeling, followed by task-specific fine-tuning, could achieve exceptional results. The key insights—bidirectional context, massive pretraining, simple fine-tuning—became the blueprint for modern language models.

While specific BERT models are less common in production now, the paradigm it established remains central to NLP in 2024. Understanding BERT is essential for comprehending modern language models, their capabilities, and their limitations. BERT's legacy is not in the specific architecture, but in showing that NLP's future belonged to large-scale pretrained models fine-tuned for specific tasks.

