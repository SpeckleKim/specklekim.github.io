---
layout: default
title: Backpropagation
lang: en
---

# Backpropagation: Training Neural Networks

Backpropagation is the fundamental algorithm that enables training of deep neural networks. It efficiently computes gradients of a loss function with respect to all network parameters, allowing us to update weights in the direction that minimizes loss.

## The Forward Pass

During the forward pass, data flows through the network layer by layer. Each neuron computes a weighted sum of its inputs plus a bias term, then applies an activation function. This process is repeated until we reach the output layer, producing predictions.

For a simple layer: `z = Wx + b`, then `a = activation(z)`.

The forward pass establishes the **computational graph**, which records how each variable depends on the inputs and parameters. This graph is essential for the backward pass.

## Computing the Loss

After predictions are made, we compute the loss—a scalar value measuring how far predictions deviate from true labels. Common loss functions include Mean Squared Error (MSE) for regression and cross-entropy for classification.

`L = (1/m) * Σ(y_true - y_pred)²` (MSE example)

The loss quantifies our model's error. Lower loss means better predictions.

## The Chain Rule Foundation

Backpropagation relies on the chain rule from calculus. If loss L depends on parameter W through multiple intermediate variables, the chain rule tells us:

`dL/dW = (dL/da) * (da/dz) * (dz/dW)`

This multiplicative relationship means we can compute gradients by multiplying local derivatives along the computational graph.

## The Backward Pass

The backward pass starts at the output layer and propagates error signals backward through the network. We compute the gradient of the loss with respect to each activation and parameter, working from the output toward the input.

Key insight: each layer only needs to know the gradient coming from the layer above (upstream gradient) and can then compute gradients with respect to its own parameters and inputs.

## Gradient Computation Through Layers

For a layer computing `a = σ(Wx + b)`:

1. Receive upstream gradient: `δ^(l+1) = dL/da^(l+1)`
2. Compute local gradient: `δ^(l) = δ^(l+1) * σ'(z^(l))`
3. Compute parameter gradients:
   - `dL/dW = δ^(l) * x^(l)T`
   - `dL/db = δ^(l)`
4. Compute input gradient: `dL/dx^(l) = W^T * δ^(l)` (passes to previous layer)

This modular approach lets us implement each layer independently.

## Computational Graph

The computational graph is a directed acyclic graph (DAG) where:
- **Nodes** represent variables (inputs, parameters, activations, loss)
- **Edges** represent operations that compute child nodes from parent nodes

Backpropagation traverses this graph in reverse topological order, computing gradients efficiently. The graph is usually built dynamically during the forward pass.

## Automatic Differentiation

Modern deep learning frameworks (PyTorch, TensorFlow) implement automatic differentiation engines that automatically build the computational graph and compute gradients—no manual calculus required.

Two approaches:
- **Reverse-mode AD** (used for backprop): efficient for scalar outputs, O(1) graph traversals
- **Forward-mode AD**: efficient for many outputs relative to few inputs

For neural networks with millions of parameters and scalar losses, reverse-mode is ideal.

## Challenges and Solutions

**Vanishing Gradients**: In deep networks, gradients can become extremely small as they backpropagate through many layers, making learning slow.

Solutions: ReLU activations, layer normalization, residual connections, careful weight initialization.

**Exploding Gradients**: Conversely, gradients can blow up exponentially.

Solutions: Gradient clipping, normalization techniques.

## Practical Considerations

Weight updates occur after computing gradients for a batch:

`W_new = W_old - learning_rate * dL/dW`

The learning rate controls step size. Larger rates converge faster but risk overshooting; smaller rates are stable but slower.

Backpropagation enables end-to-end training where high-level task performance directly drives learning at all levels. This is why deep learning has been so successful.

## Summary

Backpropagation efficiently computes gradients through the chain rule by:
1. Making a forward pass to build the computational graph and compute loss
2. Backpropagating loss gradients through each layer
3. Using local gradient information to update parameters

This elegant algorithm makes training arbitrarily deep networks practical, forming the foundation of modern deep learning.
