---
title: "LangChain 학습 노트 #3: OutputParser로 구조화된 데이터 추출하기"
author: cotes
categories: [ai, llm]
tags: [langchain, llm, ai, openai, outputparser, pydantic, json, data-parsing, python]
toc: true
toc_sticky: true
toc_label: "핵심 요약"
math: false
mermaid: true
pin: false
render_with_liquid: false
---

LLM의 응답을 단순 텍스트가 아닌 구조화된 데이터로 받고 싶다면? LangChain의 OutputParser가 해답입니다. 이메일 내용 분석부터 JSON 데이터 생성까지, 다양한 형태로 데이터를 파싱하는 방법을 알아봅니다.

## 1. OutputParser 개요: 왜 필요한가?

LLM은 기본적으로 텍스트를 반환합니다. 하지만 실제 애플리케이션에서는 구조화된 데이터가 필요한 경우가 많습니다.

### 기본 LLM 응답
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template(
    "다음의 이메일 내용중 중요한 내용을 추출해 주세요.\n\n{email_conversation}"
)

llm = ChatOpenAI(temperature=0, model_name="gpt-4.1-mini")
chain = prompt | llm

# 결과: 단순 텍스트로 반환
```

### OutputParser 사용 후
```python
# 구조화된 객체로 반환
class EmailSummary(BaseModel):
    person: str = Field(description="메일을 보낸 사람")
    email: str = Field(description="메일을 보낸 사람의 이메일 주소")
    subject: str = Field(description="메일 제목")
    summary: str = Field(description="메일 본문을 요약한 텍스트")
    date: str = Field(description="메일 본문에 언급된 미팅 날짜와 시간")
```

## 2. PydanticOutputParser: 타입 안전한 데이터 구조

### 핵심 개념
PydanticOutputParser는 두 가지 주요 메서드를 제공합니다:

- **`get_format_instructions()`**: LLM에게 출력 형식을 지시
- **`parse()`**: 텍스트 응답을 Pydantic 객체로 변환

### 실제 구현 예시

```python
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field

# 1. 데이터 모델 정의
class EmailSummary(BaseModel):
    person: str = Field(description="메일을 보낸 사람")
    email: str = Field(description="메일을 보낸 사람의 이메일 주소")
    subject: str = Field(description="메일 제목")
    summary: str = Field(description="메일 본문을 요약한 텍스트")
    date: str = Field(description="메일 본문에 언급된 미팅 날짜와 시간")

# 2. 파서 생성
parser = PydanticOutputParser(pydantic_object=EmailSummary)

# 3. 프롬프트에 형식 지시 추가
prompt = PromptTemplate.from_template("""
You are a helpful assistant. Please answer the following questions in KOREAN.

QUESTION:
{question}

EMAIL CONVERSATION:
{email_conversation}

FORMAT:
{format}
""")

# format에 파서의 지시사항 추가
prompt = prompt.partial(format=parser.get_format_instructions())

# 4. 체인 구성
chain = prompt | llm | parser
```

### 실행 결과
```python
response = chain.invoke({
    "email_conversation": email_conversation,
    "question": "이메일 내용중 주요 내용을 추출해 주세요.",
})

# EmailSummary 객체로 반환됨
print(response.person)  # "김철수"
print(response.email)   # "chulsoo.kim@bikecorporation.me"
print(response.subject) # "ZENESIS 자전거 유통 협력 및 미팅 일정 제안"
```

## 3. with_structured_output(): 더 간단한 방법

PydanticOutputParser 대신 `with_structured_output()` 메서드를 사용하면 더 간결하게 구현할 수 있습니다.

```python
# 구조화된 출력을 위한 LLM 생성
llm_with_structured = ChatOpenAI(
    temperature=0, 
    model_name="gpt-4.1-mini"
).with_structured_output(EmailSummary)

# 바로 사용 가능
answer = llm_with_structured.invoke(email_conversation)
```

> **주의사항**: `with_structured_output()`은 `stream()` 기능을 지원하지 않습니다.

## 4. 다양한 OutputParser 종류

### CommaSeparatedListOutputParser
쉼표로 구분된 리스트가 필요할 때 사용합니다.

```python
from langchain.output_parsers import CommaSeparatedListOutputParser

parser = CommaSeparatedListOutputParser()

# 결과: ["항목1", "항목2", "항목3"]
```

### JsonOutputParser
JSON 형태의 구조화된 데이터가 필요할 때 사용합니다.

```python
from langchain_core.output_parsers import JsonOutputParser

# 스키마 정의 가능
parser = JsonOutputParser()

# 또는 Pydantic 모델과 함께 사용
parser = JsonOutputParser(pydantic_object=MyDataModel)
```

### 기타 전문 파서들
- **PandasDataFrameOutputParser**: DataFrame 형태로 변환
- **DatetimeOutputParser**: 날짜/시간 데이터 파싱
- **EnumOutputParser**: 열거형 값 파싱
- **OutputFixingParser**: 파싱 오류 자동 수정

## 5. 실무 활용 팁

### Field Description 최적화
```python
class ProductInfo(BaseModel):
    name: str = Field(description="제품명 (브랜드명 포함)")
    price: float = Field(description="가격 (숫자만, 단위 제외)")
    category: str = Field(description="제품 카테고리 (전자제품, 의류, 식품 등)")
    features: list[str] = Field(description="주요 특징들의 리스트")
```

- Description은 LLM이 정확한 정보를 추출할 수 있도록 **구체적이고 명확**하게 작성
- 데이터 형식이나 제약사항을 명시

### 에러 처리
```python
try:
    result = parser.parse(llm_output)
except ValidationError as e:
    print(f"파싱 에러: {e}")
    # 에러 처리 로직
```

### 성능 고려사항
- 복잡한 스키마일수록 더 큰 모델 필요 (예: GPT-4 vs GPT-3.5)
- Field 개수가 많으면 파싱 정확도 감소 가능성
- 중요한 필드는 앞쪽에 배치

## 6. 마무리

OutputParser는 LLM의 자유로운 텍스트 생성과 애플리케이션의 구조화된 데이터 요구사항을 연결하는 핵심 도구입니다. 

### 핵심 포인트
- **PydanticOutputParser**: 타입 안전성과 검증 기능 제공
- **with_structured_output()**: 간단한 사용법, 단 스트리밍 불가
- **Field Description**: 정확한 파싱을 위한 핵심 요소
- **다양한 전문 파서**: 용도에 맞는 파서 선택

## 참고 자료
- [LangChain OutputParser 공식 문서](https://python.langchain.com/docs/modules/model_io/output_parsers/)
- [Pydantic 공식 문서](https://docs.pydantic.dev/latest/)
