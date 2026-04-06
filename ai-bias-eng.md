---
layout: default
title: "AI Bias & Fairness"
lang: en
---

# AI Bias & Fairness

## Introduction

AI bias occurs when models make systematically unfair decisions for certain groups. Bias emerges from training data, model architecture, and deployment context. Fairness requires understanding different notions of fairness, measuring bias, and designing mitigations. Addressing bias is essential for responsible, trustworthy AI.

## Types of Bias

### Selection Bias

Non-random data collection:
- Hiring datasets: overrepresent high-performing demographics
- Medical datasets: exclude patients with rare conditions
- Result: Model learns skewed population distribution

### Confirmation Bias

Human bias in data labeling:
- Annotators label examples differently based on expectations
- Leads to inconsistent labels
- Model learns biased patterns

### Algorithmic Bias

Inherent to model design:
- Feature interactions amplify existing bias
- Optimization objective misses fairness
- Feedback loops: biased predictions → biased training data

### Measurement Bias

Imperfect proxy for true outcome:
- Recidivism (criminal prediction): measures arrests, not actual crime
- Academic success: measures test scores, not true competence
- Model optimizes for imperfect proxy

## Fairness Metrics

### Demographic Parity

Equal prediction rates across groups:
- P(prediction=positive|group=A) = P(prediction=positive|group=B)
- Problem: Ignores differences in actual positive rate

### Equalized Odds

Equal error rates across groups:
- P(positive prediction|positive outcome, group=A) = ... group=B
- Requires equal false positive and false negative rates
- More challenging than demographic parity

### Calibration

Predictions accurate within groups:
- If model says "70% likely positive" for group A, ~70% are actually positive
- Doesn't require equal accuracy across groups
- Different notion of fairness

## Debiasing Techniques

### Pre-Processing

Clean training data:
- Balance datasets across groups
- Correct biased labels
- Augment underrepresented groups

### In-Processing

Modify training objective:
- Add fairness constraint to loss function
- Adversarial debiasing: add fairness penalty
- Trade-off: Usually reduces overall accuracy slightly

### Post-Processing

Adjust predictions:
- Calibrate predictions per group
- Threshold adjustment per group
- Doesn't change learned model

## Dataset Auditing

### Identify Disparities

Check for representation issues:
- Group distributions
- Label balance across groups
- Feature correlations with protected attributes

### Documenting Limitations

Model cards and datasheets:
- What demographic groups does dataset represent?
- What are known limitations?
- What fairness issues might arise?

## Regulatory Requirements

### EU AI Act

Risk classification for AI systems:
- High-risk: Requires fairness assessment
- Must document and mitigate bias
- Subject to audit and monitoring

### Executive Orders

US executive orders on AI governance:
- Agencies must conduct fairness impact assessments
- Bias testing required before deployment
- Transparency in AI decision-making

## Challenges

**Competing definitions**: Different stakeholders prefer different fairness metrics

**Measurement**: Some harms impossible to measure

**Accuracy-fairness trade-off**: Sometimes can't have both

**Context dependence**: Fairness depends on application

## Conclusion

Addressing AI bias requires technical solutions and thoughtful consideration of fairness. Best practice: involve affected communities, measure multiple fairness metrics, and be transparent about limitations.
