---
layout: default
title: Calculus & Optimization for AI
lang: en
---

# Calculus & Optimization for AI

Calculus is the mathematics of change. In machine learning, calculus enables us to understand how model outputs change with respect to parameters, which is essential for training neural networks and optimizing any machine learning model. Optimization—finding parameters that minimize error—is at the heart of learning.

## Derivatives and Rates of Change

A **derivative** measures how a function changes as its input changes. For function f(x), the derivative is denoted f'(x) or df/dx.

**Geometrically**, the derivative at a point is the slope of the tangent line to the function at that point. A steep positive slope means the function is increasing rapidly.

**Mathematically**: **f'(x) = lim_{h→0} [f(x+h) - f(x)] / h**

**Key derivatives:**
- Constant: d/dx[c] = 0
- Power rule: d/dx[x^n] = n·x^(n-1)
- Exponential: d/dx[e^x] = e^x
- Logarithm: d/dx[log x] = 1/x
- Chain rule: d/dx[f(g(x))] = f'(g(x))·g'(x)

Derivatives tell us the direction and magnitude of change, crucial for optimization.

## Partial Derivatives

In machine learning, we work with functions of multiple variables: f(x₁, x₂, ..., xₙ). A **partial derivative** measures how f changes when we vary one variable while keeping others constant.

**Notation**: ∂f/∂x_i is the partial derivative with respect to x_i.

**Example**: If f(w, b) = (y - (w·x + b))², then:
- ∂f/∂w = -2x(y - (w·x + b))
- ∂f/∂b = -2(y - (w·x + b))

In neural networks, we compute thousands of partial derivatives to understand how a single weight affects the final loss.

## The Chain Rule and Backpropagation

The **chain rule** enables us to compute derivatives of composite functions: **d/dx[f(g(x))] = df/dg · dg/dx**

Graphically, the chain rule says: "effect on output = effect through intermediate variable × effect on intermediate variable."

**Backpropagation** applies the chain rule systematically to compute gradients in neural networks:

1. Forward pass: compute predictions layer by layer
2. Backward pass: compute gradients flowing backward through layers
3. Each layer's gradient depends on: (loss derivative w.r.t. layer output) × (layer output derivative w.r.t. weights)

This propagation of gradients backward through the network is what enables efficient training.

## Gradient and Directional Derivatives

The **gradient** ∇f is a vector of all partial derivatives: **∇f = [∂f/∂x₁, ∂f/∂x₂, ..., ∂f/∂xₙ]**

**Key property**: The gradient points in the direction of steepest ascent. Its magnitude is the rate of increase in that direction.

The negative gradient -∇f points toward steepest descent—the direction that decreases the function fastest.

**Directional derivative** in direction u: **D_u f = ∇f · u**

This is the directional analog of the derivative, telling us how fast f decreases along direction u.

## Gradient Descent

**Gradient descent** is the foundational optimization algorithm: iteratively move in the direction of steepest descent.

**Update rule**: **θ_{t+1} = θ_t - α·∇L(θ_t)**

Where:
- θ are the parameters (weights and biases)
- L is the loss function (error to minimize)
- α is the learning rate (step size)
- ∇L(θ) is the gradient of loss with respect to parameters

**Learning rate matters:**
- Too small: slow convergence, lots of computation
- Too large: overshoots optima, diverges

**Batch gradient descent**: Compute gradient using all training data, then update once per epoch. Slow but stable.

**Stochastic gradient descent (SGD)**: Compute gradient using one data point, update immediately. Fast and noisy, adds implicit regularization.

**Mini-batch gradient descent**: Compute gradient using mini-batch of data (e.g., 32 samples), update once per batch. Balances speed and stability.

## Momentum and Acceleration

**Momentum** accumulates gradient information over time, accelerating convergence:

**v_{t+1} = β·v_t + (1-β)·∇L(θ_t)**
**θ_{t+1} = θ_t - α·v_t**

Where β (typically 0.9) controls how much previous velocity matters. Momentum:
- Accelerates along consistent descent directions
- Dampens oscillations in noisy directions
- Acts like a ball rolling downhill, building speed in the right direction

**Adam optimizer** combines momentum with adaptive learning rates per parameter, adjusting step sizes based on gradient magnitudes. Widely used in practice.

## Convexity and Local vs Global Minima

A function is **convex** if any line segment between two points on the function stays above the function. Convex functions have a single global minimum.

**Why convex matters**: Gradient descent converges to the global optimum for convex functions. Non-convex functions (like those with neural networks) can get stuck in local minima.

**Global minimum**: Lowest point across entire domain—the true optimum.

**Local minimum**: Lowest point in a neighborhood—gradient descent may get stuck here.

Neural networks are non-convex, so gradient descent finds local minima, not guaranteed global optima. Yet in practice, local minima often provide excellent solutions due to:
- High dimensionality effects
- Implicit regularization from SGD
- Careful initialization and hyperparameter tuning

## Saddle Points and Critical Points

A **critical point** is where ∇f = 0 (gradient is zero).

Critical points can be:
- **Minima**: local or global, where function value is low
- **Maxima**: where function value is high
- **Saddle points**: function increases in some directions, decreases in others

In high-dimensional spaces, saddle points are more common than local minima, and gradient descent naturally escapes them.

## Second-Order Optimization

**Hessian matrix** H contains all second partial derivatives: H_ij = ∂²f/(∂x_i∂x_j)

The Hessian reveals curvature information:
- Positive definite Hessian → local minimum (curved up)
- Negative definite Hessian → local maximum (curved down)
- Mixed signs → saddle point

**Newton's method** uses the Hessian for faster convergence: **θ_{t+1} = θ_t - [H^{-1}]·∇L(θ_t)**

More expensive (computing Hessian) but requires fewer iterations. Rarely used in deep learning due to computational cost.

## Applications in Neural Network Training

**Parameter updates**: Every weight and bias update in a neural network uses calculus to determine the direction and magnitude of change.

**Hyperparameter sensitivity**: Calculus helps us understand how changing learning rate, batch size, or architecture affects training dynamics.

**Regularization**: L1 and L2 regularization add terms to the loss whose gradients penalize large weights.

**Batch normalization**: Uses derivatives to understand how internal representations change during training.

**Layer-wise learning**: Calculus analysis reveals which layers train fastest and why deep networks can suffer from vanishing/exploding gradients.

Without calculus, we'd be stuck guessing how to adjust parameters. With it, we have a systematic way to improve any differentiable model toward better solutions. This is the secret to the success of deep learning.
