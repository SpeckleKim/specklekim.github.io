---
layout: default
title: 배치 정규화
lang: ko
---

# 배치 정규화: 딥러닝 안정화

배치 [정규화](/regularization-kor.html)(BatchNorm)는 계층 입력을 평균 0, 분산 1로 정규화하여 훈련을 안정화하고 높은 [학습률](/learning-rate-scheduling-kor.html)을 가능하게 합니다. 내부 공변량 시프트 문제를 해결하는 딥러닝에서 가장 영향력 있는 기법 중 하나입니다.

## 내부 공변량 시프트 (Internal Covariate Shift)

내부 공변량 시프트는 훈련 중 계층으로의 입력 분포가 변할 때 발생합니다. 이전 계층이 가중치를 업데이트하면 이후 계층으로 흐르는 데이터 분포가 변하고, 그 계층들은 끊임없이 재적응해야 합니다.

예시: 초기 계층 가중치가 크게 변하면, 두 번째 계층으로 들어오는 활성화 분포가 변환됩니다. 두 번째 계층은 이 새로운 분포에 가중치를 다시 적응시켜야 합니다.

이러한 분포의 변화는 학습을 느리게 하고 신중한 하이퍼파라미터 조정(낮은 [학습률](/learning-rate-scheduling-kor.html))을 요구합니다.

## 배치 정규화 알고리즘

훈련 중, 배치 [정규화](/regularization-kor.html)는 각 미니배치 내에서 활성화를 정규화합니다:

1. **배치 통계 계산**:
   - `μ_배치 = (1/m) * Σ x_i` (배치 평균)
   - `σ²_배치 = (1/m) * Σ (x_i - μ_배치)²` (배치 분산)

2. **정규화**:
   - `x̂_i = (x_i - μ_배치) / √(σ²_배치 + ε)`

3. **스케일 및 시프트** (학습 가능 매개변수):
   - `y_i = γ * x̂_i + β`

매개변수 γ(스케일)과 β(시프트)는 훈련 중 학습되므로, 네트워크는 필요하면 [정규화](/regularization-kor.html)를 취소할 수 있습니다.

## 훈련 vs. 추론

배치[정규화](/regularization-kor.html)는 훈련과 추론에서 다르게 작동합니다:

**훈련 중**: 현재 미니배치의 통계를 사용합니다.

**추론 중**: 훈련 중 계산된 누적 통계(지수 이동 평균)를 사용하여 [정규화](/regularization-kor.html)합니다:
- `x_[정규화](/regularization-kor.html) = (x - running_mean) / √(running_var + ε)`

이것은 중요합니다: 추론 중 배치 통계를 사용하면 테스트 배치가 다른 분포를 가질 수 있으므로 잘못됩니다.

## 누적 통계 (Running Statistics)

훈련 중, 지수 이동 평균이 배치 통계를 누적합니다:

`running_mean = momentum * running_mean + (1 - momentum) * batch_mean`
`running_var = momentum * running_var + (1 - momentum) * batch_var`

momentum=0.99일 때, 최근 배치가 누적 통계에 크게 영향을 미치면서도 안정성을 유지합니다.

이러한 누적 통계는 추론 중 사용되는 매개변수가 됩니다.

## 배치 정규화의 이점

**더 빠른 수렴**: 내부 공변량 시프트를 감소시켜 불안정 없이 높은 [학습률](/learning-rate-scheduling-kor.html)을 가능하게 합니다.

**정규화 효과**: 배치 통계의 무작위성이 암묵적 [정규화](/regularization-kor.html)로 작동하여 과적합을 종종 줄입니다.

**가중치 초기화 민감도 감소**: [정규화](/regularization-kor.html) 계층이 활성화를 재안정화하므로 네트워크는 신중한 초기화에 덜 의존합니다.

**더 높은 학습률 가능**: 더 안정적인 기울기 흐름으로 더 공격적인 [학습률](/learning-rate-scheduling-kor.html) 스케줄을 허용합니다.

**단순화된 하이퍼파라미터 튜닝**: BatchNorm은 신중한 [학습률](/learning-rate-scheduling-kor.html)과 초기화 튜닝의 필요성을 줄입니다.

## CNN에서의 배치 정규화

[합성곱 신경망](/cnn-deep-dive-kor.html)에서 배치[정규화](/regularization-kor.html)는 일반적으로 합성곱 후, 활성화 전에 적용됩니다:

`합성곱 → BatchNorm → ReLU → 풀링`

배치[정규화](/regularization-kor.html)는 각 채널에서 독립적으로 작동하여 공간 및 배치 차원에 걸쳐 통계를 계산합니다.

형태 (batch_size, height, width, channels)의 배치의 경우, 통계는 축 (배치, 높이, 너비)에 걸쳐 계산되어 채널별 통계가 남습니다.

## RNN에서의 배치 정규화

순차적 의존성이 시간 단계를 [정규화](/regularization-kor.html)하면 시간적 상관관계가 손상될 수 있으므로 배치정규화를 RNN에 적용하는 것은 더 복잡합니다.

해결책:
- **계층 정규화**: 배치가 아닌 특성을 [정규화](/regularization-kor.html). 예시별 평균/분산을 계산하여 RNN에 적합.
- **가중치 정규화**: 가중치를 직접 재매개변수화.
- **순환 배치정규화**: 순환 연결에 신중하게 배치[정규화](/regularization-kor.html)를 적용.

계층 [정규화](/regularization-kor.html)는 RNN과 [트랜스포머](/llm-kor.html)에서 배치정규화보다 더 인기가 되었습니다.

## 배치 정규화의 대안

**계층 정규화 (Layer Normalization)**: 특성을 [정규화](/regularization-kor.html):
- 배치 크기 독립적(작은 배치에 좋음)
- RNN과 [트랜스포머](/llm-kor.html)에서 잘 작동
- 현대 아키텍처의 표준

**그룹 정규화 (Group Normalization)**: 채널 그룹을 [정규화](/regularization-kor.html):
- 배치 크기가 작을 때 유용
- 모든 배치 크기에서 작동

**인스턴스 정규화 (Instance Normalization)**: 인스턴스당 채널별로 [정규화](/regularization-kor.html):
- 스타일 이전과 이미지-이미지 번역에 사용

**위치 정규화 (Positional Normalization)**: 공간적으로 [정규화](/regularization-kor.html):
- 특정 애플리케이션을 위한 신흥 기법

## 배치 정규화의 도전 과제

**배치 크기 의존성**: 작은 배치 크기는 노이즈 많은 통계를 초래. 작은 배치(batch_size < 16)에서 성능 저하.

**훈련-추론 불일치**: 배치 통계와 누적 통계 사용 간의 불일치가 때때로 성능을 해칠 수 있습니다.

**계산 오버헤드**: BatchNorm은 메모리와 계산을 추가합니다(보통 약 30% 오버헤드).

**항상 도움이 되지는 않음**: 일부 경우(작은 데이터셋, 작은 모델)에서 배치[정규화](/regularization-kor.html)는 도움이 되지 않거나 해를 미칠 수 있습니다.

## 모범 사례

1. **대규모 네트워크에는 항상 사용**: 깊은 네트워크는 크게 이점을 봅니다.

2. **작은 배치 주의**: 배치 크기 < 32일 때 대안 [정규화](/regularization-kor.html)(GroupNorm, LayerNorm) 사용.

3. **모멘텀 값**: 0.9 또는 0.99가 전형적. 더 높은 모멘텀은 더 오래된 이력 유지.

4. **다른 정규화와 제거**: [드롭아웃](/dropout-kor.html)을 많이 사용하면 배치[정규화](/regularization-kor.html)의 정규화 효과가 중복될 수 있습니다.

5. **현대적 선호**: 많은 최신 아키텍처는 배치[정규화](/regularization-kor.html) 대신 계층정규화 또는 그룹정규화를 사용합니다.

## 요약

배치 [정규화](/regularization-kor.html)는 중간 활성화를 정규화하여 내부 공변량 시프트를 해결하는 강력한 기법입니다. 더 빠르고 안정적인 훈련을 가능하게 하며 현대 딥러닝에서 표준이 되었습니다. 훈련 및 추론 모드의 구분, 그리고 계층정규화 같은 대안이 더 적절한 경우를 아는 것이 효과적인 딥러닝 실무에 필수적입니다. 배치정규화가 더 이상 보편적으로 적용되지는 않지만(현대 [트랜스포머](/llm-kor.html)는 계층정규화를 선호), 네트워크가 어떻게 학습하고 안정화하는지 이해하는 데 여전히 중요합니다.
