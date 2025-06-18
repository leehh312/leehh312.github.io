---
title: "Progressive Delivery 이해하기: Flagger로 배우는 배포 전략 이론편"
author: cotes
categories: [devops, strategy]
tags: [devops, deploy, strategy, progressive-delivery, flagger, canary, blue-green, kubernetes]
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true
pin: false
render_with_liquid: false
---

빠르게 변화하는 현대 IT 환경에서 서비스 배포는 단순히 새로운 코드를 반영하는 수준을 넘어섰습니다. 사용자 수가 많아지고 인프라가 복잡해질수록, 한 번의 잘못된 배포가 심각한 장애로 이어질 수 있기 때문이죠. 특히 마이크로서비스 아키텍처와 클라우드 네이티브 환경에서는 더 자주, 더 안전하게 배포할 수 있는 체계가 필수적입니다.

이러한 요구에 부응해 등장한 개념이 바로 **Progressive Delivery**입니다. 롤링 업데이트에서 한 단계 발전한 이 전략은, 배포를 점진적으로 진행하며 실시간으로 안정성을 검증합니다. 문제가 발생하면 즉시 대응할 수 있도록 설계되어 개발 속도는 높이고 장애 위험은 낮추는, 두 마리 토끼를 모두 잡는 방식이죠. 이미 많은 글로벌 기업들의 표준으로 자리 잡았으며, Kubernetes 기반 시스템에서 활발히 적용되고 있습니다.

---

### **왜 Flagger 인가?**

Progressive Delivery를 구현하는 도구로는 **Argo Rollouts**도 널리 사용됩니다. Argo Rollouts는 Kubernetes 네이티브 환경에서 강력한 UI와 GitOps 연동을 통해 직관적인 배포 흐름을 제공하는 훌륭한 도구입니다.

하지만 이 글에서 Flagger를 선택한 이유는 다음과 같습니다.

* GitOps 철학을 공유하는 **Flux와의 완벽한 통합**
* Canary, Blue/Green 배포를 위한 **간결하고 정교한 정책 기반 제어**
* Prometheus, Grafana 등 **모니터링 도구와의 뛰어난 연동성** 및 유연한 자동화
* 다양한 서비스 메시 및 Ingress Controller 지원과 풍부한 운영 사례

Flagger는 비교적 간단한 설정으로 안정적이고 확장성 있는 Progressive Delivery 전략을 구현할 수 있어, 이제 막 도입을 고민하는 팀에게 훌륭한 출발점이 될 수 있습니다.

이제 본격적으로 Progressive Delivery의 개념부터 주요 배포 전략, 그리고 Flagger 설정 예시까지 하나씩 살펴보겠습니다.


## **Progressive Delivery란?**
Progressive Delivery는 새 버전을 전체 사용자에게 한 번에 배포하는 대신, 일부 트래픽만 점진적으로 전환하며 배포하는 전략입니다. 이를 통해 실시간 모니터링을 기반으로 문제 발생 시 신속히 롤백할 수 있어,  서비스의 안정성을 높이고 배포 리스크를 최소화할 수 있습니다.


### 핵심 특징

1. 점진적인 배포

   새로운 애플리케이션 버전을 한 번에 배포하는 것이 아니라, 점진적으로 배포 방식입니다. 이러한 배포 방식을 통해 문제가 발생할 경우 전체 사용자에게 영향을 미치지 않고 빠르게 수정하거나 롤백 가능


2. 자동화된 배포 및 모니터링

   배포 자동화 도구를 사용하여 애플리케이션을 점진적으로 배포하고 배포된 새 버전의 성능 지표와 사용자 피드백을 실시간으로 모니터링을 통해서 배포된 새 버전이 성능이나 안정성 면에서 문제가 없다고 판단되면, 점진적으로 더 많은 트래픽을 새 버전으로 전환


3. 피드백 기반 롤백

   배포 중에 발생하는 실시간 데이터를 기반으로 자동 롤백이 가능


4. 배포 전략 다양성

   여러 배포 전략을 지원하며, Canary 배포, Blue/Green 배포, A/B 테스트, 기능 플래그(Feature Flags) 등의 다양한 전략을 사용하여 배포를 진행


### 주요 도구
* [Flagger](https://flagger.app/): GitOps 기반 자동화 배포 컨트롤러
* [Argo Rollouts](https://argoproj.github.io/rollouts/): Kubernetes 네이티브 롤링 배포 확장 기능
* [LaunchDarkly](https://launchdarkly.com/): 기능 플래그 중심의 Progressive Delivery 플랫폼


## **주요 Progressive Delivery 전략 비교**
### **Canary 배포**
Canary 배포는 Progressive Delivery의 가장 널리 사용되는 방식 중 하나이며, 이 Canary는 새로운 애플리케이션 버전을 전체 트래픽의 일부(예 5% 또는 10%)만 배포하고 메트릭을 통해서 성능 및 안정성을 모니터링하고 있다가 문제가 없으면 점진적으로 더 많은 트래픽을 새로운 버전으로 전환하는 전략입니다.

 ![Canary](/assets/img/devops/Canary.webp)

 ![https://docs.flagger.app/tutorials/nginx-progressive-delivery 참고](/assets/img/devops/https___docs.flagger.app_tutorials_nginx-progressive-delivery%20참고.png)

### Canary 동작원리

1. 시작 - Canary 배포 준비

   이 시점에서는 모든 트래픽이 v1으로만 전송되고 있으며, v2는 아직 트래픽을 받지 않고 있는 상태입니다.
2. Canary 배포 시작 (5% 트래픽 전송)

   새 버전 v2가 배포되고 5%의 트래픽이 v2로 전송되고 나머지 95%의 트래픽은 여전히 v1으로 전송 되고, 이 단계에서는 v2에 소량의 트래픽을 보내 성능이나 안정성 문제를 확인하기 위한 단계입니다.
3.  Canary 트래픽 증가 (10% \~ 50% 트래픽)

   트래픽이 10% \~ 50%까지 점진적으로 v2로 전환되면서 메트릭 수집해서 v2가 안정적으로 동작하는지 평가하는 단계이며, 즉 v2는 안정적인 새 버전으로 프로덕션 환경에 완전히 배포 되는것을 의미합니다.
4. Canary 트래픽 증가 (50% 트래픽)

   v2가 실행되는 동안 메트릭을 수집합니다. 메트릭은 주로 애플리케이션의 상태(예: CPU 사용량, 메모리 사용량, 오류율, 성공 요청 비율 등)를 평가하고 수집된 메트릭은 Prometheus 등의 모니터링 도구를 통해 분석합니다.
5. 트래픽 100% 전환

   모든 메트릭 분석이 완료되고 문제가 발견되지 않으면 v2로 100% 트래픽이 전환되면서 안정적인 새 버전으로 프로덕션 환경에 완전히 배포 됩니다.
6. v1 제거 및 배포 완료

   v2가 프로덕션 버전으로 완전히 승격되고 더 이상 v1은 사용되지 않기 때문에 제거 됩니다.


### Blue/Green 배포
Blue/Green 배포는 두 개의 환경을 사용하여 새로운 애플리케이션 버전을 배포하는 방식입니다. 예를들어 Blue는 현재 애플리케이션이 실행 중인 버전이고, Green은 애플리케이션 새 버전이였을 때 Green에 배포가 완료되면 트래픽을 Blue에서 Green으로 한 번에 전환하는 전략입니다.

 ![Blue/Green](/assets/img/devops/Blue_Green.png)

 ![https://docs.flagger.app/tutorials/kubernetes-blue-green 참고](/assets/img/devops/https___docs.flagger.app_tutorials_kubernetes-blue-green%20참고.png)

### Blue/Green 동작원리

1. 시작 - 새 버전 배포 감지 (Detect new version)

   새로운 버전의 배포가 감지되면, v2를 배포하여 v1과 함께 실행되고 이 시점에는 v2로 트래픽이 전달되지 않습니다.
2. Conformance 테스트

   애플리케이션의 기본적인 동작을 확인하기 위한 것으로, 애플리케이션이 정상적으로 실행되고 있는지 확인하는 절차를 거칩니다.
3. 로드 테스트 (Load Tests)

   실제 프로덕션 환경에서 발생할 수 있는 다양한 부하를 시뮬레이션하여, v2가 높은 트래픽이나 요청을 처리할 수 있는지 검증하는 단계 입니다.
4. 메트릭 수집 (Collect Metrics)

   v2가 실행되는 동안 메트릭을 수집합니다. 메트릭은 주로 애플리케이션의 상태(예: CPU 사용량, 메모리 사용량, 오류율, 성공 요청 비율 등)를 평가하고 수집된 메트릭은 Prometheus 등의 모니터링 도구를 통해 분석 합니다.
5. Validate SLOs(SLO - 서비스 수준 목표 검증)

   SLO는 성공률, 응답 시간, 오류율 등의 메트릭이 설정한 임계값 내에 있는지 확인하는 단계를 말하고, 즉 애플리케이션의 성능이 서비스 수준 목표(SLO, Service Level Objectives)를 충족하는지 검증하는 단계 입니다.
6. v2 승격 또는 롤백

   검증이 완료되면, v2를 승격할지 롤백할지 결정
   * 승격: 모든 테스트를 통과하고 SLO를 충족하면 v2가 승격되어 모든 트래픽이 v2로 전환 합니다.
   * 롤백: 만약 SLO 검증에 실패하거나 설정한 메트릭의 임계값 또는 기준이 벗어나면 v2는 롤백되고 기존 버전 v1이 계속 프로덕션 환경에서 사용 합니다.


## Flagger란?
Flagger는 [Flux](https://fluxcd.io/)의 일부로, GitOps 기반 배포 자동화 및 트래픽 제어를 제공하는 컨트롤러입니다. Service Mesh(Istio, Linkerd) 또는 Ingress Controller(Nginx, Traefik)와 연동하여 Canary 및 Blue/Green 배포를 제어합니다.


### Flagger의 핵심 기능
* 트래픽 점진 전환 (stepWeights)
* 메트릭 기반 자동 롤백
* Slack, Webhook 기반 피드백 루프
* Prometheus, Grafana 등과 통합



> [!NOTE]
> Flagger 1.41.0 버전 기준으로 작성되었습니다.
### Canary 배포 전략
Canary 배포는 점진적으로 새 버전에 트래픽을 보내면서 성능과 안정성을 검증하는 방식입니다. Flagger에서는 Canary 배포 시간과 롤백 시간 계산이 다음과 같이 이루어집니다

* 전체 배포 시간 계산 공식:

```bash
interval * (maxWeight / stepWeight)
```

* 롤백 소요 시간 계산 공식:

```bash
interval * threshold
```

* Canary 배포 예시 옵션:

```bash
  analysis:
    # 테스트를 주기적으로 실행하는 간격 (여기서는 1분마다 테스트 수행)
    interval: 1m
    # 버전으로 전달되는 트래픽을 한 번에 증가시키는 비율을 설정
    stepWeight: 20
    # 버전으로 전환될 수 있는 트래픽의 최대 가중치를 설정
    maxWeight: 100
    # 배포가 실패로 간주될 수 있는 임계값을 설정 함. 보통 메트릭을 기반으로 한 임계값
    threshold: 10
    
    # stepWeight & maxWeight or stepWeights 2가지 방식 제공
    # canary(v2): 10 -> 25 -> 50 -> 90 -> promote
    # primary(v1): 90 -> 75 -> 50 -> 10 -> promote
    stepWeights: [10, 25, 50, 90]
```


### Blue/Green 배포 전략
Blue/Green 배포는 새 버전(Green)을 기존 버전(Blue)과 함께 띄우고, 테스트 완료 후 트래픽을 한 번에 전환하는 방식입니다.

* 전체 테스트 소요 시간 계산 공식:

```bash
interval * iterations
```

* 롤백 소요 시간 계산 공식:

```bash
interval * threshold
```

* Blue/Green 배포 예시 옵션:

```bash
 analysis:
    # 테스트를 주기적으로 실행하는 간격 (여기서는 1분마다 테스트 수행)
    interval: 1m
    # 총 반복 횟수
    iterations: 10
    # 배포가 실패로 간주될 수 있는 임계값을 설정 함. 보통 메트릭을 기반으로 한 임계값
    threshold: 2
```


### 공통 설정 예시
Flagger는 Canary와 Blue/Green 배포 모두에 대해 다음과 같은 공통 설정을 지원합니다:

```bash
  # 라우팅 공급자 이름 (예: nginx, istio, linkerd 등)
  provider: nginx
  # 대상 리소스를 참조 (Canary 대상이 될 Deployment 등 설정)
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hello-v1
    
  # 참조할 Ingress 리소스 설정
  ingressRef:
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    name: prod-ingress
    
  # HPA(HorizontalPodAutoscaler)를 참조할 경우 설정
  autoscalerRef:
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    name: podinfo
  
  # 트래픽을 라우팅할 서비스 정의 (Deployment와 연결할 서비스 직접 생성)
  service:
    name: hoya-demo
    port: 5000
    targetPort: 5000
    
  # 배포가 설정된 시간 안에 완료되지 않으면 중단하는 시간 제한 (초 단위, 기본값 600초)
  progressDeadlineSeconds: 60

  # 배포 시 분석 설정
  analysis:
    # 메트릭 기반 상태 분석
    metrics:
      # 1분 간격으로 HTTP 요청 성공률이 95% 이상인지 체크
    - name: request-success-rate
      interval: 1m
      thresholdRange:
        min: 95
      # 1분 간격으로 요청 응답 시간이 500ms 이하인지 체크
    - name: request-duration
      interval: 1m
      thresholdRange:
        max: 500
        
    # 배포 중 실행할 웹훅 테스트 설정
    webhooks:
      - name: load-test
        # Webhook 실행 유형 설정
        # 아래 중 하나를 선택할 수 있습니다:
        # - confirm-rollout: 배포 시작 전 수동 승인 요청
        # - pre-rollout: 트래픽을 카나리로 보내기 전 테스트 실행
        # - rollout: 분석 주기마다 반복 실행 (ex: 부하 테스트 등)
        # - confirm-traffic-increase: 트래픽 증가 전 수동 승인 요청
        # - confirm-promotion: 새 버전 승격 전 수동 승인 요청
        # - post-rollout: 승격 또는 롤백 후 후처리 실행
        # - rollback: 분석 중이나 대기 중일 때 조건부 롤백 허용
        # - event: Flagger 이벤트 발생 시마다 HTTP POST 전송
        type: pre-rollout
        # 실행할 webhook의 URL
        url: http://flagger-loadtester.default/
        # webhook 호출 최대 대기 시간
        timeout: 5s
        # 실행 방식 및 추가 명령어 정보
        metadata:
          type: cmd
          # 부하 테스트 도구 hey를 사용해 60초 동안 초당 10개 요청을,
          # 2개의 클라이언트가 병렬로 Ingress에 요청을 보내도록 설정
          # Host 헤더는 ingress 경로에 맞게 설정
          cmd: "hey -z 60s -q 10 -c 2 -host uracle.local http://ingress-nginx-controller.ingress-nginx"
        
    # Canary 배포의 마지막 단계에서 트래픽을 대폭 증가시켜 빠르게 새로운 버전으로 전환할 수 있도록 하는 옵션
    stepWeightPromotion: 40
    
  # true로 설정하면 메트릭 분석을 건너뛰고
  # 정해진 트래픽 증가 단계만 따라 자동으로 배포 진행됨
  # 성공률, 응답 시간, 오류율 등은 확인하지 않음 (비추천)
  skipAnalysis: false
```


## **Hey 도구 소개**
hey는 Flagger 배포 중 webhook에서 자주 사용되는 Go 기반의 HTTP 부하 테스트 도구이며, 다양한 설정을 통해 특정 웹 서비스에 대량의 요청을 보내 성능을 테스트할 수 있습니다.

주요 옵션은 다음과 같습니다:

* `-z`: 테스트 지속 시간 (예: `-z 60s`는 60초간 테스트)
* `-q`: 초당 요청 수 (예: `-q 10`은 초당 10개의 요청)
* `-c`: 병렬 요청 수 (예: `-c 2`는 동시에 2개의 요청 발생)
* `-host`: 요청의 Host 헤더 지정 (Ingress에서 가상 호스트 사용 시 유용)
* `-n`: 총 요청 수 지정 (지속 시간 대신 총 횟수 기반 테스트)
* `-H`: HTTP 헤더 직접 설정
* `-d`: POST 요청 본문 데이터 지정
* `-m`: HTTP 메서드 지정 (기본값은 GET)


지금까지 Progressive Delivery의 개념과 주요 배포 전략, 그리고 Flagger를 활용한 실전 설정 방법까지 이론 중심으로 살펴보았습니다. 점진적 배포는 단순한 기술적 선택을 넘어서, 서비스의 안정성과 사용자 경험(UX)을 동시에 지켜낼 수 있는 강력한 무기입니다.

특히 Flagger는 Kubernetes 기반 환경에서 Canary 및 Blue/Green 배포를 손쉽게 구현하고, 실시간 메트릭 기반의 피드백 루프를 통해 배포의 품질과 안정성을 높일 수 있도록 도와줍니다. GitOps 방식에 익숙하지 않은 팀이라도 비교적 간단한 구성으로 시작할 수 있다는 점도 큰 장점입니다.

모든 조직에 복잡하고 정교하게 튜닝된 배포 전략이 필요한 것은 아니지만, 빠르게 변화하는 환경 속에서 장애 없는 안정적인 릴리즈를 원한다면 Progressive Delivery는 더 이상 선택이 아닌 필수가 되어가고 있습니다.