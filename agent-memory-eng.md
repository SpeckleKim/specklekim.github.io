---
layout: default
title: "Memory Systems for Agents"
lang: en
---

# Memory Systems for Agents

## Introduction

AI agents need memory to maintain context, learn from past interactions, and make informed decisions. Unlike humans who have embodied memory spanning years, AI agents rely entirely on explicit memory systems. Effective agent memory encompasses short-term context, long-term knowledge, episodic recall, and semantic understanding—each serving different roles in agent cognition.

## Short-Term Memory (Context Window)

### Definition

The sequence of tokens the model currently processes:
- Typical context window: 4k-200k tokens
- Latest LLMs (GPT-4 Turbo, Claude 3): 100k-200k tokens
- Larger context = more conversation history accessible

### Challenges

**Token counting**: Each turn of conversation uses tokens (prompt + response)
- Typical turn: 500 tokens
- Long conversation: quickly exhausts context

**Attention complexity**: Processing long sequences is O(n²) complexity
- Longer sequences = slower inference
- Memory-bandwidth bottleneck

### Optimization Strategies

**Summarization**: Periodically summarize old context
```
Old context: [1000 tokens of detailed conversation]
Summary: "User discussed quarterly revenue targets; prefers Q3 focus"
New context: [summary + recent messages]
```

**Sliding window**: Keep only most recent N messages
- Trade-off: lose older context
- Good for conversational agents

**Hierarchical compression**: Multi-level summary (minute → hour → day)
- Preserves high-level patterns
- Enables long-term memory access

## Long-Term Memory (Vector Stores)

### Semantic Memory

Structured knowledge about facts and concepts:
- Knowledge graph: "Python is a programming language"
- Embeddings: Dense vector representations
- Retrieved based on semantic similarity

**Storage**: Vector databases (Pinecone, Weaviate, Milvus)

**Retrieval**: 
```
User query: "best practices for web development"
Embedding: convert to dense vector
Search: find similar embedded knowledge
Return: top 5 relevant facts/documents
```

### Episodic Memory

Memories of specific events and interactions:
- "User X asked about quarterly revenue on March 15"
- Timestamped with context
- Enables personalization and consistency

**Implementation**:
- Store interaction transcripts in vector DB
- Embed interactions, retrieve past similar interactions
- Provide context to current interaction

## Memory Types

### Working Memory

Temporary storage for current problem-solving:
```
Task: Calculate customer lifetime value
Working memory:
  - Customer ID: 12345
  - Purchase history: [list of purchases]
  - Current time: March 2024
  - Formula: sum(purchases) * (1 + retention_rate)
```

- Used during active computation
- Discarded after task completion

### Semantic Memory

Persistent, abstract knowledge:
- "Machine learning requires training data"
- "Interest rates affect loan approval"
- Domain-specific facts

**Storage strategy**: Knowledge graph + embeddings

### Episodic Memory

Specific events and their context:
- "Conversation with user on March 15"
- "Model achieved 95% accuracy in test run"
- "System encountered timeout error at 2:30 PM"

**Storage strategy**: Timestamped, searchable database

## Memory Management

### Storage Limits

Memory systems can't grow infinitely:

**Hard limits**: Database size constraints
**Practical limits**: Cost, retrieval latency

**Strategies**:
- Expiration: Discard old memories after time T
- Compression: Summarize old memories
- Archiving: Move to slower storage when old
- Pruning: Remove redundant memories

### Retrieval Efficiency

Finding relevant memories quickly:

**Vector similarity search**:
- User question → embedding → search vector DB → retrieve similar memories
- Fast: typically 10-100ms

**Hybrid search**:
- Combine vector similarity + keyword search
- Better coverage

### Memory Prioritization

Some memories more important than others:

**Frequency-based**: Recent/frequently accessed memories stay longer

**Importance-based**: Critical memories (security, user preferences) always available

**Relevance-based**: Prioritize memories related to current task

## Hierarchical Memory Organization

### Levels of Memory

```
Level 1: Context Window
  Current conversation (recent messages)
  
Level 2: Conversation Summary
  Abstracted summaries of previous conversations
  
Level 3: User Profile
  Long-term user preferences, history
  
Level 4: Domain Knowledge
  General facts, system documentation
```

**Retrieval strategy**:
1. Check context window (fastest)
2. If not found, query conversation summaries
3. If not found, query user profile
4. If not found, query domain knowledge (slowest)

## Practical Memory Implementation

### Simple Approach

List of recent messages:
```python
memory = [
    {"role": "user", "content": "What's my balance?"},
    {"role": "assistant", "content": "Your balance is $5000"},
    {"role": "user", "content": "Show transaction history"}
]
```

Simple, works for short conversations.

### Vector DB Approach

Embed and store interactions:
```python
# Store
embedding = embed("User asked about balance")
vector_db.add(embedding, {"timestamp": now, "content": "..."})

# Retrieve
query_embedding = embed(user_query)
similar_memories = vector_db.search(query_embedding, top_k=5)
```

Better for long-term memory.

### Hybrid Approach

Combine multiple memory types:
- Recent context in context window
- Summaries in traditional DB
- Full interactions in vector DB
- Semantic knowledge in knowledge graph

## Memory-Augmented Generation (MAG)

### Process

```
User query
    ↓
Retrieve relevant memories
    ↓
Augment prompt with memories
    ↓
Generate response with context
```

Example:
```
User: "What was my total spending last year?"

Retrieved memories:
- Jan: $500
- Feb: $600
- ...
- Dec: $700

Augmented context: "User's monthly spending: [list]. 
Calculate annual total."

Response: Your total spending was $7,200
```

## Challenges

**Hallucination**: Model may invent memories
**Staleness**: Memories may become outdated
**Privacy**: Storing user data raises privacy concerns
**Consistency**: Conflicting memories from different times
**Retrieval failure**: Relevant memory not retrieved

## Best Practices

- **Separability**: Easy to add/remove memories without disruption
- **Searchability**: Efficient retrieval by content/time/relevance
- **Reliability**: Accurate storage, no corruption
- **Privacy**: Encrypt sensitive memories, audit access
- **Expiration**: Auto-delete old memories to manage cost

## Future Directions

- **Continual learning**: Update memory from new interactions
- **Forgetting**: Selectively forget irrelevant memories
- **Memory consolidation**: Merge similar memories
- **Multi-modal memory**: Store images, audio, documents
- **Distributed memory**: Shared memories across multiple agents

## Conclusion

Effective memory systems are essential for building intelligent agents. The best approach depends on use case: short conversations need only context windows; long-term agents need vector DBs and knowledge graphs. As agents become more sophisticated, memory management becomes increasingly important for maintaining coherence, privacy, and performance.
