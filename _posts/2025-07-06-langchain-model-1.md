---
title: "LangChain Model 기초 - Chat Models & 기본 설정"
author: cotes
categories: [ai, llm]
tags: [langchain, llm, chatmodel, openai, anthropic, ai]
toc: true
toc_sticky: true
toc_label: "핵심 요약"
math: false
mermaid: false
pin: false
render_with_liquid: false
---

## 개요

LangChain은 다양한 Large Language Model(LLM)과 쉽게 연동할 수 있는 강력한 프레임워크입니다. 이번 포스트에서는 LangChain의 Chat Model 기본 개념과 OpenAI, Anthropic 등 주요 모델들을 활용하는 방법에 대해 살펴보겠습니다.

## 환경 설정

먼저 필요한 패키지들을 설치하고 환경을 설정해보겠습니다.

```bash
pip install langchain langchain-openai langchain-anthropic python-dotenv
```

### API 키 설정

보안을 위해 API 키는 환경변수로 관리하는 것이 좋습니다.

```python
# .env 파일 생성
OPENAI_API_KEY=your_openai_api_key_here
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

```python
from dotenv import load_dotenv
import os

# API KEY 정보 로드
load_dotenv()
```

## OpenAI Chat Model

### 기본 사용법

OpenAI의 GPT 모델을 LangChain에서 사용하는 방법은 매우 간단합니다.

```python
from langchain_openai import ChatOpenAI

# ChatOpenAI 모델 초기화
llm = ChatOpenAI(
    model="gpt-4o-mini",  # 사용할 모델명
    temperature=0.7,      # 창의성 조절 (0.0~2.0)
    max_tokens=1000,      # 최대 토큰 수
)

# 메시지 생성
from langchain_core.messages import HumanMessage

messages = [
    HumanMessage(content="LangChain이 무엇인지 간단히 설명해줘.")
]

# 모델 실행
response = llm.invoke(messages)
print(response.content)
```

### 스트리밍 응답

실시간으로 응답을 받아보고 싶다면 스트리밍을 사용할 수 있습니다.

```python
# 스트리밍 응답
for chunk in llm.stream(messages):
    print(chunk.content, end="", flush=True)
```

### 모델 파라미터 설정

OpenAI Chat Model에서 설정할 수 있는 주요 파라미터들입니다:

```python
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,        # 0.0(결정적) ~ 2.0(창의적)
    max_tokens=1000,        # 응답 최대 길이
    top_p=1.0,             # 누적 확률 임계값
    frequency_penalty=0.0,  # 반복 페널티 (-2.0 ~ 2.0)
    presence_penalty=0.0,   # 새로운 주제 장려 (-2.0 ~ 2.0)
    n=1,                   # 생성할 응답 수
    streaming=False,       # 스트리밍 여부
    verbose=True          # 디버그 정보 출력
)
```

## Anthropic Claude Model

Anthropic의 Claude 모델도 LangChain에서 쉽게 사용할 수 있습니다.

```python
from langchain_anthropic import ChatAnthropic

# Claude 모델 초기화
claude = ChatAnthropic(
    model="claude-3-5-sonnet-20241022",
    temperature=0.7,
    max_tokens=1000,
)

# 메시지 실행
response = claude.invoke(messages)
print(response.content)
```

## 메시지 타입

LangChain에서는 다양한 메시지 타입을 제공합니다:

```python
from langchain_core.messages import (
    HumanMessage,      # 사용자 메시지
    AIMessage,         # AI 응답 메시지
    SystemMessage,     # 시스템 메시지
    FunctionMessage    # 함수 호출 메시지
)

# 시스템 메시지와 함께 사용
messages = [
    SystemMessage(content="당신은 도움이 되는 AI 어시스턴트입니다."),
    HumanMessage(content="Python으로 간단한 계산기를 만드는 방법을 알려줘.")
]

response = llm.invoke(messages)
print(response.content)
```

## 대화 히스토리 관리

연속적인 대화를 위해서는 메시지 히스토리를 관리해야 합니다:

```python
# 대화 히스토리 초기화
conversation_history = [
    SystemMessage(content="당신은 Python 전문가입니다.")
]

def chat_with_ai(user_input):
    # 사용자 메시지 추가
    conversation_history.append(HumanMessage(content=user_input))
    
    # AI 응답 생성
    response = llm.invoke(conversation_history)
    
    # AI 응답을 히스토리에 추가
    conversation_history.append(AIMessage(content=response.content))
    
    return response.content

# 사용 예시
response1 = chat_with_ai("리스트 컴프리헨션이 뭐야?")
print("AI:", response1)

response2 = chat_with_ai("예시 코드를 보여줘")
print("AI:", response2)
```

## 모델 비교 예시

여러 모델의 응답을 비교해보는 유용한 예시입니다:

```python
def compare_models(question):
    models = {
        "GPT-4o-mini": ChatOpenAI(model="gpt-4o-mini", temperature=0.7),
        "Claude-3.5": ChatAnthropic(model="claude-3-5-sonnet-20241022", temperature=0.7)
    }
    
    messages = [HumanMessage(content=question)]
    
    results = {}
    for model_name, model in models.items():
        try:
            response = model.invoke(messages)
            results[model_name] = response.content
        except Exception as e:
            results[model_name] = f"Error: {str(e)}"
    
    return results

# 모델 비교 실행
question = "인공지능의 윤리적 문제점에 대해 설명해줘."
results = compare_models(question)

for model_name, response in results.items():
    print(f"\n=== {model_name} ===")
    print(response)
```

## 에러 처리

실제 애플리케이션에서는 적절한 에러 처리가 중요합니다:

```python
import time
from langchain_core.exceptions import LangChainException

def safe_invoke(llm, messages, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = llm.invoke(messages)
            return response
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {str(e)}")
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)  # 지수 백오프
            else:
                raise e

# 안전한 호출
try:
    response = safe_invoke(llm, messages)
    print(response.content)
except Exception as e:
    print(f"모든 재시도 실패: {str(e)}")
```

## 마무리

이번 포스트에서는 LangChain에서 Chat Model을 사용하는 기본적인 방법들을 살펴봤습니다. 주요 포인트는 다음과 같습니다:

- **환경 설정**: API 키를 안전하게 관리하기
- **모델 초기화**: 다양한 파라미터로 모델 설정하기
- **메시지 타입**: 적절한 메시지 타입 사용하기
- **대화 관리**: 연속적인 대화를 위한 히스토리 관리
- **에러 처리**: 안정적인 애플리케이션을 위한 예외 처리

## 참고 자료

- [LangChain Documentation](https://python.langchain.com/docs/get_started/introduction)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Anthropic Claude Documentation](https://docs.anthropic.com/)
## 마무리

이번 포스트에서는 LangChain에서 Chat Model을 사용하는 기본적인 방법들을 살펴봤습니다. 주요 포인트는 다음과 같습니다:

- **환경 설정**: API 키를 안전하게 관리하기
- **모델 초기화**: 다양한 파라미터로 모델 설정하기
- **메시지 타입**: 적절한 메시지 타입 사용하기
- **대화 관리**: 연속적인 대화를 위한 히스토리 관리
- **에러 처리**: 안정적인 애플리케이션을 위한 예외 처리

다음 포스트에서는 LangChain Model의 성능 최적화를 위한 캐싱, 직렬화, 토큰 사용량 관리에 대해 자세히 알아보겠습니다.

## 참고 자료

- [LangChain Documentation](https://python.langchain.com/docs/get_started/introduction)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Anthropic Claude Documentation](https://docs.anthropic.com/)
