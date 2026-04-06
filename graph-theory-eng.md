---
layout: default
title: Graph Theory for AI
lang: en
---

# Graph Theory for AI

Graph theory is fundamental to modern AI systems. From social networks to knowledge graphs to [neural network](/neural-eng.html) architectures, graphs represent relationships and dependencies. This guide explores graph concepts essential for AI practitioners and researchers.

## Graph Basics: Nodes and Edges

A graph G = (V, E) consists of vertices (nodes) V and edges E connecting them. Graphs model any system with entities (nodes) and relationships (edges).

**Directed vs Undirected Graphs:**
- Undirected graphs have symmetric edges (friendship networks)
- Directed graphs have asymmetric edges with direction (follower relationships, knowledge base implications)

**Weighted graphs** assign values to edges, representing strength, cost, or [probability](/probability-eng.html) of relationships. A knowledge graph might assign weights representing confidence in assertions.

**Graph properties** characterize structure:
- **Degree**: number of edges connected to a node (undirected) or in-degree/out-degree (directed)
- **Density**: ratio of actual to possible edges (sparse vs dense graphs)
- **Diameter**: longest shortest path between any two nodes, measuring graph "width"
- **Clustering coefficient**: tendency of nodes to form triangles (cliques)

In social networks, high [clustering](/clustering-eng.html) coefficient indicates tight friend groups. In biological networks, it reveals functional modules.

## Adjacency Matrices: Compact Representations

An adjacency matrix A represents graph structure as an n×n matrix where A[i,j] indicates connection from node i to j.

For unweighted undirected graphs:
- A[i,j] = 1 if edge exists, 0 otherwise
- A is symmetric: A[i,j] = A[j,i]

For weighted graphs:
- A[i,j] = weight of edge from i to j
- Zero indicates no edge

For directed graphs:
- A need not be symmetric

**Computational advantages:**
- Matrix multiplication A² gives paths of length 2: (A²)[i,j] counts 2-hop paths
- (Aᵏ)[i,j] counts k-step paths between i and j
- Eigenvalues of A reveal structural properties

**Sparsity consideration:** Real networks are typically sparse (few edges relative to possible edges). Sparse matrix representations use only O(|E|) space rather than O(|V|²), critical for large-scale networks.

## Graph Traversal: Exploring Connections

Traversal algorithms systematically visit nodes. Two fundamental approaches:

**Breadth-First Search (BFS):**
- Explores nodes level by level from a starting node
- Uses a queue data structure
- Finds shortest paths in unweighted graphs
- Discovers connected components
- Time complexity: O(|V| + |E|)

**Depth-First Search (DFS):**
- Explores deeply before backtracking
- Uses a stack (or recursion)
- Detects cycles
- Identifies strongly connected components in directed graphs
- Also O(|V| + |E|)

In recommendation systems, BFS from a user finds "nearby" other users (within 2-3 connections). In [neural networks](/neural-eng.html), DFS-like computation propagates information through layers.

## Shortest Paths: Optimal Routes

**Dijkstra's Algorithm** finds shortest paths from a source to all other nodes in weighted graphs with non-negative weights.

Mechanism: maintain distance estimates to each node, iteratively improve them by exploring edges. Uses a priority queue, achieving O((|V| + |E|) log |V|) complexity.

**Floyd-Warshall Algorithm** computes shortest paths between all pairs of nodes:
- Time: O(|V|³)
- Space: O(|V|²)
- Handles negative weights (but not negative cycles)

**Bellman-Ford Algorithm** handles negative weights:
- Time: O(|V| * |E|)
- Detects negative cycles
- Less efficient than Dijkstra but more general

**Applications in AI:**
- Route planning in navigation systems
- Inference paths in knowledge graphs
- Cost optimization in decision networks

## Advanced Graph Properties

**Connectivity:** A graph is connected if a path exists between every pair of nodes. Directed graphs have stronger notions:
- Strongly connected: directed path exists both ways between all pairs
- Weakly connected: underlying undirected graph is connected

**Bipartite graphs** have nodes partitioned into two sets with edges only between sets (not within). Applications include:
- Recommendation systems (users connected to items)
- Collaborative filtering networks
- Matching problems

**Planarity:** Can a graph be drawn without edge crossings? Planar graphs have special properties enabling efficient algorithms.

## Knowledge Graphs: Structured Knowledge

Knowledge graphs represent facts as triples: (subject, predicate, object). Graphs connect concepts with labeled relationships.

Example triple: (Albert Einstein, wrote, E=mc²)

**Graph structure enables:**
- Semantic reasoning: traverse relationships to infer new facts
- Link prediction: estimate missing facts using neighboring relationships
- Knowledge reasoning with graph [neural networks](/neural-eng.html)

**Heterogeneous graphs** have multiple node and edge types, representing richer information. A knowledge graph might have node types {Person, Paper, Venue} and edge types {authorOf, citedBy, publishedAt}.

## Social Networks: Relationship Mining

Graphs naturally model social connections. Graph algorithms reveal network structure:

**Community detection** finds groups of densely connected nodes:
- Louvain algorithm optimizes modularity
- Identifies natural communities without predefined count
- Applications: friend suggestions, market segmentation

**Centrality measures** identify important nodes:
- **Degree centrality**: highly connected nodes (hubs)
- **Betweenness centrality**: nodes on many shortest paths (bridges)
- **Closeness centrality**: nodes near all others
- **Eigenvector centrality**: connected to other important nodes (PageRank for web graphs)

In social media, high betweenness nodes are influencers bridging communities. High eigenvector centrality indicates important figures.

## Graph Neural Networks: Deep Learning on Graphs

GNNs extend [neural networks](/neural-eng.html) to graph-structured data. Node features and graph structure jointly determine representations.

**Message Passing Framework:** Each node aggregates information from neighbors:

1. Message computation: each neighbor computes a message
2. Aggregation: messages are combined (sum, mean, max, etc.)
3. Update: node representation updates using aggregated information

**Common GNN architectures:**

**Graph Convolutional Networks (GCN):**
- Aggregate neighbor features with normalized adjacency matrix
- h^(l+1) = σ(D^(-1/2) A D^(-1/2) h^l W)
- D is degree matrix, normalization prevents [vanishing gradients](/vanishing-gradients-eng.html)

**GraphSAGE:**
- Sample neighbors to handle large graphs
- Learns inductive functions (generalize to unseen nodes)
- Critical for scalability

**Graph Attention Networks (GAT):**
- Learn attention weights over neighbors
- Different neighbors contribute differently
- Interpretable: attention weights reveal important relationships

**Applications:**
- Node classification (predict node labels)
- Link prediction (infer missing edges)
- Graph classification (predict properties of entire graphs)
- Recommendation systems (represent user-item interactions)
- Molecular property prediction (atoms as nodes)

## Spectral Methods: Algebraic Properties

**Graph Laplacian** L = D - A (D is degree matrix, A is adjacency) has powerful properties:

- Eigenvalues and eigenvectors reveal structural properties
- Number of zero eigenvalues equals number of connected components
- Second-smallest eigenvalue (algebraic connectivity) measures robustness
- Eigenvectors correspond to meaningful node partitions

**Spectral clustering** uses Laplacian eigenvectors:
1. Compute eigenvectors of Laplacian
2. Treat eigenvectors as node representations
3. Cluster using traditional methods (k-means)
4. Finds balanced node partitions

Spectral methods provide theoretical foundation for GNNs, connecting graph structure to learning objectives.

## Practical Considerations for Scale

**Storage:**
- Dense adjacency matrices: O(|V|²) - prohibitive for large graphs
- Sparse representations: adjacency lists or sparse matrices store only edges

**Computation:**
- Graph algorithms scale with edges, not just nodes
- Distributed computing necessary for billion-node graphs
- Sampling and approximation balance accuracy with feasibility

**Libraries:**
- NetworkX (Python): educational and small-scale graphs
- PyTorch Geometric: GNNs with [GPU](/gpu-hardware-eng.html) acceleration
- DGL (Deep Graph Library): unified GNN implementations
- GraphQL for knowledge graph query

## Real-World Applications

1. **Recommendation Systems:** User-item bipartite graphs with link prediction
2. **Knowledge Bases:** DBpedia, Wikidata using semantic relationships
3. **Protein Networks:** Interaction graphs for drug discovery
4. **Transportation:** City graphs for routing and congestion prediction
5. **Biology:** Metabolic networks, gene interaction networks
6. **NLP:** Parse trees, dependency graphs, language model attention patterns

## Conclusion

Graph theory provides elegant mathematical tools for representing relational data. From basic properties to advanced algorithms, graphs enable reasoning about complex systems. Graph [neural networks](/neural-eng.html) extend deep learning to structured data, while classical algorithms remain indispensable for analysis and optimization. Understanding graphs is essential for modern AI systems handling relational, hierarchical, and network-structured data.
