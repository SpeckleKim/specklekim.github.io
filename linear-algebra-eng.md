---
layout: default
title: Linear Algebra for AI
lang: en
---

# Linear Algebra for AI

Linear algebra is the foundation of machine learning and [artificial intelligence](/ai-eng.html). Nearly every ML algorithm, from [neural networks](/neural-eng.html) to recommendation systems, relies on linear algebra operations. Understanding vectors, matrices, and their transformations is essential for building, training, and optimizing AI models.

## Vectors and Matrices

A **vector** is an ordered list of numbers arranged in a column (or row). In AI, vectors represent data points, features, or embeddings. For example, an image can be represented as a vector of pixel intensities.

A **matrix** is a rectangular grid of numbers with rows and columns. Matrices represent transformations, datasets, or weight parameters in [neural networks](/neural-eng.html). An m×n matrix has m rows and n columns.

```
Vector v = [3, 1, 4]
Matrix A = [[1, 2],
            [3, 4],
            [5, 6]]
```

## Matrix Operations

**Matrix Addition and Subtraction**: Element-wise operations between matrices of the same dimensions.

**Matrix Multiplication**: The dot product of rows and columns. If A is m×n and B is n×p, then AB is m×p. This is fundamental in [neural networks](/neural-eng.html) where we multiply input vectors by weight matrices.

**Transpose**: Flipping a matrix across its diagonal. Written as A^T, swapping rows and columns. Essential in many ML formulas and optimization algorithms.

**Determinant**: A scalar value representing whether a matrix is invertible. Zero determinant means the matrix is singular (non-invertible).

**Matrix Inverse**: A^(-1) is the inverse of A if A·A^(-1) = I (identity matrix). Used in solving linear systems and optimization.

## Eigenvalues and Eigenvectors

An **eigenvector** of a matrix A is a non-zero vector v such that A·v = λ·v, where λ is the **eigenvalue**. Geometrically, applying A to an eigenvector only scales it—it doesn't change direction.

Eigenvalues reveal important properties:
- They indicate the magnitude of transformation along eigenvector directions
- All eigenvalues are positive for positive semi-definite matrices (common in ML)
- The largest eigenvalues capture the most important variance in data

## Singular Value Decomposition (SVD)

SVD decomposes any matrix A into: **A = U·Σ·V^T**

Where:
- **U**: orthogonal matrix of left singular vectors
- **Σ**: diagonal matrix of singular values
- **V^T**: transpose of orthogonal matrix of right singular vectors

**Why SVD matters in AI:**
- Reduces dimensionality while preserving maximum variance
- Powers image compression and noise reduction
- Fundamental to Principal Component Analysis (PCA)
- Used in matrix factorization for recommender systems

## Matrix Factorization

Matrix factorization approximates a large matrix as a product of smaller matrices. In recommender systems, a user-item rating matrix R is approximated as **R ≈ U·V^T**, where U contains user factors and V contains item factors.

This enables:
- **Dimensionality reduction**: finding latent patterns
- **Computational efficiency**: working with smaller matrices
- **Collaboration filtering**: predicting missing ratings based on patterns

## Principal Component Analysis (PCA)

PCA uses eigenvalues and eigenvectors to reduce data dimensionality while preserving variance:

1. Center data (subtract mean)
2. Compute covariance matrix
3. Find eigenvalues and eigenvectors of covariance matrix
4. Select top k eigenvectors with largest eigenvalues
5. Project data onto these k dimensions

**Applications:**
- Data visualization (reducing 784-dimensional MNIST images to 2D)
- Preprocessing for faster training
- Removing correlated features

## Embeddings and Transformations

In modern AI, **embeddings** represent words, items, or concepts as vectors in high-dimensional space. Embeddings capture semantic relationships: similar items have similar vectors.

Linear transformations using matrices enable:
- **Rotation**: changing coordinate systems
- **Scaling**: adjusting feature magnitudes
- **Projection**: mapping to lower dimensions
- **Composition**: chaining multiple transformations

[Neural networks](/neural-eng.html) learn weight matrices that transform embeddings at each layer, ultimately learning task-relevant representations.

## Linear Algebra in Neural Networks

**Forward pass**: Computing layer outputs involves matrix multiplication: y = W·x + b, where W is the weight matrix, x is input, and b is bias.

**Backpropagation**: Computing gradients uses the chain rule and matrix operations. The Jacobian matrix (matrix of partial derivatives) is central to understanding how gradients flow backward.

**Optimization**: [Gradient descent](/calculus-eng.html) updates are linear algebra operations: W ← W - α·∇W, where ∇W is the gradient matrix and α is [learning rate](/learning-rate-scheduling-eng.html).

## Why Master Linear Algebra?

- **Algorithm efficiency**: knowing which matrix operations are computationally efficient
- **Deep understanding**: grasping what models really do internally
- **Implementation**: building efficient custom layers or operations
- **Debugging**: understanding why models behave certain ways
- **Research**: developing novel architectures and algorithms

Linear algebra transforms abstract data into geometric objects we can manipulate, analyze, and optimize. It's the language in which machine learning thinks.
