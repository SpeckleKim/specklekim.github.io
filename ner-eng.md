---
layout: default
title: Named Entity Recognition
lang: en
---

# Named Entity Recognition

## What is Named Entity Recognition?

Named Entity Recognition (NER) is the task of identifying and classifying named entities in text. A named entity is a real-world object (person, organization, location, date, etc.) that can be denoted by a proper noun. NER automatically extracts structured information from unstructured text, identifying entity boundaries and assigning semantic types.

For example, in the sentence "Apple Inc. was founded by Steve Jobs in Cupertino on April 1, 1976", NER would identify:
- "Apple Inc." → ORGANIZATION
- "Steve Jobs" → PERSON
- "Cupertino" → LOCATION
- "April 1, 1976" → DATE

## Common Entity Types

Standard NER systems typically recognize several entity categories:

- **PERSON**: Individual names (John Smith, Marie Curie)
- **ORGANIZATION**: Company, institution, or group names (Microsoft, UN, Harvard University)
- **LOCATION**: Geographic locations including cities, countries, regions (Paris, Brazil, Silicon Valley)
- **DATE**: Temporal expressions (January 15, 2023, last week, tomorrow)
- **TIME**: Specific times (3:45 PM, noon, midnight)
- **MONEY**: Monetary values and currency (€50, $1.2 million)
- **PERCENT**: Percentages (42%, 0.5%)
- **FACILITY**: Buildings, airports, highways (Golden Gate Bridge, JFK Airport)
- **PRODUCT**: Product names (iPhone 15, Windows 11)
- **EVENT**: Named events (Olympic Games, World War II)

Custom entity types can be defined for domain-specific applications.

## BIO and BIOES Tagging Schemes

NER is typically formulated as a sequence tagging problem where each token receives a tag. The most common tagging scheme is BIO (Begin-Inside-Outside):

- **B-X**: Beginning of entity type X
- **I-X**: Inside (continuation) of entity type X
- **O**: Outside any entity

For example:
```
Apple    → B-ORG
Inc.     → I-ORG
was      → O
founded  → O
by       → O
Steve    → B-PER
Jobs     → I-PER
```

The BIOES scheme adds:
- **E-X**: End of entity type X
- **S-X**: Single-token entity of type X

BIOES is more informative, capturing entity boundaries explicitly, but requires more training data. BIO is simpler and more widely used.

## Conditional Random Fields (CRF)

Conditional Random Fields are probabilistic graphical models used for sequence labeling. Unlike independent classifiers that predict each token's label independently, CRFs model label dependencies, capturing constraints like:
- Certain tag sequences are valid (B-PER → I-PER is valid; B-PER → I-ORG is unlikely)
- Labels depend on neighboring tokens and their labels

A linear-chain CRF models: P(y|x) ∝ exp(Σ_i λ_i f_i(y_{t-1}, y_t, x, t))

Where f_i are feature functions that can capture emissions (token→label) and transitions (label→label). CRFs achieve strong performance, especially on datasets with clear label dependencies.

## BiLSTM-CRF Architecture

A popular and effective approach combines BiLSTM with CRF:

1. **Embedding Layer**: Maps tokens to word embeddings (pretrained or learned)
2. **BiLSTM**: Bidirectional LSTM processes tokens forward and backward, capturing context
3. **CRF Layer**: Models label sequences with transitions, enforcing valid tag sequences

The BiLSTM learns rich contextual representations while the CRF layer ensures structural validity of output sequences. This combination consistently outperforms simpler models and has become a standard baseline for NER.

## BERT for Named Entity Recognition

BERT provides contextualized token representations from bidirectional transformers. For NER, the approach is:

1. **Tokenize**: Split text into BERT wordpiece tokens
2. **Get Representations**: Pass through pretrained BERT layers
3. **Token Classification**: Add a classification layer on top of BERT token representations
4. **Fine-tune**: Train on NER task labels

BERT's contextualized embeddings naturally handle word boundaries, polysemy, and long-range dependencies better than static embeddings. Fine-tuned BERT models achieve state-of-the-art NER performance, often exceeding 90% F1 on standard benchmarks.

## spaCy for NER

spaCy is an industrial-strength NLP library with efficient NER capabilities. It provides:

- **Pretrained Models**: Ready-to-use models for English, German, French, Portuguese, and more
- **Fast Performance**: Optimized C implementations enable processing large document volumes
- **Easy Integration**: Simple Python API for entity extraction
- **Custom Models**: Support for training custom NER models on domain-specific data
- **Entity Linking**: Connect recognized entities to external knowledge bases

Example usage:
```python
import spacy
nlp = spacy.load("en_core_web_sm")
doc = nlp("Apple Inc. was founded by Steve Jobs.")
for ent in doc.ents:
    print(f"{ent.text} → {ent.label_}")
```

## Applications of NER

### Information Extraction
NER is fundamental for extracting structured information from unstructured documents. For example, extracting all companies, people, and locations mentioned in news articles enables automated document organization and search.

### Knowledge Graphs
NER identifies entities that can be linked to create knowledge graphs—structured representations of relationships between real-world objects. Google Knowledge Graph, Wikidata, and similar systems rely on NER as a foundational component.

### Question Answering
NER systems extract candidate answers for factoid questions. When asked "When was Einstein born?", a QA system might use NER to locate DATE entities, improving answer accuracy.

### Relation Extraction
Following entity recognition, relation extraction determines relationships between entities. NER identifies entity mentions; subsequent processing determines relations like "(Einstein, born_in, Germany)".

### Document Classification and Retrieval
Entities mentioned in documents can serve as features for classification and retrieval systems. Documents mentioning "Apple" and "Microsoft" can be grouped differently than those mentioning "Obama" and "Trump".

## Challenges in NER

- **Entity Boundary Detection**: Determining where entities begin and end is non-trivial (is "New York" one entity or two?)
- **Ambiguity**: Entities can belong to multiple categories (Stanford → ORGANIZATION or LOCATION)
- **Domain Adaptation**: Models trained on news fail on medical or scientific texts
- **Rare Entities**: Low-frequency entity types are difficult to recognize
- **Emerging Entities**: New entities constantly appear; models must generalize

## Conclusion

Named Entity Recognition is a foundational NLP task enabling extraction of structured information from text. From early rule-based and statistical approaches through modern neural architectures, NER has evolved to leverage deep contextualized representations. Contemporary BERT-based and transformer models achieve excellent performance, while practical tools like spaCy provide efficient, production-ready solutions. Understanding NER principles and tools is essential for information extraction, knowledge graph construction, and numerous downstream NLP applications.
