---
layout: default
title: "Model Serving & API Design"
lang: en
---

# Model Serving & API Design

## Introduction

Model serving is the final mile of the ML pipeline: getting trained models into production to make predictions on new data. Model serving systems must balance competing demands: low latency, high throughput, reliability, and resource efficiency. A poorly designed serving layer can bottleneck even excellent models.

## Popular Model Serving Frameworks

### TorchServe

PyTorch's production serving framework:
- Creates HTTP APIs from PyTorch models
- Built-in batching and GPU support
- Model management: versioning, A/B testing
- Scalable: single server or distributed cluster

```python
# Handler defines request/response logic
def handle(data, context):
    model = context.model
    return model(data)
```

### TensorFlow Serving

Google's production ML serving:
- Optimized for TensorFlow models
- Supports SavedModel format
- Model versioning and canary deployment built-in
- gRPC and REST API endpoints

### vLLM

Specialized for large language model inference:
- Token-level batching (batches incomplete sequences)
- Paged attention mechanism (memory efficient)
- Supports multi-GPU inference
- State-of-art throughput for LLMs

### Triton Inference Server

NVIDIA's inference platform:
- Supports multiple frameworks (TensorFlow, PyTorch, ONNX)
- Dynamic batching with configurable delays
- GPU memory optimization
- Model repository for easy deployment

### ONNX Runtime

Optimized inference runtime:
- Framework-agnostic (convert models to ONNX format)
- Cross-platform (CPU, GPU, mobile, web)
- Graph optimization and quantization
- Low latency inference

## Batching Strategies

### Static Batching

Fixed batch size, wait until batch is full:
- Predictable latency and throughput
- Wasteful if request rate varies
- Requires understanding traffic patterns

### Dynamic Batching

Variable batch size, timeout-based:
- Wait up to T milliseconds for batch
- Accumulate up to B requests
- More efficient under variable load
- Risk: high latency if timeout approaches

**Example**:
- Max batch size: 32
- Max wait: 100ms
- If 64 requests arrive within 100ms: two batches of 32
- If 16 requests, none more arrive: send batch of 16 after 100ms

### Token-Level Batching (for LLMs)

LLMs generate tokens sequentially. vLLM tokenizes requests on arrival, batches tokens:
- Far better throughput than padding to max length
- Interleave inference of multiple requests
- Essential for LLM efficiency

## Latency Optimization

### Request Batching

Group requests to amortize overhead:
- Single-request latency increases (wait for batch)
- Total throughput increases dramatically
- Trade-off between latency and throughput

### Model Compilation

Compile models to target hardware:
- Reduce model size via quantization
- Optimize operations for specific hardware
- TensorRT (NVIDIA), CoreML (Apple), TFLite quantization

### KV-Cache Optimization

For autoregressive models (language models, transformers):
- Precompute and cache key-value pairs
- Avoid recomputing on each token
- Reduces memory bandwidth requirements
- Critical for low-latency inference

### Mixed-Precision Inference

Use FP16/BF16 instead of FP32:
- 2x faster on modern hardware
- Minimal accuracy loss for many models
- Supported natively on A100, H100, newer GPUs

## API Design

### RESTful Inference

HTTP-based, simple integration:

```
POST /v1/predict
{
  "instances": [
    {"feature1": 1.0, "feature2": 2.0},
    {"feature1": 3.0, "feature2": 4.0}
  ]
}

Response:
{
  "predictions": [0.95, 0.12]
}
```

**Pros**: Language-agnostic, easy debugging
**Cons**: Higher latency, overhead of HTTP

### gRPC

Binary protocol, high-performance RPC:

```protobuf
service PredictionService {
  rpc Predict(PredictRequest) returns (PredictResponse);
}
```

**Pros**: Low latency, binary efficiency, HTTP/2 multiplexing
**Cons**: More complex, requires gRPC client

### Real-Time Streaming

For continuous predictions:
- Event streams (Kafka, Kinesis)
- WebSocket connections
- Useful for monitoring, anomaly detection

## Serving Architecture Patterns

### Single Model Serving

Deploy one model version:
- Simple, low overhead
- No version management complexity
- Limited for A/B testing

### Multi-Model Serving

Multiple model versions available:
- A/B testing: route traffic to different versions
- Model ensemble: combine predictions
- Requires careful resource allocation

### Model Ensemble

Combine multiple model predictions:

```
predictions = [
  model_1.predict(x),
  model_2.predict(x),
  model_3.predict(x)
]
ensemble_output = average(predictions)
```

**Benefits**: Improved robustness, better generalization
**Cost**: Higher latency, more resource usage

## Deployment Strategies

### Blue-Green Deployment

Two identical environments:
- Blue (old model), Green (new model)
- Route all traffic to one, test the other
- Instant rollback by switching traffic

### Canary Deployment

Gradual traffic shift:
- Route 5% traffic to new model
- Monitor metrics
- Increase if no issues: 10%, 25%, 50%, 100%
- Automatic rollback if errors detected

### Shadow Deployment

New model runs in parallel:
- Receive same requests as production model
- Don't use predictions (just log)
- Compare predictions offline
- Zero risk to users

## Monitoring & Observability

**Metrics**:
- Throughput (requests/sec)
- Latency (p50, p99)
- Error rate
- Hardware utilization (GPU, memory, CPU)
- Model performance (accuracy, precision on live data)

**Alerting**:
- Latency degradation
- Error rate spikes
- Model performance drops
- Resource saturation

**Logging**:
- Request/response pairs (for debugging)
- Model predictions with metadata
- Errors and exceptions
- Enable offline analysis and model improvement

## Cost Optimization

- **Right-sizing**: Match hardware to workload (don't over-provision)
- **Spot instances**: Use cheaper preemptible VMs for non-critical services
- **Auto-scaling**: Scale down during low traffic periods
- **Model distillation**: Train smaller models matching larger ones
- **Quantization**: Reduce precision for faster inference
- **Caching**: Cache frequent queries (same input = same output)

## Conclusion

Model serving bridges the gap between research and production. Success requires understanding latency-throughput trade-offs, designing efficient architectures, and implementing comprehensive monitoring. The best model is worthless if it doesn't reach users reliably and efficiently.
