---
title: "LangChain Model 최적화 - Cache, Serialization & Token Usage"
author: cotes
categories: [ai, llm]
tags: [langchain, cache, optimization, token, serialization, performance]
toc: true
toc_sticky: true
toc_label: "핵심 요약"
math: false
mermaid: false
pin: false
render_with_liquid: false
---

## 개요

LangChain 애플리케이션을 운영할 때 성능과 비용 최적화는 매우 중요합니다. 이번 포스트에서는 캐싱을 통한 API 호출 최적화, 모델 직렬화, 그리고 토큰 사용량 추적 방법에 대해 자세히 알아보겠습니다.

## 캐싱 (Caching)

### 캐싱이 필요한 이유

LangChain의 캐싱 기능은 두 가지 주요 이점을 제공합니다:

1. **비용 절감**: 동일한 요청에 대해 API 재호출을 방지
2. **성능 향상**: 응답 속도 대폭 개선

### 메모리 캐시 (In-Memory Cache)

가장 간단한 형태의 캐싱입니다:

```python
from langchain.globals import set_llm_cache
from langchain.cache import InMemoryCache
from langchain_openai import ChatOpenAI

# 메모리 캐시 설정
set_llm_cache(InMemoryCache())

# 모델 초기화
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# 첫 번째 호출 (실제 API 호출)
print("첫 번째 호출:")
response1 = llm.invoke("Python의 장점을 3가지 알려줘")
print(response1.content)

# 두 번째 호출 (캐시에서 반환)
print("\n두 번째 호출 (캐시):")
response2 = llm.invoke("Python의 장점을 3가지 알려줘")
print(response2.content)
```

### SQLite 캐시

지속적인 캐싱을 위해 SQLite 데이터베이스를 사용할 수 있습니다:

```python
from langchain.cache import SQLiteCache
import os

# SQLite 캐시 설정
cache_db_path = "langchain_cache.db"
set_llm_cache(SQLiteCache(database_path=cache_db_path))

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

# 캐시가 파일에 저장되므로 프로그램 재시작 후에도 유지됩니다
response = llm.invoke("머신러닝과 딥러닝의 차이점은?")
print(response.content)
```

### Redis 캐시

분산 환경에서는 Redis 캐시를 사용할 수 있습니다:

```python
from langchain.cache import RedisCache
import redis

# Redis 캐시 설정 (Redis 서버가 실행 중이어야 함)
try:
    redis_client = redis.Redis(host='localhost', port=6379, db=0)
    set_llm_cache(RedisCache(redis_client))
    print("Redis 캐시가 설정되었습니다.")
except Exception as e:
    print(f"Redis 연결 실패: {e}")
    # 대안으로 메모리 캐시 사용
    set_llm_cache(InMemoryCache())
```

### 의미론적 캐시 (Semantic Cache)

단순 문자열 매칭이 아닌 의미론적 유사성을 기반으로 한 캐싱입니다:

```python
from langchain.cache import RedisSemanticCache
from langchain_openai import OpenAIEmbeddings

# 의미론적 캐시 설정
try:
    redis_client = redis.Redis(host='localhost', port=6379, db=0)
    embeddings = OpenAIEmbeddings()
    
    set_llm_cache(RedisSemanticCache(
        redis_url="redis://localhost:6379",
        embedding=embeddings,
        score_threshold=0.2  # 유사도 임계값
    ))
    
    # 유사한 질문들이 캐시로 처리됩니다
    response1 = llm.invoke("Python이 뭐야?")
    response2 = llm.invoke("파이썬에 대해 설명해줘")  # 캐시에서 반환될 가능성
    
except Exception as e:
    print(f"의미론적 캐시 설정 실패: {e}")
```

## 모델 직렬화 (Model Serialization)

### 모델 저장하기

LangChain 모델을 파일로 저장하여 재사용할 수 있습니다:

```python
from langchain_openai import ChatOpenAI
import json
import pickle

# 모델 설정
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    max_tokens=1000
)

# JSON으로 모델 설정 저장
model_config = {
    "model_name": llm.model_name,
    "temperature": llm.temperature,
    "max_tokens": llm.max_tokens,
    "model_kwargs": llm.model_kwargs
}

with open("model_config.json", "w") as f:
    json.dump(model_config, f, indent=2)

print("모델 설정이 저장되었습니다.")
```

### 모델 불러오기

저장된 설정으로 모델을 복원합니다:

```python
def load_model_from_config(config_path):
    with open(config_path, "r") as f:
        config = json.load(f)
    
    return ChatOpenAI(
        model=config["model_name"],
        temperature=config["temperature"],
        max_tokens=config["max_tokens"],
        **config.get("model_kwargs", {})
    )

# 모델 복원
loaded_llm = load_model_from_config("model_config.json")
print("모델이 복원되었습니다.")

# 테스트
response = loaded_llm.invoke("안녕하세요!")
print(response.content)
```

### 체인 직렬화

복잡한 체인도 직렬화할 수 있습니다:

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 체인 생성
prompt = ChatPromptTemplate.from_template("다음 주제에 대해 간단히 설명해줘: {topic}")
output_parser = StrOutputParser()
chain = prompt | llm | output_parser

# 체인 설정 저장
chain_config = {
    "prompt_template": prompt.template,
    "model_config": model_config,
    "output_parser": "StrOutputParser"
}

with open("chain_config.json", "w") as f:
    json.dump(chain_config, f, indent=2)
```

## 토큰 사용량 추적 (Token Usage)

### 기본 토큰 추적

토큰 사용량을 모니터링하여 비용을 관리할 수 있습니다:

```python
from langchain.callbacks import get_openai_callback
from langchain_core.messages import HumanMessage

def track_token_usage(llm, messages):
    with get_openai_callback() as cb:
        response = llm.invoke(messages)
        
        print(f"총 토큰: {cb.total_tokens}")
        print(f"프롬프트 토큰: {cb.prompt_tokens}")
        print(f"완료 토큰: {cb.completion_tokens}")
        print(f"총 비용: ${cb.total_cost:.4f}")
        
    return response

# 사용 예시
messages = [HumanMessage(content="인공지능의 미래에 대해 500자 정도로 설명해줘.")]
response = track_token_usage(llm, messages)
print(f"\n응답: {response.content}")
```

### 배치 처리 토큰 추적

여러 요청을 배치로 처리할 때의 토큰 사용량 추적:

```python
def batch_process_with_tracking(llm, message_batches):
    total_cost = 0
    total_tokens = 0
    results = []
    
    for i, messages in enumerate(message_batches):
        with get_openai_callback() as cb:
            response = llm.invoke(messages)
            
            batch_info = {
                "batch_id": i,
                "response": response.content,
                "tokens": cb.total_tokens,
                "cost": cb.total_cost
            }
            
            results.append(batch_info)
            total_cost += cb.total_cost
            total_tokens += cb.total_tokens
            
            print(f"배치 {i}: {cb.total_tokens} 토큰, ${cb.total_cost:.4f}")
    
    print(f"\n전체 통계:")
    print(f"총 토큰: {total_tokens}")
    print(f"총 비용: ${total_cost:.4f}")
    
    return results

# 배치 처리 예시
message_batches = [
    [HumanMessage(content="Python의 장점은?")],
    [HumanMessage(content="JavaScript의 특징은?")],
    [HumanMessage(content="SQL과 NoSQL의 차이점은?")]
]

results = batch_process_with_tracking(llm, message_batches)
```

### 토큰 제한 구현

토큰 사용량을 제한하는 안전장치를 구현할 수 있습니다:

```python
class TokenLimitedChatOpenAI:
    def __init__(self, llm, daily_token_limit=10000):
        self.llm = llm
        self.daily_token_limit = daily_token_limit
        self.daily_usage = 0
        self.reset_date = None
        
    def _check_and_reset_daily_usage(self):
        from datetime import date
        today = date.today()
        
        if self.reset_date != today:
            self.daily_usage = 0
            self.reset_date = today
    
    def invoke(self, messages):
        self._check_and_reset_daily_usage()
        
        if self.daily_usage >= self.daily_token_limit:
            raise Exception(f"일일 토큰 한도 초과: {self.daily_usage}/{self.daily_token_limit}")
        
        with get_openai_callback() as cb:
            response = self.llm.invoke(messages)
            self.daily_usage += cb.total_tokens
            
            print(f"사용된 토큰: {cb.total_tokens}, 일일 사용량: {self.daily_usage}/{self.daily_token_limit}")
            
        return response

# 토큰 제한된 모델 사용
limited_llm = TokenLimitedChatOpenAI(llm, daily_token_limit=5000)

try:
    response = limited_llm.invoke([HumanMessage(content="안녕하세요!")])
    print(response.content)
except Exception as e:
    print(f"에러: {e}")
```

## 성능 모니터링

### 응답 시간 측정

```python
import time
from datetime import datetime

class PerformanceMonitor:
    def __init__(self):
        self.metrics = []
    
    def monitor_call(self, llm, messages, call_id=None):
        start_time = time.time()
        start_datetime = datetime.now()
        
        with get_openai_callback() as cb:
            response = llm.invoke(messages)
            
        end_time = time.time()
        response_time = end_time - start_time
        
        metric = {
            "call_id": call_id or len(self.metrics),
            "timestamp": start_datetime.isoformat(),
            "response_time": response_time,
            "tokens": cb.total_tokens,
            "cost": cb.total_cost,
            "tokens_per_second": cb.total_tokens / response_time if response_time > 0 else 0
        }
        
        self.metrics.append(metric)
        
        print(f"호출 {metric['call_id']}: {response_time:.2f}초, {cb.total_tokens} 토큰")
        
        return response
    
    def get_stats(self):
        if not self.metrics:
            return "데이터가 없습니다."
        
        avg_response_time = sum(m["response_time"] for m in self.metrics) / len(self.metrics)
        total_cost = sum(m["cost"] for m in self.metrics)
        total_tokens = sum(m["tokens"] for m in self.metrics)
        
        return {
            "총 호출 수": len(self.metrics),
            "평균 응답 시간": f"{avg_response_time:.2f}초",
            "총 비용": f"${total_cost:.4f}",
            "총 토큰": total_tokens
        }

# 성능 모니터링 사용
monitor = PerformanceMonitor()

# 여러 호출 모니터링
test_messages = [
    [HumanMessage(content="안녕하세요!")],
    [HumanMessage(content="Python의 특징을 알려주세요.")],
    [HumanMessage(content="머신러닝이 무엇인가요?")]
]

for i, messages in enumerate(test_messages):
    monitor.monitor_call(llm, messages, f"test_{i}")

# 통계 출력
stats = monitor.get_stats()
for key, value in stats.items():
    print(f"{key}: {value}")
```

## 실전 최적화 전략

### 1. 적응형 캐싱 전략

```python
class AdaptiveCachingStrategy:
    def __init__(self, llm):
        self.llm = llm
        self.cache_hit_rate = 0
        self.total_calls = 0
        self.cache_hits = 0
        
    def invoke_with_adaptive_caching(self, messages, use_cache=True):
        self.total_calls += 1
        
        if use_cache and self.cache_hit_rate > 0.3:  # 30% 이상 캐시 히트율
            # 강화된 캐싱 사용
            set_llm_cache(InMemoryCache())
        else:
            # 캐싱 비활성화
            set_llm_cache(None)
        
        response = self.llm.invoke(messages)
        
        # 캐시 히트율 업데이트 (실제로는 더 정교한 로직 필요)
        if use_cache:
            self.cache_hits += 1
            self.cache_hit_rate = self.cache_hits / self.total_calls
        
        return response
```

### 2. 비용 최적화 팁

```python
def optimize_model_selection(task_complexity, budget_limit):
    """
    작업 복잡도와 예산에 따라 최적 모델 선택
    """
    if budget_limit < 0.01:  # 매우 낮은 예산
        return ChatOpenAI(model="gpt-3.5-turbo", temperature=0.1)
    elif task_complexity == "simple" and budget_limit < 0.05:
        return ChatOpenAI(model="gpt-4o-mini", temperature=0.3)
    elif task_complexity == "complex":
        return ChatOpenAI(model="gpt-4o", temperature=0.7)
    else:
        return ChatOpenAI(model="gpt-4o-mini", temperature=0.5)

# 사용 예시
simple_model = optimize_model_selection("simple", 0.02)
complex_model = optimize_model_selection("complex", 0.10)
```

## 마무리

이번 포스트에서는 LangChain Model의 성능과 비용을 최적화하는 다양한 방법들을 살펴봤습니다:

### 주요 최적화 기법

1. **캐싱 전략**
   - 메모리 캐시: 단기 세션용
   - SQLite 캐시: 지속적 캐싱
   - Redis 캐시: 분산 환경
   - 의미론적 캐시: 지능형 캐싱

2. **모델 직렬화**
   - 설정 저장 및 복원
   - 체인 직렬화
   - 버전 관리

3. **토큰 사용량 관리**
   - 실시간 추적
   - 배치 처리 최적화
   - 사용량 제한

4. **성능 모니터링**
   - 응답 시간 측정
   - 처리량 분석
   - 비용 추적

### 실무 적용 팁

- **개발 환경**: 메모리 캐시 사용
- **테스트 환경**: SQLite 캐시로 일관성 유지
- **프로덕션 환경**: Redis 캐시로 확장성 확보
- **예산 관리**: 토큰 제한과 모델 선택 자동화

다음 포스트에서는 Google AI 모델과 LangChain을 연동하는 방법에 대해 알아보겠습니다.

## 참고 자료

- [LangChain Caching Documentation](https://python.langchain.com/docs/modules/model_io/llms/llm_caching)
- [OpenAI Pricing](https://openai.com/pricing)
- [Redis Documentation](https://redis.io/documentation)
