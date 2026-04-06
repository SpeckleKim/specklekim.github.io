---
layout: default
title: Hyperparameter Tuning
lang: en
---

# Hyperparameter Tuning

Hyperparameter tuning is the process of finding the optimal configuration for model training. Unlike model parameters (weights, biases) that are learned from data, hyperparameters control the learning process itself and must be set before training.

## Hyperparameters vs Parameters

**Parameters** are learned from data during training:
- Neural network weights and biases
- Decision tree split values
- Linear regression coefficients

**Hyperparameters** are set before training and control how learning happens:
- Learning rate, batch size, number of epochs
- Regularization strength (λ)
- Model architecture (depth, width)
- Dropout rate, early stopping patience
- Tree depth, number of trees in ensemble

Hyperparameters are the ones you tune; parameters are automatically optimized.

## Grid Search

Grid search exhaustively evaluates all combinations of hyperparameters from discrete sets.

**Algorithm:**
1. Define a grid of hyperparameter values
2. Train a model for each combination
3. Evaluate using cross-validation
4. Select combination with best CV score

**Example:**
```
learning_rate ∈ {0.001, 0.01, 0.1}
batch_size ∈ {16, 32, 64}
dropout ∈ {0.2, 0.5}

Total combinations: 3 × 3 × 2 = 18 models
```

**Advantages:**
- Exhaustive: guaranteed to find best combination
- Simple to implement and understand
- Embarrassingly parallel (evaluate combinations independently)
- Works well with small search spaces

**Disadvantages:**
- Exponential growth with number of hyperparameters
- Wasted computation on bad regions
- Assumes discrete values (not continuous ranges)
- Example: 5 hyperparameters × 10 values each = 100,000 models

Grid search becomes impractical with >4 hyperparameters.

## Random Search

Random search samples hyperparameter combinations randomly from the search space.

**Algorithm:**
1. Define ranges for each hyperparameter
2. Sample random combinations (often more efficiently distributed)
3. Train a model for each sample
4. Evaluate using cross-validation
5. Select sample with best CV score

**Number of iterations:** Much fewer than grid search for same hyperparameter count

**Example:**
```
learning_rate ~ Uniform(0.0001, 0.1)
batch_size ~ Discrete Uniform({16, 32, 64, 128})
dropout ~ Uniform(0.1, 0.7)

Sample 50 random combinations instead of 1000+
```

**Advantages:**
- More efficient for high dimensions
- Not biased toward grid points
- Can handle continuous hyperparameter ranges
- Often finds competitive solutions with 10-20% of grid search cost

**Disadvantages:**
- Not guaranteed to find absolute best
- Less systematic than grid search
- Still wasteful in poor regions

**Best practice:** Random search often outperforms grid search on high-dimensional problems.

## Bayesian Optimization

Bayesian optimization uses a probabilistic model to intelligently sample the hyperparameter space.

**Core idea:** Build a surrogate model (usually a Gaussian Process) that predicts model performance from hyperparameters, then use this model to decide which hyperparameters to try next.

**Algorithm:**
1. Sample initial random hyperparameters (typically 5-10)
2. Train models and record performance
3. Fit a probabilistic surrogate model
4. Use acquisition function (e.g., Expected Improvement) to select next promising hyperparameters
5. Train model with selected hyperparameters
6. Update surrogate model with new observation
7. Repeat steps 4-6

**Acquisition functions:**
- **Expected Improvement (EI):** Balance exploitation (try near best) and exploration (try uncertain regions)
- **Upper Confidence Bound (UCB):** Optimistic estimate favoring uncertain areas
- **Probability of Improvement (PoI):** Greedy approach favoring likely improvements

**Advantages:**
- Much more efficient than grid/random search
- Leverages past evaluations to inform future choices
- Works with expensive objectives (e.g., training takes hours)
- Handles mixed parameter types (continuous, discrete, categorical)
- Often finds better solutions than random search

**Disadvantages:**
- More complex to implement
- Surrogate model fitting adds overhead
- Requires observations to build model
- Less parallelizable than grid/random search

**Common tools:** Gaussian Processes (GP), Tree-structured Parzen Estimator (TPE)

## Hyperband and BOHB

Hyperband accelerates hyperparameter search using early stopping and resource allocation.

**Hyperband Algorithm:**
1. Allocate equal resources to many random hyperparameter configurations
2. Evaluate all configurations with minimum resource
3. Discard worst-performing half
4. Allocate more resources to survivors
5. Repeat steps 3-4 until single configuration remains

**Resource:** Training epochs, data size, or any measurable quantity

**Example with 81 configurations:**
- Round 1: Train 81 configs for 1 epoch each
- Round 2: Keep top 27, train for 3 epochs each
- Round 3: Keep top 9, train for 9 epochs each
- Round 4: Keep top 3, train for 27 epochs each
- Round 5: Keep top 1, train for 81 epochs

**BOHB (Bayesian Optimization + Hyperband):**
- Combines Bayesian optimization with Hyperband
- Uses TPE to select which configurations to promote
- Very efficient for expensive hyperparameter searches

**Advantages:**
- Much faster than Bayesian optimization alone
- Intelligent allocation of compute
- Can run many evaluations in parallel
- Minimal manual tuning required

## Optuna

Optuna is a modern hyperparameter optimization framework emphasizing ease of use.

**Features:**
- Sophisticated Bayesian optimization (TPE, CMA-ES)
- Define search space in Python naturally
- Automatic pruning (early stopping unpromising trials)
- Parallel and distributed optimization
- Visualization and analysis tools
- Storage of trial history

**Example:**
```python
import optuna

def objective(trial):
    lr = trial.suggest_float('lr', 1e-5, 1e-1, log=True)
    batch_size = trial.suggest_int('batch_size', 16, 128)
    dropout = trial.suggest_float('dropout', 0.1, 0.7)

    # Train model and return validation score
    score = train_and_evaluate(lr, batch_size, dropout)
    return score

study = optuna.create_study(direction='maximize')
study.optimize(objective, n_trials=100)
```

**Advantages:**
- Intuitive Python API
- Automatic pruning saves computation
- Excellent for iterative development
- Built-in visualizations
- Easy to parallelize

## Ray Tune

Ray Tune is a scalable hyperparameter tuning framework designed for distributed computing.

**Features:**
- Distributed training across clusters
- Integration with deep learning frameworks
- Multiple tuning algorithms (Bayesian, PBT, Population Based Training)
- Checkpointing and fault tolerance
- Population Based Training (PBT): evolve hyperparameters during training

**Best for:**
- Large-scale distributed experiments
- Very expensive training runs
- Complex experiment workflows
- Teams using distributed clusters

## Practical Tuning Strategies

### 1. Start Simple
- Tune 1-2 most important hyperparameters first
- Use coarse grid search (few values per parameter)
- Identify reasonable ranges

### 2. Coarse to Fine
- First pass: Random search with wide ranges
- Identify promising region
- Second pass: Finer grid around best region

### 3. Importance Ranking
- Prioritize hyperparameters by sensitivity
- Learning rate often most important
- Batch size, regularization strength next
- Model architecture details last

### 4. Cross-Validation is Essential
- Never tune on test set (causes overfitting in hyperparameter space)
- Use nested CV for honest evaluation
- Reserve test set for final assessment only

### 5. Parallel Search
- Evaluate multiple configurations simultaneously
- Grid and random search parallelize perfectly
- Bayesian optimization parallelizes partially
- Use all available compute resources

### 6. Early Stopping
- Monitor validation performance during training
- Stop unpromising configurations early
- Hyperband and Optuna include automatic pruning
- Saves massive computation on bad configurations

## Typical Hyperparameter Ranges

**Neural Networks:**
- Learning rate: 1e-5 to 1e-1 (log scale)
- Batch size: 16 to 256
- Dropout: 0.2 to 0.5
- L2 regularization: 1e-6 to 1e-3

**Decision Trees:**
- Max depth: 3 to 20
- Min samples per leaf: 1 to 20
- Criterion: gini or entropy

**Gradient Boosting:**
- Learning rate: 0.001 to 0.3
- Number of estimators: 100 to 1000
- Max depth: 3 to 10
- Subsample: 0.5 to 1.0

## Checklist for Hyperparameter Tuning

1. Define clear objective metric (accuracy, AUC, MSE)
2. Split data into train/validation/test before tuning
3. Choose search strategy (random > grid for high dimensions)
4. Set reasonable ranges based on domain knowledge
5. Use cross-validation for honest evaluation
6. Parallelize when possible
7. Monitor progress and convergence
8. Document best hyperparameters
9. Report test set performance (never use for tuning)
10. Consider computational cost vs improvement

Hyperparameter tuning is essential but should not consume all computational budget. Allocate resources to data quality, feature engineering, and tuning appropriately.
