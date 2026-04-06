---
layout: default
title: "Vector Databases"
lang: en
---

# Vector Databases

## Introduction

Vector databases store and search high-dimensional vectors (embeddings) efficiently. They power semantic search, similarity matching, and retrieval-augmented generation (RAG) systems that are central to modern AI applications. Traditional databases index on exact matches; vector databases search by similarity.

## Why Vector Databases?

### The Similarity Search Problem

Given a query vector q and a corpus of millions of vectors, find the k most similar:

**Naive approach**: Compute distance from q to every vector (O(n) complexity). Slow for large datasets.

**Indexing approach**: Build structure enabling fast approximate nearest neighbor (ANN) search.

### Use Cases

- **Semantic search**: Find documents similar in meaning, not just keywords
- **Recommendation systems**: Recommend items based on embedding similarity
- **Duplicate detection**: Identify similar content across datasets
- **RAG systems**: Retrieve relevant context for LLM inference
- **Image search**: Find visually similar images by embedding

## Approximate Nearest Neighbor (ANN) Algorithms

### HNSW (Hierarchical Navigable Small World)

Creates hierarchical graph structure:
- Build multiple layers of proximity graphs
- Higher layers have fewer, more distant nodes (navigation)
- Lower layers densely connected (precision)
- Navigate from top layer to bottom, then search locally

**Advantages**: Fast search (logarithmic complexity), good quality

**Trade-offs**: Memory overhead for graph structure, slower indexing

### IVF (Inverted File Index)

Divides vector space into regions:
- Cluster vectors into k centroids (e.g., k-means)
- Assign each vector to nearest centroid (inverted index)
- Query: find nearest centroids, search only their vectors

**Advantages**: Memory efficient, parallelizable

**Trade-offs**: Quality depends on centroid selection, potential for dead zones

### LSH (Locality-Sensitive Hashing)

Hash similar vectors to same bucket:
- Use family of hash functions preserving locality
- Similar vectors hash to same bucket with high probability
- Query: hash query vector, search same buckets

**Advantages**: Theoretical guarantees, simple to implement

**Trade-offs**: Many hash functions needed for high recall

## Popular Vector Database Platforms

### Pinecone

Fully managed vector database:
- Easy API, zero maintenance
- Automatic indexing and scaling
- Integrated with LLM ecosystems (LangChain, LlamaIndex)
- Pricing: pay per storage and API calls

**Best for**: Teams wanting production-grade vector search without DevOps

### Weaviate

Open-source, flexible:
- Schema-flexible, GraphQL interface
- Built-in ML modules (text2vec, image2vec)
- Hybrid search (combine vector and keyword)
- Cloud and self-hosted options

**Best for**: Custom requirements, on-premise deployments

### Milvus

Open-source, high-performance:
- Written in C++, optimized for speed
- Supports multiple index types
- Distributed, cluster-ready
- Active open-source community

**Best for**: Large-scale deployments, cost-conscious teams

### Chroma

Lightweight, developer-friendly:
- In-process embedding storage
- No external dependencies
- Great for prototyping and local development
- Can scale to cloud with proper deployment

**Best for**: Rapid prototyping, small to medium projects

### Qdrant

Feature-rich open-source:
- Built in Rust, high performance
- Advanced filtering, payload storage
- Good recommendation system support
- Production-ready

**Best for**: Production systems with complex queries

### pgvector

PostgreSQL extension:
- Brings vector search to PostgreSQL
- SQL interface, leverage existing databases
- Approximate and exact search
- Limited but improving functionality

**Best for**: Teams with PostgreSQL infrastructure

## Indexing Strategies

### Dense Index

Keeps all vectors in fast-access memory:
- Lowest latency
- Highest memory cost
- Best for small to medium datasets (<1B vectors)

### Sparse Index

Partitions data, loads on demand:
- Lower memory per partition
- Slightly higher latency
- Better for large datasets

### Hierarchical Index

Multi-level structure optimized for different scales:
- HNSW uses this naturally
- Best of both worlds: speed and memory efficiency

## Distance Metrics

### Euclidean Distance

L2 norm: sqrt(sum((a_i - b_i)^2))
- Intuitive, commonly used
- Appropriate for continuous features

### Cosine Similarity

(a · b) / (||a|| ||b||)
- Ranges from -1 to 1 (1 = identical direction)
- Standard for text embeddings (normalized)
- Scale-invariant

### Inner Product

a · b
- Fast computation (single dot product)
- Requires normalized vectors for similarity interpretation
- Common in deep learning

## Integration with RAG

Vector databases are core to RAG:

```
1. Embed documents: Convert docs to vectors, store in DB
2. Query processing: Embed user query
3. Retrieval: Find k most similar document vectors
4. Context assembly: Retrieve full text of similar documents
5. LLM inference: Pass documents as context to LLM
```

Quality of retrieval dramatically affects final answer quality. Embedding model choice and database design critical.

## Performance Considerations

**Throughput**: Queries per second
- Depends on index type, data distribution
- HNSW typically 100-1000 QPS single-core

**Latency**: Time per query
- Milliseconds for well-optimized single queries
- Batch queries amortize overhead

**Memory**: Storage per vector
- Compressed indices: 1-10GB per million vectors
- Uncompressed: 4-8GB per million 768-dim vectors

**Recall**: Percentage of true nearest neighbors found
- Trade-off between recall and latency
- High recall (>90%) typically requires 10x larger search radius

## Best Practices

- **Embedding model choice**: Fundamental; domain-specific models often outperform general
- **Dimensionality reduction**: PCA or quantization can reduce memory with minimal quality loss
- **Batch operations**: More efficient than individual queries
- **Monitoring**: Track query latency, recall, index freshness
- **Regular reindexing**: As data changes, index quality may degrade
- **Distance metric matching**: Choose metric appropriate for embedding model

## Future Directions

- **Multimodal vectors**: Single embedding space for text, images, audio
- **Learned indices**: ML-based indices replacing hand-tuned structures
- **Quantum approaches**: Quantum computing for certain similarity searches
- **Hybrid dense-sparse**: Combining dense vectors with sparse features for flexibility

## Conclusion

Vector databases enable semantic understanding in AI systems. Choosing the right database depends on scale, latency requirements, and infrastructure constraints. As RAG becomes standard in LLM applications, understanding vector database design becomes increasingly important for AI engineers.
