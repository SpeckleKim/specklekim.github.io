---
layout: default
title: 혼합 정밀도 학습
lang: ko
---

# 혼합 정밀도 학습

## 소개

혼합 정밀도 학습(Mixed Precision Training)은 [신경망](/neural-kor.html) 학습 중 서로 다른 수치 정밀도 형식을 결합합니다. 모든 연산에 32비트 부동소수점(FP32)을 사용하는 대신, 혼합 정밀도 학습은 특정 연산에는 16비트 형식(FP16 또는 BF16)을 사용하고 다른 부분에는 FP32를 유지합니다. 이 접근 방식은 메모리 소비를 극적으로 줄이고 모델 정확도에 미미한 영향을 미치면서 학습 속도를 증가시킵니다.

## 정밀도 형식: FP32, FP16, BF16

**FP32 (Float32)**:
- 32비트 형식: 1 부호 비트, 8 지수 비트, 23 가수 비트
- 범위: ±1.4×10⁻⁴⁵ ~ ±3.4×10³⁸
- 높은 정밀도, 깊은 학습의 표준 형식
- 더 높은 메모리 및 계산 비용

**FP16 (Float16)**:
- 16비트 형식: 1 부호 비트, 5 지수 비트, 10 가수 비트
- 범위: ±6.1×10⁻⁵ ~ ±65,504
- FP32 대비 2배 메모리 감소
- [GPU](/gpu-hardware-kor.html) 텐서 코어에서 더 빠른 계산
- 제한된 수치 범위로 인한 오버플로우/언더플로우 문제

**BF16 (Brain Float16)**:
- 16비트 형식: 1 부호 비트, 8 지수 비트, 7 가수 비트
- 범위: ±1.4×10⁻⁴⁵ ~ ±3.4×10³⁸ (FP32와 동일!)
- FP16 메모리로 FP32 동적 범위 유지
- Google에서 더 나은 안정성을 위해 개발
- 많은 응용에서 FP16보다 우수

## 메모리 및 속도 이점

**메모리 감소**:
- 활성화를 FP16/BF16로 저장: 50% 감소
- FP16/BF16의 그래디언트: 50% 감소
- FP32의 [옵티마이저](/optimizers-kor.html) 상태: 안정성을 위해 필요
- 전체적: 20-50% 메모리 감소 (아키텍처에 따라)

**계산 속도**:
- NVIDIA 텐서 코어: 행렬 연산용 특화 하드웨어
- FP16 텐서 연산: 최신 GPU에서 FP32보다 최대 8배 빠름
- 대역폭 감소: 절반의 메모리 전송
- 실제 속도: 메모리 대역폭으로 제한되어 실제로는 2-4배

## 도전: 수치 안정성

낮은 정밀도로의 학습은 수치 문제를 야기합니다:

**그래디언트 언더플로우**: 그래디언트가 너무 작아져 FP16에서 표현할 수 없게 되어 중요한 정보 손실

**손실 스케일링 해결책**:
```
1. FP32에서 손실 계산
2. 큰 계수로 손실 스케일링 (일반적으로 2^15 또는 2^16)
3. 스케일된 손실의 그래디언트 계산 (언더플로우 방지)
4. 옵티마이저 스텝 이전에 그래디언트 언스케일링
5. FP32에서 매개변수 업데이트
```

**동적 손실 스케일링**:
- 정적 스케일링은 너무 많이 스케일링할 수 있음 (오버플로우) 또는 너무 적게 할 수 있음
- 오버플로우 감지에 따라 동적으로 스케일링 계수 조정
- 알고리즘:
  1. 현재 스케일로 학습 스텝 시도
  2. 오버플로우 감지되면 스케일 감소 (2로 나눔)
  3. N 스텝 동안 오버플로우 없으면 스케일 증가 (2로 곱함)
- PyTorch/TensorFlow 구현은 자동으로 처리

## NVIDIA 텐서 코어

텐서 코어는 행렬 곱셈 가속을 위한 특화 하드웨어 단위입니다:

**아키텍처**:
- 각 코어는 하나의 연산으로 수행: D = A×B + C
- A와 B는 FP16 또는 BF16, FP32에서 누적
- 클록당 8x4x4 연산
- 최신 GPU는 SM당 64-80개의 텐서 코어

**성능**:
- V100: 125 TFLOPS FP16 텐서 연산 vs 15.7 TFLOPS FP32
- A100: 312 TFLOPS BF16 vs 19.5 TFLOPS FP32
- 순수 텐서 연산에 대해 약 16배의 이론적 속도향상

**활용 요구사항**:
- 배치 크기와 행렬 차원은 8의 배수여야 함
- 정렬된 메모리 접근 패턴
- 큰 행렬 (>32×32)을 오버헤드 상각하기 위해

## 실제 구현: PyTorch AMP

**자동 혼합 정밀도(AMP)**:
- PyTorch의 기본 혼합 정밀도 지원
- 두 가지 접근: torch.cuda.amp 및 더 새로운 torch.amp

**torch.cuda.amp 사용**:

```python
from torch.cuda.amp import autocast, GradScaler

model = MyModel().cuda()
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
scaler = GradScaler()

for epoch in range(epochs):
    for batch in dataloader:
        optimizer.zero_grad()

        # 낮은 정밀도에서의 전진 패스
        with autocast():
            outputs = model(batch['input'])
            loss = criterion(outputs, batch['target'])

        # 손실 스케일링을 사용한 역진 패스
        scaler.scale(loss).backward()
        scaler.unscale_(optimizer)
        scaler.step(optimizer)
        scaler.update()
```

**주요 구성 요소**:
- `autocast()`: 자동으로 연산을 FP16/BF16으로 캐스팅
- `GradScaler()`: 손실 스케일링을 자동으로 처리
- 입력 유형에 따라 연산이 자동으로 캐스팅됨

**TensorFlow 구현**:

```python
optimizer = tf.keras.optimizers.SGD(learning_rate=0.01)
loss_scale = tf.keras.mixed_precision.LossScale()

with tf.GradientTape() as tape:
    outputs = model(inputs)
    loss = criterion(outputs, targets)
    scaled_loss = loss_scale.scale(loss)

gradients = tape.gradient(scaled_loss, model.trainable_variables)
gradients = loss_scale.unscale(gradients)
optimizer.apply_gradients(zip(gradients, model.trainable_variables))
```

## 모범 사례 및 고려 사항

**배치 크기**:
- 혼합 정밀도로 더 큰 배치(128-512)가 종종 필요
- 메모리 절감으로 더 큰 배치 허용
- 더 큰 배치로 그래디언트 안정성 개선

**학습률**:
- FP32 학습과 비교하여 조정 필요할 수 있음
- 종종 약간 낮은 [학습률](/learning-rate-scheduling-kor.html)이 더 잘 작동
- [학습률](/learning-rate-scheduling-kor.html) 스케일링 기법 (선형 워밍업) 권장

**그래디언트 자르기**:
- 규범 기반 자르기는 오버플로우 방지에 도움
- 언스케일 후 자르기: `clip_grad_norm_(model.parameters(), max_norm=1.0)`
- 극한 그래디언트 값 방지

**배치 정규화**:
- 배치 규범 계산을 FP32로 유지
- 통계 정확도 유지에 도움
- 최신 프레임워크는 자동으로 처리

**초기화**:
- 표준 He 초기화가 잘 작동
- 특별한 초기화 일반적으로 필요 없음
- 일부 모델은 신중한 초기화의 이점

## 개선된 학습을 위한 기법

**자동 그래디언트 누적**:
- FP16 계산에도 불구하고 FP32에서 그래디언트 누적
- 수렴 안정성 개선
- 대부분의 AMP 구현에 내장

**마스터 가중치 복사**:
- FP32에서 마스터 가중치 유지
- FP16에서 전진/역진
- [옵티마이저](/optimizers-kor.html)는 FP32 마스터 업데이트, 다음 반복에 FP16으로 복사

**정밀도 튜닝**:
- 일부 연산이 다른 것보다 민감함
- Softmax는 일반적으로 안정성을 위해 FP32 필요
- 계층 [정규화](/regularization-kor.html)는 종종 더 높은 정밀도의 이점
- PyTorch의 `cast_to_lowest_precision` 매개변수는 세밀한 제어 허용

## 호환성 및 한계

**하드웨어 요구사항**:
- FP16 텐서 연산: 계산 능력 ≥ 7.0의 NVIDIA [GPU](/gpu-hardware-kor.html) (V100, A100)
- BF16: NVIDIA A100 이상
- AMD [GPU](/gpu-hardware-kor.html): RDNA 시리즈 이상
- Intel Habana Gaudi

**모델 아키텍처 민감성**:
- 표준 아키텍처 (ResNet, Transformer, VGG): 잘 작동
- 매우 깊은 계층이나 [스킵 연결](/residual-connections-kor.html)이 있는 모델: 때때로 추가 주의 필요
- 커스텀 연산은 최적화된 FP16 커널이 없을 수 있음

**학습 역학**:
- 손실 곡선이 FP32와 약간 다를 수 있음
- 최종 모델 정확도는 일반적으로 FP32의 0.1-0.5% 이내
- 수렴 패턴이 다를 수 있음

## 성능 측정

**실제 속도향상 공식**:
```
속도향상 ≈ 2.5배 ~ 4배 (현실)
(텐서 코어의 이론적 최대는 8-16배, 메모리 대역폭 및 기타 요소로 제한)
```

**벤치마킹**:
1. FP32 학습 시간을 에포크당 측정
2. 혼합 정밀도 학습 시간을 에포크당 측정
3. 모델 정확도가 원본의 0.5% 이내인지 확인
4. 실제 속도향상 비율 계산

## 혼합 정밀도 학습을 사용할 시기

**사용할 때**:
- 텐서 코어가 있는 [GPU](/gpu-hardware-kor.html) (V100 또는 최신 NVIDIA)
- 학습 시간이 중요함
- 메모리가 병목
- 모델이 큼 ([BERT](/bert-kor.html), ResNet-50 등)
- 배치 크기를 증가할 수 있음

**건너뛸 때**:
- 제한된 정밀도 [GPU](/gpu-hardware-kor.html) (오래된 NVIDIA, 소비자 GPU)
- CPU에서 실행 (이점 없음)
- 매우 작은 모델 학습
- 정밀도 요구사항이 중요함 (금융 데이터)

## 결론

혼합 정밀도 학습은 정밀도 손실을 최소화하면서 상당한 속도향상과 메모리 절감을 제공하는 강력한 최적화 기법입니다. 텐서 코어와 같은 특화 하드웨어를 활용하고 지능형 손실 스케일링을 결합함으로써, 최신 깊은 학습 프레임워크는 혼합 정밀도 학습을 사용하기 쉽고 효과적으로 만듭니다. 모델이 커지고 계산 자원이 제한될수록, 혼합 정밀도는 프로덕션 깊은 학습 시스템의 표준 관행이 되었습니다.

