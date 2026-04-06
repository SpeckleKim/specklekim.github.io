---
layout: default
title: "도구 사용과 함수 호출"
lang: ko
---

# 도구 사용과 함수 호출

## 소개

도구 사용(함수 호출이라고도 함)은 언어 모델이 추론 중에 외부 함수와 API를 호출할 수 있게 합니다. 텍스트만 생성하는 대신 LLM은 이제 도구를 호출할 수 있습니다. 데이터베이스 검색, 코드 실행, 실시간 데이터 가져오기, 장치 제어로 모델을 외부 시스템과 상호 작용할 수 있는 [에이전트](/agent-kor.html)로 만듭니다. 도구 사용은 실질적인 AI 시스템 구축의 기초입니다.

## 도구 사용 패러다임

### 전통적인 LLM 출력

LLM은 텍스트만 생성합니다:
```
사용자: "런던의 날씨는?"
LLM: "실시간 날씨 데이터에 액세스할 수 없습니다..."
```

### 도구 사용 포함

LLM은 도구를 호출할 수 있습니다:
```
사용자: "런던의 날씨는?"
LLM: [도구 호출] weather_api(location="London")
도구: {"temp": 15C, "condition": "rainy"}를 반환
LLM: "런던은 15°C이고 비가 옵니다."
```

## 함수 호출 API

### OpenAI

OpenAI의 [GPT](/gpt-kor.html) 모델은 기본적으로 함수_호출을 지원합니다:

```python
functions = [
    {
        "name": "get_weather",
        "description": "위치의 날씨 가져오기",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string"}
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "런던 날씨?"}],
    functions=functions
)
```

모델은 구조화된 형식으로 도구 호출을 반환하고 시스템이 실행합니다.

### Anthropic

Claude는 messages API를 통해 도구 사용을 지원합니다:

```python
tools = [
    {
        "name": "weather",
        "description": "날씨 데이터 가져오기",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {"type": "string"}
            }
        }
    }
]

response = client.messages.create(
    model="claude-opus",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "날씨?"}]
)
```

### 오픈 소스 모델

- Gorilla: 함수 호출에 특화
- Hermes가 있는 LLaMA: 함수 호출 미세 조정
- Mistral: 기본 도구 호출 지원
- 더 작은 모델은 복잡한 도구 사용에 덜 신뢰함

## 도구 선택 및 라우팅

### 고정 도구 집합

모델에 사용 가능한 사전 정의 도구:
- 도구를 미리 선언
- 모델이 사용할 도구 선택
- 간단하고 결정론적

**제한**: 모델이 정의되지 않은 도구를 호출할 수 없으며, 모든 필요를 예상해야 합니다.

### 동적 도구 발견

런타임에 발견된 도구:
- 모델에 사용 가능한 도구 설명
- 모델이 사용 가능한 세트에서 선택
- 더 유연하지만 모델은 설명을 이해해야 함

**도전**: 도구 기능의 모델 이해 보장.

### 계층적 도구 선택

여러 수준의 도구 선택:
- 상위 수준: "도구 범주"
- 하위 수준: "범주 내 특정 도구"
- 각 단계의 결정 공간 감소

**예**:
1. 범주: "정보 검색"
2. 특정: "웹 검색" vs "데이터베이스 쿼리"

## 도구 컴포지션

### 순차적 도구 호출

한 도구의 호출 출력이 다음으로 공급:

```
사용자: "Amazon vs BestBuy에서 iPhone 15 가격 비교"
1. search_amazon("iPhone 15")
2. search_bestbuy("iPhone 15")
3. LLM이 가격 비교
```

### 병렬 도구 호출

여러 도구 동시 호출:

```
search_amazon("iPhone 15") ------\
search_bestbuy("iPhone 15") ------> LLM 비교
search_walmart("iPhone 15") -----/
```

순차보다 빠르지만 더 많은 계산이 필요합니다.

### 조건부 도구 호출

도구 선택이 조건에 따라 달라짐:

```
if user asks about time:
    call get_current_time()
elif user asks about weather:
    call weather_api()
else:
    call general_knowledge()
```

## 오류 처리 및 검증

### 도구 실패 처리

도구가 오류를 반환하면?

```python
try:
    result = weather_api("London")
except ToolException as e:
    # 옵션 1: LLM에 오류 알림, 다르게 재시도하게 함
    # 옵션 2: 다른 파라미터로 자동 재시도
    # 옵션 3: 사용자에게 오류 반환
```

LLM은 우아한 오류 처리가 필요합니다. 유효하지 않은 도구 호출에 취약합니다.

### 파라미터 검증

도구 파라미터가 유효한지 확인:

```python
def get_weather(location: str, units: str = "celsius"):
    if units not in ["celsius", "fahrenheit"]:
        raise ValueError("Invalid units")
    return weather_api(location, units)
```

형식이 잘못된 데이터를 도구에 전달하는 것을 방지합니다.

### 환각 방지

LLM은 도구 이름이나 파라미터를 만들어낼 수 있습니다:

```
사용자: "날씨는?"
LLM [환각]: "도구 호출: weather_predictor..."
시스템: "도구 'weather_predictor' 찾을 수 없음"
LLM: "대신 weather() 사용하겠습니다"
```

명시적 오류 메시지를 포함하여 수정을 안내합니다.

## 실제 도구 통합

### 검색 도구

모델이 웹을 검색할 수 있게:
- 검색 엔진 (Google, Bing API)
- 지식 기반 (Wikipedia, 문서)
- 내부 데이터베이스

**예**: "최근 AI 혁신에 대해 알려주세요" → 뉴스 + Wikipedia 검색.

### 계산 도구

올바른지 확인하기 위해 수학을 오프로드:
- 계산기
- 대수 솔버
- 통계 함수

**예**: "17234 * 8472는?" → 계산기 도구로 정확성 보장.

### 코드 실행

동적으로 코드 실행:
- Python REPL
- 샌드박스된 코드 환경
- 데이터 분석 (pandas, matplotlib)

**예**: "판매 데이터를 그려주세요" → 제공된 데이터로 Python 실행.

### API 통합

프로덕션 API 호출:
- CRM (Salesforce, HubSpot)
- 데이터베이스 (SQL 쿼리)
- SaaS 서비스 (Slack, 이메일)

**도전**: 인증, 권한 경계, 감사 추적.

### 파일 시스템

파일 읽기/쓰기:
- 지식 기반 쿼리
- 문서 검색
- 파일 생성

**보안**: 파일 액세스를 샌드박스하여 무단 읽기 방지.

## 모범 사례

### 명확한 도구 설명

설명은 명확하게 목적을 전달해야:

```python
# 나쁨: "데이터 가져오기"
# 좋음: "지정된 도시의 현재 날씨 조건 검색"

# 나쁨: "정보 처리"
# 좋음: "샌드박스된 환경에서 Python 코드 실행"
```

### 최소 도구 집합

도구가 적을수록 모델 성능이 좋음:
- 혼동 감소
- 빠른 의사 결정
- 더 쉬운 디버깅

필수 도구로 시작하여 필요시 추가합니다.

### 명시적 도구 제약

제한을 명확히 합니다:
```python
"도구 X는 미국 위치에서만 작동"
"도구 Y는 일일 100개 요청 제한"
```

### 도구 사용 모니터링

호출되는 도구 로깅:
- 모델 동작 이해
- 오용 식별 (잘못된 도구 선택)
- 환각 감지

### 우아한 저하

도구를 사용할 수 없어도 모델이 작동:
- 도구 타임아웃 → 부분 답변 반환
- 도구 실패 → 대체 접근 시도
- 네트워크 없음 → 캐시된 지식 사용

## 제한 및 도전

**정확성**: LLM은 존재하지 않는 도구를 호출하거나 잘못된 파라미터를 사용 가능

**지연**: 도구 호출은 네트워크 지연 추가 (호출당 100ms-1s)

**비용**: 도구 호출은 토큰을 소비하고 규모에서 비쌈

**보안**: 모든 도구 호출을 검증하고 무단 작업 방지 필요

**복잡성**: 각 도구는 잠재적 실패 지점 추가

## 미래 방향

- **자동 도구 발견**: 모델이 자동으로 사용 가능한 도구 발견
- **도구 학습**: 모델이 경험에서 시간이 지남에 따라 도구 사용 개선
- **도구 컴포지션**: 간단한 도구에서 복잡한 워크플로우
- **멀티모달 도구**: 이미지, 오디오를 소비/생산하는 도구
- **도구 특화 모델**: 특정 도구 사용에 최적화된 미세 조정 모델

## 결론

도구 사용은 LLM을 텍스트 생성기에서 실제 시스템과 상호 작용할 수 있는 [에이전트](/agent-kor.html)로 변환합니다. 효과적인 도구 사용은 명확한 API 설계, 견고한 오류 처리, 신중한 보안 경계가 필요합니다. 도구 생태계가 성숙해지면서 에이전트 시스템은 점점 더 강력하고 실질적이 됩니다.
