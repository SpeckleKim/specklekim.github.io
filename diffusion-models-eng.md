---
layout: default
title: Diffusion Models
lang: en
---

# Diffusion Models

## Introduction

Diffusion Models represent a paradigm shift in generative modeling, achieving state-of-the-art [image generation](/image-generation-eng.html) quality through a process inspired by diffusion in statistical physics. Unlike [GANs](/gan-eng.html) or VAEs, diffusion models generate images by iteratively denoising random noise, learning to reverse a process of gradual noise addition.

## The Core Concept: Forward and Reverse Processes

### Forward Process (Diffusion)

The forward process gradually adds Gaussian noise to data over T timesteps:

```
q(x_t | x_{t-1}) = N(x_t; √(1-β_t) * x_{t-1}, β_t * I)
```

Starting from clean data x_0, noise is progressively added until x_T becomes pure noise indistinguishable from Gaussian distribution. This process is deterministic and tractable, with a closed form:

```
x_t = √(ᾱ_t) * x_0 + √(1-ᾱ_t) * ε, where ε ~ N(0,I)
```

The coefficients ᾱ_t (cumulative product of 1-β_t) determine noise level at each step.

### Reverse Process (Denoising)

Generation occurs by learning to reverse this process: starting from noise x_T and iteratively denoising to recover x_0. The reverse process is also Gaussian:

```
p(x_{t-1} | x_t) = N(x_{t-1}; μ_θ(x_t, t), Σ_θ(x_t, t))
```

A [neural network](/neural-eng.html) learns the mean μ_θ and variance Σ_θ at each denoising step. During generation, we iteratively sample from this learned reverse process.

## DDPM: Denoising Diffusion Probabilistic Models

DDPM, introduced by Ho et al. (2020), establishes the foundation for modern diffusion models. The key insight is that the network can be simplified to predict the noise added at each step:

```
Loss = ||ε - ε_θ(x_t, t)||²
```

Instead of directly predicting the mean, the model predicts the noise ε that was added during the forward process. This reformulation dramatically improves training stability and convergence.

The training procedure:
1. Sample random t from 1 to T
2. Sample noise ε ~ N(0,I)
3. Compute noisy image x_t using the forward formula
4. Train network to predict ε from x_t and t
5. Backpropagate reconstruction error

## Noise Scheduling

Noise scheduling defines how β_t changes across timesteps, controlling the amount of noise added at each step. Effective scheduling is crucial for model performance.

### Linear Schedule

Simple linear progression from small β at early steps to large β at later steps. Easy to implement but may not be optimal.

### Cosine Schedule

```
β_t = sin²(π/2 * t/T)
```

Empirically performs better by gradually increasing noise in a more principled way, creating smoother transitions between timesteps.

### Squared Schedule and Variants

Various empirical schedules have been proposed, with success depending on dataset and application. Recent work explores learned or adaptive scheduling strategies.

## Denoising Score Matching

A deeper theoretical perspective connects diffusion models to score-based generative models. Score functions estimate the gradient of log-[probability](/probability-eng.html):

```
s_θ(x_t, t) = ∇_{x_t} log p(x_t)
```

Denoising score matching trains the network to predict this score, providing theoretical justification for the noise prediction objective. This connection opens paths to improved sampling and understanding.

## Classifier-Free Guidance

Classifier-free guidance enables conditional generation without training a separate classifier. Instead of using a classifier to guide generation toward a target class, the model learns to compare conditional and unconditional predictions.

During generation, the denoising step is adjusted:

```
ˆε_θ(x_t, t) = ε_θ(x_t, t, ∅) + w * (ε_θ(x_t, t, c) - ε_θ(x_t, t, ∅))
```

Where w is a guidance weight. Higher w values increase class conditioning strength but can reduce diversity. This technique enables powerful text-to-[image generation](/image-generation-eng.html) without additional networks.

## Stable Diffusion Architecture

Stable Diffusion introduced latent space diffusion, performing diffusion not on full-resolution images but on compressed latent representations. This dramatically reduces computational cost while maintaining quality.

The architecture consists of:

**Autoencoder:** Compresses images to compact latent space, reducing dimensionality 8-16x

**Diffusion U-Net:** Operates in latent space, performing iterative denoising

**CLIP Text Encoder:** Converts text prompts to embeddings for conditional generation

**Variational Decoder:** Reconstructs high-resolution images from latent representations

This three-component design made high-quality diffusion models accessible to broader audiences and enabled widespread adoption.

## Latent Diffusion Models

Latent diffusion extends the principle further by applying diffusion to learned latent spaces rather than pixel space. Benefits include:

- Computational efficiency: 4-16x speedup in training and inference
- Better alignment with perceptual distance metrics
- Ability to compose with other latent space methods
- Cleaner feature representations for manipulation

Latent diffusion enables practical deployment of high-resolution generation on consumer hardware.

## Training and Sampling

**Training:** Simple and stable compared to [GANs](/gan-eng.html). Requires predicting noise at random timesteps, with straightforward [backpropagation](/backpropagation-eng.html). No mode collapse or training instability.

**Sampling:** Iterative denoising from noise through T steps. Though slower than single-pass [GANs](/gan-eng.html), the quality-speed tradeoff favors diffusion models. Techniques like DDIM reduce required steps without sacrificing quality.

## Applications

**Text-to-Image:** DALL-E 3, Stable Diffusion, Midjourney—generating photorealistic images from descriptions

**Image-to-Image:** Style transfer, conditional generation based on sketch or mask

**Inpainting:** Intelligent image editing and content generation in masked regions

**Video Generation:** Temporal diffusion models for consistent frame synthesis

**3D Generation:** Extending diffusion to 3D shape and scene synthesis

**Medical Imaging:** Artifact removal, [super-resolution](/super-resolution-eng.html), and [data augmentation](/data-augmentation-eng.html)

## Advantages and Challenges

**Advantages:**
- State-of-the-art generation quality
- Stable, principled training without adversarial dynamics
- Flexible conditioning mechanisms (classifier-free guidance)
- Theoretical foundation from statistical physics
- Scalable to high resolutions

**Challenges:**
- Slow generation requiring many denoising steps
- High computational cost during sampling
- Training requires significant [GPU](/gpu-hardware-eng.html) memory
- Difficulty in fine-grained control over generated content
- Potential for mode averaging (less diverse than expected)

## Conclusion

Diffusion models revolutionized generative AI by achieving superior quality through principled probabilistic learning. Their stable training, flexibility, and scalability position them as the dominant generative paradigm. As research addresses sampling speed through techniques like consistency distillation and latent flows, diffusion models continue evolving toward practical deployment across countless applications.
