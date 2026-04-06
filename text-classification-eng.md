---
layout: default
title: Text Classification
lang: en
---

# Text Classification

## Overview

Text classification is the task of assigning predefined categories or labels to text documents. It is one of the most fundamental problems in natural language processing with wide-ranging applications including sentiment analysis, spam detection, topic classification, and intent recognition. The goal is to learn a mapping from text input to discrete class labels.

## Bag-of-Words (BoW)

The Bag-of-Words model represents text as an unordered collection of words, ignoring grammar and word order. Each document is represented as a vector where each dimension corresponds to a unique word in the vocabulary, and the value represents the word's frequency in that document.

**Advantages:**
- Simple and interpretable
- Fast to compute
- Works reasonably well for many classification tasks

**Disadvantages:**
- Loses word order information
- Treats "good not bad" and "bad not good" identically
- Ignores semantic relationships between words
- Sensitive to very frequent common words

## TF-IDF (Term Frequency-Inverse Document Frequency)

TF-IDF improves upon bag-of-words by weighting terms based on their importance. A term's TF-IDF score is high if it appears frequently in a document but rarely across the entire corpus, reducing the influence of common words.

The formula is: TF-IDF(t,d) = TF(t,d) × IDF(t)

Where:
- TF(t,d) = frequency of term t in document d
- IDF(t) = log(total documents / documents containing t)

This weighting scheme captures the intuition that rare, distinctive words are more informative for classification than common words.

## CNN for Text Classification

Convolutional Neural Networks can be adapted for text by applying 1D convolutions over word embeddings. A typical architecture includes:

1. **Embedding Layer**: Maps words to dense vectors (pre-trained word embeddings or learned)
2. **Convolutional Layer**: Applies filters of varying widths to capture n-gram patterns
3. **Max Pooling**: Extracts the most important feature for each filter
4. **Fully Connected Layer**: Maps pooled features to class logits

CNNs efficiently capture local word patterns and are computationally efficient. They excel at identifying key phrases and patterns that indicate document class.

## RNN/LSTM for Text Classification

Recurrent Neural Networks process sequences iteratively, maintaining a hidden state that accumulates information from previous tokens. Long Short-Term Memory (LSTM) units address the vanishing gradient problem, enabling networks to capture long-range dependencies.

A typical RNN/LSTM classifier:
1. Embeds words into vectors
2. Processes sequence through LSTM cells
3. Uses the final hidden state or a pooled representation for classification
4. Passes through a fully connected layer to produce class probabilities

LSTMs capture word order and long-range semantic dependencies, outperforming CNNs on many datasets where context matters.

## BERT Fine-Tuning for Classification

BERT (Bidirectional Encoder Representations from Transformers) revolutionized NLP by providing contextualized word representations from a bidirectional transformer. For classification tasks, the standard approach is:

1. **Tokenize**: Break text into BERT tokens
2. **Add Classification Token**: Prepend [CLS] token
3. **Run BERT**: Pass through pretrained layers
4. **Extract Representation**: Use the [CLS] token's representation
5. **Fine-tune**: Add a classification head and train on the downstream task

This approach achieves state-of-the-art performance by leveraging rich contextual representations learned during pretraining on massive text corpora.

## Multi-Label Classification

Multi-label classification assigns multiple labels to each document, unlike binary or multi-class classification which assigns a single label. For example, a news article might be labeled as both "politics" and "economy".

Key differences:
- **Loss Function**: Use binary cross-entropy instead of categorical cross-entropy
- **Evaluation**: Precision, recall, and F1 are computed per label
- **Threshold**: Often requires a probability threshold (e.g., 0.5) to binarize predictions

Multi-label problems require careful handling of label dependencies and class imbalance.

## Sentiment Analysis

Sentiment analysis determines whether text expresses positive, negative, or neutral sentiment. It's widely used for monitoring brand perception, analyzing customer reviews, and tracking public opinion.

**Challenges:**
- Sarcasm and irony often reverse sentiment
- Context-dependent sentiment (a feature can be negative or positive)
- Mixed sentiments within single documents
- Domain adaptation (models trained on reviews may fail on tweets)

Modern approaches use BERT and similar transformers fine-tuned on sentiment datasets, achieving 90%+ accuracy on standard benchmarks.

## Spam Detection

Spam classification aims to identify unwanted or malicious messages (emails, SMS, social media posts). This is a binary classification task with important real-world impact.

**Approaches:**
- **Traditional**: TF-IDF + logistic regression or SVM
- **Deep Learning**: LSTM or BERT-based classifiers
- **Ensemble**: Combine multiple models for robustness

Key features include sender reputation, message content (suspicious links, common spam phrases), and user behavior patterns.

## Topic Classification

Topic classification assigns documents to predefined topic categories (news topics, product categories, etc.). Unlike sentiment which is about emotional content, topic classification focuses on semantic content.

**Approaches:**
- **Supervised**: Train a classifier on labeled documents (CNN, LSTM, BERT)
- **Unsupervised**: Use topic modeling (Latent Dirichlet Allocation)
- **Zero-shot**: Use pretrained models with prompt-based classification

Topic classification is foundational for content organization, recommendation systems, and content moderation.

## Conclusion

Text classification encompasses a spectrum of techniques from simple statistical methods (BoW, TF-IDF) to complex deep learning approaches (BERT, transformers). The choice of method depends on available data, computational resources, and task requirements. Modern approaches leverage pretrained transformers for state-of-the-art results, while traditional methods remain valuable for resource-constrained scenarios. Understanding the full spectrum of techniques enables practitioners to select appropriate tools for their specific classification challenges.
