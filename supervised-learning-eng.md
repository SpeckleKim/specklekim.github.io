---
layout: default
title: Supervised Learning
lang: en
---

# Supervised Learning: Building Predictive Models with Labeled Data

Supervised learning is the foundation of modern machine learning, where models learn from labeled datasets to make predictions on unseen data. It's called "supervised" because the training data comes with correct answers—the ground truth labels guide the learning process.

## What is Supervised Learning?

In supervised learning, we have a dataset of input-output pairs (X, y), where X represents features and y represents the target label or value. The model learns a mapping function f: X → y that generalizes well to new, unseen examples. This contrasts with [unsupervised learning](/unsupervised-learning-eng.html), where we only have inputs without explicit labels.

### Core Principle
The learning objective is to minimize the error between predictions and actual values. During training, the model adjusts its parameters to reduce this error, ideally capturing underlying patterns in the data.

## Classification vs Regression

The two primary tasks in supervised learning address different prediction goals:

**Classification** predicts discrete categories. Examples include:
- Email spam detection (spam vs. not spam)
- Image recognition (dog, cat, bird, etc.)
- Disease diagnosis (positive, negative)
- [Sentiment analysis](/sentiment-analysis-eng.html) (positive, negative, neutral)

Classification outputs are class labels, and success is measured by accuracy, precision, recall, or F1-score.

**Regression** predicts continuous numerical values. Examples include:
- House price prediction
- Temperature forecasting
- Stock price estimation
- Demand prediction for inventory management

Regression outputs are real numbers, and success is measured by Mean Squared Error (MSE), Mean Absolute Error (MAE), or R² score.

## Common Algorithms

### Linear Regression
Linear regression models the relationship between features and target as a linear equation: y = w₁x₁ + w₂x₂ + ... + wₙxₙ + b. It's interpretable, computationally efficient, and serves as a baseline. However, it assumes linear relationships, which often don't hold in real data.

**Variants:** Ridge regression adds [L2 regularization](/regularization-eng.html) to prevent overfitting. Lasso regression uses [L1 regularization](/regularization-eng.html), which can perform feature selection by shrinking some weights to zero.

### Logistic Regression
Despite its name, logistic regression is a classification algorithm. It applies the logistic sigmoid function to linear regression outputs, mapping values to probabilities between 0 and 1. It's widely used for binary classification and interpretable, making it popular in healthcare and finance.

### Decision Trees
[Decision trees](/decision-trees-eng.html) recursively split data into subsets based on feature values, creating a tree structure. Each internal node represents a decision, each branch represents an outcome, and each leaf represents a class prediction. Trees are interpretable and handle non-linear relationships well, but tend to overfit without pruning.

**Ensemble variants:** [Random Forests](/decision-trees-eng.html) combine multiple [decision trees](/decision-trees-eng.html), improving accuracy and reducing overfitting. [Gradient Boosting](/gradient-boosting-eng.html) ([XGBoost](/gradient-boosting-eng.html), [LightGBM](/gradient-boosting-eng.html)) sequentially builds trees, each correcting previous errors, achieving state-of-the-art performance on many datasets.

### Support Vector Machines (SVM)
SVMs find the optimal hyperplane that maximizes the margin between classes. Using kernel tricks, they can handle non-linear problems by implicitly mapping data to higher-dimensional spaces. SVMs work well with high-dimensional data and smaller datasets, though they scale poorly to very large datasets.

### K-Nearest Neighbors (KNN)
[KNN](/knn-eng.html) is a lazy learning algorithm that classifies new examples by voting from their k nearest training neighbors. It's simple, requires no training, but computationally expensive during prediction. Distance metrics like Euclidean distance determine "nearness."

## The Training Process

### Step 1: Data Preparation
Collect and clean data, handle missing values, remove duplicates, and check for data quality issues.

### Step 2: Feature Engineering
Select relevant features, create new features from existing ones, and normalize/standardize data. Good features are predictive, uncorrelated, and scale-appropriate.

### Step 3: Train-Test Split
Divide data into training, validation, and test sets (typically 70-80% training, 10-15% validation, 10-15% test). This prevents data leakage and enables honest performance evaluation.

### Step 4: Model Training
The algorithm learns from training data by optimizing parameters. For example, [gradient descent](/calculus-eng.html) iteratively updates weights to reduce loss.

### Step 5: Hyperparameter Tuning
Adjust hyperparameters ([learning rate](/learning-rate-scheduling-eng.html), tree depth, [regularization](/regularization-eng.html) strength) using validation data. Grid search or random search systematically explores hyperparameter combinations.

### Step 6: Evaluation and Testing
Assess final model performance on the held-out test set using appropriate metrics.

## Loss Functions

[Loss functions](/loss-functions-eng.html) quantify prediction error:

**Mean Squared Error (MSE):** Averages squared differences between predictions and ground truth, emphasizing large errors. Used for regression.

**Cross-Entropy Loss:** Measures divergence between predicted and actual [probability](/probability-eng.html) distributions. Standard for classification, especially with [neural networks](/neural-eng.html).

**Hinge Loss:** Used by SVMs, encourages correct classification with margin.

**Mean Absolute Error (MAE):** Averages absolute differences, more robust to outliers than MSE.

The choice of [loss function](/loss-functions-eng.html) affects model behavior—MSE penalizes outliers more heavily, while MAE treats all errors equally.

## Evaluation Metrics

**Accuracy** (Classification): Percentage of correct predictions. Simple but misleading with imbalanced datasets.

**Precision:** Of predicted positives, how many are actually positive? Important when false positives are costly.

**Recall:** Of actual positives, how many did we find? Critical when false negatives are costly (e.g., disease detection).

**F1-Score:** Harmonic mean of precision and recall, balancing both.

**Confusion Matrix:** Shows true positives, false positives, true negatives, false negatives—the foundation for other metrics.

**ROC-AUC:** Area under the receiver operating characteristic curve, measures classification performance across thresholds.

**Mean Squared Error (Regression):** Average squared prediction errors.

**R² Score (Regression):** Proportion of variance explained by the model (0 to 1, higher is better).

## Overfitting and Underfitting

**Overfitting** occurs when a model learns training data too well, including noise and quirks that don't generalize. The model has high training accuracy but poor test accuracy. Causes include:
- Model complexity too high relative to training data
- Insufficient training data
- No [regularization](/regularization-eng.html)

Prevention strategies: Use simpler models, collect more data, apply [regularization](/regularization-eng.html) (L1, L2, [dropout](/dropout-eng.html)), use early stopping, or increase training data diversity.

**Underfitting** occurs when the model is too simple to capture underlying patterns. Both training and test accuracy are poor. Causes include:
- Model complexity too low
- Insufficient training
- Wrong features

Solutions: Use more complex models, engineer better features, train longer, or collect more relevant data.

## The Balance

The goal is the sweet spot: a model complex enough to capture patterns but simple enough to generalize. This is achieved through proper train-validation-test splits, [cross-validation](/cross-validation-eng.html), and careful [hyperparameter tuning](/hyperparameter-tuning-eng.html).

Supervised learning powers countless applications today—from medical diagnosis to autonomous vehicles—making it essential knowledge for any AI practitioner.

