---
layout: default
title: "AI 코드 생성"
lang: ko
---

# AI 코드 생성

## 소개

AI 코드 생성은 모델이 자동으로 코드를 작성, 디버그, 최적화할 수 있게 합니다. GitHub Copilot, Claude Code, 특화된 모델(CodeLlama, StarCoder) 같은 도구는 개발자가 자연어 설명에서 코드를 생성할 수 있게 지원합니다. 코드 생성은 개발을 가속화하고, 보일러플레이트를 줄이고, 비전문가가 코드를 작성할 수 있게 합니다.

## 코드 생성 모델

### 대형 언어 모델

코드로 미세 조정된 범용 [LLM](/llm-kor.html):
- **GPT-4**: 강력한 일반 코드 생성, 다국어
- **Claude**: 코드와 함께 설명에 좋음
- **GPT-3.5**: 빠름, 단순 작업에 적절

### 특화된 코드 모델

주로 코드로 학습한 모델:
- **CodeLlama**: Meta의 코드 특화 LLaMA 변형
- **StarCoder**: BigCode의 다국어 코드 모델
- **Codex**: OpenAI의 [GPT](/gpt-kor.html)-4 선행

## 벤치마크

### HumanEval

코드 생성의 표준 [벤치마크](/benchmarks-kor.html):
- 164개 Python 함수
- Pass@1, Pass@10 메트릭 필요
- 최고 모델: [GPT](/gpt-kor.html)-4 (92%), Claude (88%), CodeLlama (53%)

### MBPP

Mostly Basic Python Programming:
- 974개 짧은 Python 함수
- HumanEval보다 더 다양
- 광범위한 코딩 능력 평가

## 사용 사례

### 코드 완성

개발자 자동 완성:
- 다음 줄 또는 블록 제안
- 저장소 컨텍스트에서 학습
- IDE에 통합 (Copilot, Cursor)

### 보일러플레이트 생성

반복적 코드 생성:
- 프레임워크 설정 코드
- 테스트 스캐폴딩
- 구성 파일

### 버그 수정

오류 식별 및 수정:
- 수정 제안
- 버그 설명
- 리팩토링 제안

## 도전 과제

### 환각된 함수

모델이 존재하지 않는 함수/라이브러리 발명:

```python
import non_existent_library
result = non_existent_library.solve(problem)
```

**완화**: 알려진 라이브러리로 제한, 가져오기 검증.

### 논리 오류

생성된 코드의 미묘한 버그:

```python
# 버그: off-by-one 오류
for i in range(len(array)-1):  # 마지막 요소 건너뜀
    process(array[i])
```

**완화**: 테스트 필요, 코드 검토, 테스트.

### 비효율적 솔루션

작동하지만 느린 코드.

## 결론

AI 코드 생성은 강력하지만 완벽하지 않습니다. 개발자 지원으로 가장 잘 사용됩니다.
