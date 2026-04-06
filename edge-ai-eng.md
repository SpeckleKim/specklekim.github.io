---
layout: default
title: "Edge AI & On-Device ML"
lang: en
---

# Edge AI & On-Device ML

## Introduction

Edge AI brings machine learning inference to devices at the edge of the network—smartphones, tablets, IoT devices, embedded systems. Rather than sending data to cloud servers for inference, models run locally on device. This enables privacy-preserving inference, reduced latency, offline functionality, and bandwidth efficiency. Edge AI is becoming critical as mobile and IoT deployments explode.

## Why Edge AI?

### Privacy & Data Security

- Data stays on device, never sent to servers
- Complies with privacy regulations (GDPR, CCPA)
- Users retain control of personal information
- Particularly important for health, financial, biometric data

### Low Latency

- No network round-trip
- Inference in milliseconds on modern devices
- Critical for real-time applications (gaming, AR/VR, autonomous vehicles)

### Offline Functionality

- Works without internet connection
- Essential for rural areas, airplanes, unreliable networks
- Enables battery-powered devices to operate indefinitely

### Bandwidth Efficiency

- No need to upload data to cloud
- Reduces data plan usage
- Lowers infrastructure costs

## Model Compression Techniques

### Quantization

Reduce numerical precision from FP32 to lower precision:

**INT8 Quantization**: 4x model size reduction
- Scale weights and activations to 8-bit integers
- Minimal accuracy loss for most models
- Supported on all mobile devices

**FP16 Quantization**: 2x size reduction
- Half-precision floating point
- Preserves more information than INT8
- Requires compatible hardware

**Binary/Ternary Quantization**: 32x size reduction
- Weights as single bit or ternary (-1, 0, +1)
- Significant accuracy trade-offs
- Research stage, limited production use

### Pruning

Remove unimportant weights or neurons:

**Magnitude-based**: Remove weights < threshold
- Simple to implement
- Can damage important connections

**Lottery Ticket Hypothesis**: Sparse subnetwork works as well as full network
- Train network, identify important weights, remove others
- Re-train only important weights

**Structured Pruning**: Remove entire filters or layers
- Produces actual speedups (vs unstructured = irregular memory access)
- Smaller accuracy gains than unstructured

### Knowledge Distillation

Train small student model from large teacher model:

```
Teacher (large) -----(knowledge)----> Student (small)
                  Temperature scaling
                  to soften probabilities
```

- Student learns to mimic teacher's behavior
- Much smaller than teacher
- Maintains most of teacher's performance

### Tensor Decomposition

Decompose weight tensors into smaller factors:
- Rank reduction: approximate high-rank tensor with lower-rank
- Reduces computation and memory
- Less common than other techniques

## Mobile ML Frameworks

### TensorFlow Lite

Google's lightweight inference engine:
- Converter optimizes models for mobile ([quantization](/quantization-eng.html), pruning)
- Native support for Android, iOS
- Supports models from TensorFlow, PyTorch (via ONNX)
- 3-4x smaller than full TensorFlow

```python
# Convert model
converter = tf.lite.TFLiteConverter.from_saved_model('model/')
tflite_model = converter.convert()

# Load and run on device
interpreter = tf.lite.Interpreter('model.tflite')
interpreter.invoke()
```

### ONNX (Open Neural Network Exchange)

Framework-agnostic model format:
- Convert models from PyTorch, TensorFlow, etc. to ONNX
- Standardized format for model interchange
- ONNX Runtime optimizes inference
- Good ecosystem support

### Core ML (Apple)

Apple's on-device ML framework:
- Automatic optimization for Apple hardware
- Neural Engine support (M-series, A-series chips)
- Privacy-by-default (no data leaves device)
- Convert from TensorFlow, PyTorch via converter

### MediaPipe

Google's multimedia ML framework:
- Pre-built solutions: pose detection, hand tracking, [object detection](/object-detection-eng.html)
- Optimized for real-time mobile inference
- Cross-platform (Android, iOS, web)
- Highly usable for production

## Hardware for Edge AI

### Mobile Processors

**Apple Neural Engine**: 16-core NPU in M-series, A-series
- 100x more efficient than CPU for ML
- Specialized for INT8 operations
- Often faster than [GPU](/gpu-hardware-eng.html) for small models

**Qualcomm Hexagon NPU**: In Snapdragon processors
- Integer and fixed-point computation
- Competitive with Apple Neural Engine
- Available across Android ecosystem

**Google Tensor Chip**: In Pixel phones
- Custom hardware for specific ML tasks
- TPU-like design optimized for Google services
- Strong for AI tasks like portrait mode, real-time translation

### Embedded Accelerators

**TPU Lite**: Smaller TPU for edge deployment
- Better power efficiency than [GPU](/gpu-hardware-eng.html)
- Smaller batch sizes than cloud TPUs

**NVIDIA Jetson**: [GPU](/gpu-hardware-eng.html)-based edge computing
- Full CUDA support for complex models
- Overkill for simple inference
- Good for robotics, autonomous vehicles

## Optimization Techniques for Edge

### Post-Training Quantization

Quantize already-trained model without retraining:
- Fast, simple
- Accuracy loss varies by model
- Good baseline approach

### Quantization-Aware Training

Simulate [quantization](/quantization-eng.html) during training:
- Model learns to handle lower precision
- Better accuracy than post-training [quantization](/quantization-eng.html)
- Requires retraining

### Model Architecture Search for Edge

AutoML techniques to find edge-optimized architectures:
- MobileNet, ShuffleNet: designed for mobile
- EfficientNet: scale compound (width, depth, resolution)
- MobileViT: [vision transformers](/vision-transformers-eng.html) for mobile

### Dynamic Input Resolution

Accept variable input sizes:
- Adapt to device capability
- Resizes images to optimal resolution
- Balances accuracy vs speed

## On-Device Learning

Fine-tune models with user data:
- Adapt to specific user/context
- Preserve privacy (no uploading user data)
- [Federated learning](/privacy-ai-eng.html): train on multiple devices, aggregate updates

**Challenges**:
- Limited computational resources
- Battery drain
- Memory constraints

**Approaches**:
- Gradient-based fine-tuning on small subsets
- Adapter modules: add small trainable components
- Hypernetworks: generate network weights dynamically

## Deployment Considerations

### Model Size Management

- Typical budget: 20-100 MB total app size
- Model alone: 5-50 MB depending on complexity
- Trade off model size vs capability

### Memory Management

- RAM constraints (1-4 GB typical)
- Model loaded once, shared across requests
- Feature preprocessing impacts memory

### Battery Impact

- ML inference draws power
- Optimize for battery efficiency
- Balance accuracy vs power consumption

### Update Mechanism

- How to update models? OTA updates?
- Version management
- [A/B testing](/ab-testing-ml-eng.html) on device

## Production Examples

**Face Recognition (Apple FaceID)**
- Real-time face detection and recognition
- Entirely on device
- Sub-millisecond latency

**Real-time Pose Detection (TikTok, Instagram)**
- 17-point body skeleton detection
- 30-60 FPS on smartphones
- Runs on Snapdragon/Apple chips

**Smart Keyboard Prediction**
- Next-word prediction
- On-device learned from user behavior
- Improves over time with usage

## Future Directions

- **Neuromorphic hardware**: Event-based processing, ultra-low power
- **Photonic processing**: Optical inference at edge
- **Quantum**: Quantum processors for specific edge tasks
- **Continual learning**: Continuous adaptation to user/environment

## Conclusion

Edge AI removes computation from cloud to device, enabling privacy, responsiveness, and offline capability. Success requires careful model optimization, understanding hardware constraints, and robust deployment pipelines. As mobile and IoT proliferate, edge AI becomes central to responsible, performant AI deployment.
