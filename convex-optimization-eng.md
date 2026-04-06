---
layout: default
title: Convex Optimization for AI
lang: en
---

# Convex Optimization for AI

Convex optimization is the mathematical backbone of machine learning. From [neural network](/neural-eng.html) training to [support vector machines](/svm-eng.html), optimization problems are central to AI. This guide explores the theory and practical applications of convex optimization.

## Convex Sets: Foundational Geometry

A set C is convex if for any two points x, y ∈ C, the entire line segment between them lies in C:

**λx + (1-λ)y ∈ C for all λ ∈ [0,1]**

**Examples of convex sets:**
- Hyperplanes: {x | aᵀx = b} (affine subsets)
- Half-spaces: {x | aᵀx ≤ b} (linear inequalities)
- Balls: {x | ||x - c|| ≤ r}
- Polyhedra: intersections of half-spaces
- Cone: {αx | α ≥ 0, x ∈ C}

**Non-convex sets:**
- Circles and ellipses (only the boundary)
- Union of disjoint regions

Convex geometry matters for optimization because convex sets have nice properties: any local minimum within a convex set is a global minimum. This eliminates the curse of multiple local minima.

## Convex Functions: Key Properties

A function f is convex over a convex set C if:

**f(λx + (1-λ)y) ≤ λf(x) + (1-λ)f(y) for all x,y ∈ C and λ ∈ [0,1]**

Intuitively, the function lies below any chord connecting two points on its graph.

**Examples of convex functions:**
- Linear functions: f(x) = aᵀx + b
- Quadratic functions: f(x) = xᵀQx + bᵀx + c (Q positive semidefinite)
- Norms: ||x||₂, ||x||₁ (L1 and L2 norms)
- Exponential: f(x) = eᵃˣ
- Log-sum-exp: f(x) = log(∑ eˣⁱ)
- Negative [entropy](/information-theory-eng.html): f(x) = ∑ xᵢ log(xᵢ)

**Operations preserving convexity:**
- Non-negative weighted sum: if f, g convex and α, β ≥ 0, then αf + βg is convex
- Composition: if g convex and h non-decreasing convex, h(g(x)) is convex
- Pointwise maximum: max(f, g) is convex if f, g are convex
- Partial minimization: if f convex in (x,y), then min_y f(x,y) is convex in x

These operations enable constructing complex convex functions from simple ones.

## The Fundamental Theorem: Global vs Local Optima

**Critical insight:** For convex optimization problems, any local minimum is a global minimum.

**Proof sketch:** Suppose x* is a local minimum but not global. Then ∃y with f(y) < f(x*). Consider the line segment between them: z(λ) = λy + (1-λ)x*. By convexity:

f(z(λ)) ≤ λf(y) + (1-λ)f(x*) < f(x*)

For small λ > 0, f(z(λ)) < f(x*), contradicting that x* is a local minimum.

This eliminates the primary difficulty in non-convex optimization. Moreover, if the objective is strictly convex, the minimum is unique.

**First-order necessary condition:** At a minimum x* of differentiable f:
- ∇f(x*) = 0

For convex f, this condition is also sufficient. This enables simple gradient-based algorithms: follow the negative gradient, and you reach the global optimum.

## Lagrange Multipliers: Handling Constraints

Real optimization problems often have constraints. Lagrange multipliers extend unconstrained optimization theory.

**Problem formulation:**
minimize f₀(x)
subject to: fᵢ(x) ≤ 0, i = 1,...,m
           hⱼ(x) = 0, j = 1,...,p

**Lagrangian:** L(x, λ, ν) = f₀(x) + ∑ λᵢfᵢ(x) + ∑ νⱼhⱼ(x)

Where λᵢ are inequality constraint multipliers (λᵢ ≥ 0) and νⱼ are equality constraint multipliers (unconstrained).

**Complementary slackness:** At optimality, if an inequality constraint is inactive (fᵢ(x*) < 0), its multiplier is zero: λᵢ = 0. If active (fᵢ(x*) = 0), the multiplier can be positive.

Intuition: only binding constraints affect the optimal solution. The multiplier λᵢ represents the marginal cost of relaxing constraint i.

## KKT Conditions: Optimality Characterization

For a convex problem with differentiable objectives and constraints, the Karush-Kuhn-Tucker (KKT) conditions are necessary and sufficient for optimality:

1. **Primal feasibility:**
   - fᵢ(x*) ≤ 0 for i = 1,...,m
   - hⱼ(x*) = 0 for j = 1,...,p

2. **Dual feasibility:**
   - λᵢ* ≥ 0 for i = 1,...,m

3. **Complementary slackness:**
   - λᵢ* fᵢ(x*) = 0 for i = 1,...,m

4. **Stationarity:**
   - ∇f₀(x*) + ∑ λᵢ* ∇fᵢ(x*) + ∑ νⱼ* ∇hⱼ(x*) = 0

These conditions can be checked to verify optimality without knowing the problem's structure.

**Interpretation:** At optimality, the gradient of the objective is a linear combination of constraint gradients. The weights (multipliers) represent importance of each constraint.

## Duality: Problem Reformulation

**Dual problem:** For every optimization problem (primal), we can formulate an associated dual.

**Dual function:** g(λ, ν) = inf_x L(x, λ, ν)

The dual problem is:
maximize g(λ, ν)
subject to: λ ≥ 0

**Weak duality:** g(λ, ν) ≤ f₀(x*) for any feasible (x, λ, ν).

This provides lower bounds on the optimal value without solving the primal.

**Strong duality:** For convex problems (under mild regularity conditions, such as Slater's condition), optimal primal and dual values are equal.

p* = d*

This enables solving the dual instead, sometimes more computationally efficient.

**Dual decomposition:** Duality enables parallel algorithms for large-scale problems. Decompose the problem, solve sub-problems independently using dual variables to coordinate.

## Applications in Support Vector Machines

[SVM](/svm-eng.html) is a canonical application of convex optimization.

**SVM primal problem** (linear case):
minimize (1/2)||w||² + C∑ξᵢ
subject to: yᵢ(wᵀφ(xᵢ) + b) ≥ 1 - ξᵢ
           ξᵢ ≥ 0

Here w is the decision boundary, φ is the feature map, ξᵢ are slack variables allowing misclassification, and C trades off margin width vs misclassification.

This is a convex quadratic program: minimize convex objective subject to linear constraints.

**SVM dual problem:**
maximize ∑ αᵢ - (1/2)∑∑ αᵢαⱼyᵢyⱼK(xᵢ,xⱼ)
subject to: ∑ αᵢyᵢ = 0
           0 ≤ αᵢ ≤ C

Where K(xᵢ,xⱼ) = φ(xᵢ)ᵀφ(xⱼ) is the kernel function.

The dual enables kernel methods: compute K directly without explicit feature maps, enabling non-linear classification in high-dimensional spaces efficiently.

**Complementary slackness** from KKT conditions:
- αᵢ > 0 only if yᵢ(wᵀφ(xᵢ) + b) = 1 (on the margin)
- αᵢ = C only if ξᵢ > 0 (misclassified)

Only a subset (support vectors) determine w, enabling efficient sparse solutions.

## Constrained Optimization in Machine Learning

**Regularized loss minimization:**
minimize ∑ loss(yᵢ, fθ(xᵢ)) + λR(θ)

Where R(θ) is a [regularization](/regularization-eng.html) term. This is equivalent to constrained optimization:
minimize ∑ loss(yᵢ, fθ(xᵢ))
subject to: ||θ||² ≤ t

The Lagrange multiplier λ controls the constraint tightness.

**Federated learning constraints:** Distributed learning problems often have privacy constraints enforced via Lagrangian duality.

**Portfolio optimization:** Maximize return subject to risk and leverage constraints—a canonical application of convex optimization in finance.

## Optimization Algorithms: From Theory to Practice

**Gradient descent:** Simple but effective. For convex f:
xₖ₊₁ = xₖ - αₖ∇f(xₖ)

With appropriate step size αₖ, converges to optimum. Slow for ill-conditioned problems.

**Newton's method:** Uses second-order information (Hessian):
xₖ₊₁ = xₖ - [∇²f(xₖ)]⁻¹∇f(xₖ)

Quadratic convergence: error is squared each iteration. Expensive Hessian computation limits applicability.

**Coordinate descent:** Optimize one variable at a time:
xₖ₊₁ = argmin_xⱼ f(x₁,...,xₖ,xⱼ,xₖ₊₁,...)

Useful for high-dimensional convex problems with separable structure.

**Proximal methods:** Handle non-smooth convex functions by using proximal operators.

**Projected gradient descent:** For constrained problems:
xₖ₊₁ = Π_C(xₖ - αₖ∇f(xₖ))

Where Π_C is projection onto the feasible set C.

## Approximating Non-Convex with Convex

Many problems of interest are non-convex. Convex relaxation provides approximate solutions.

**Convex relaxation:** Replace non-convex constraints with convex ones. For example, in compressed sensing, replace the non-convex "minimize ||x||₀ subject to Ax = b" with the convex L1 relaxation "minimize ||x||₁ subject to Ax = b".

Under suitable conditions, L1 minimization recovers sparse solutions achievable by L0 minimization.

**Semidefinite programming:** Relax quadratic constraints to positive semidefinite constraints, creating convex programs.

## Limitations and When Convex Optimization Fails

Convex optimization assumes:
1. Convex objective function
2. Convex feasible set

Many real problems violate these: deep [neural network](/neural-eng.html) training is highly non-convex. Local minima matter when the landscape is non-convex.

However, [neural networks](/neural-eng.html) often have favorable geometry: many local minima are near global optima. Moreover, modern techniques (normalization, [regularization](/regularization-eng.html)) implicitly enforce structure.

## Conclusion

Convex optimization provides a rigorous mathematical framework guaranteeing convergence to global optima under mild conditions. Understanding convex sets, functions, and duality informs algorithm design and problem formulation. From SVMs to [federated learning](/privacy-ai-eng.html) to large-scale ML systems, convex optimization principles underpin successful algorithms. While modern deep learning extends beyond convex problems, convex theory remains foundational for understanding optimization in AI systems.
