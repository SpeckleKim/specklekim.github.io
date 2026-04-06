---
layout: default
title: "설명 가능한 AI"
lang: ko
---

# 설명 가능한 AI (XAI)

## 소개

설명 가능한 AI(XAI)는 모델 결정을 투명하고 해석 가능하게 만듭니다. AI 시스템이 점점 더 중요한 결정(대출 승인, 의료 진단, 채용)을 내릴수록 모델이 결정을 내린 *이유*를 이해하는 것이 중요해집니다.

## XAI 기술

### SHAP

SHapley Additive exPlanations:
- 특성 중요도에 대한 게임 이론 접근
- 예측을 특성에 공정하게 귀속
- 계산 비용이 높지만 원칙 기반

### LIME

Local Interpretable Model-agnostic Explanations:
- 해석 가능한 대체로 로컬 모델 근사
- 특정 예측에 대한 설명 생성
- 모든 모델 유형에서 작동

### Grad-CAM

Gradient-weighted Class Activation Mapping:
- 이미지 분류기용
- 어느 영역이 예측에 영향을 미쳤는지 시각화
- 예측 클래스의 경사 사용

## 결론

XAI는 신뢰할 수 있는 AI에 필수입니다. 여러 기술을 사용하고 설명을 검증합니다.
