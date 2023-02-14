---
title: 하이퍼레저 비수(이더리움 기반) 구축(3)
author: cotes
categories: [blockchain, ethereum]
tags: [blockchain, hyperledger besu, ethereum]
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true
pin: true
render_with_liquid: false
---

## QBFT 네트워크 기반 하이퍼레저 비수 구축
**부트노드값 확인 후 저장**
```console
besu --data-path=/QBFT-Network/Node-1/data --genesis-file=/QBFT-Network/genesis.json --p2p-port=30303 --rpc-http-enabled --rpc-http-api=ETH,NET,QBFT --host-allowlist="*" --rpc-http-cors-origins="all"


besu --data-path=/QBFT-Network/Node-2/data --genesis-file=/QBFT-Network/genesis.json --p2p-port=30304 --rpc-http-enabled --rpc-http-api=ETH,NET,QBFT --host-allowlist="*" --rpc-http-cors-origins="all"
```
![부트노드](/assets/img/blockchain/%EB%B6%80%ED%8A%B8%EB%85%B8%EB%93%9C1.png)

**블록체인 옵션 리스트 및 설정값**[^option]

| 옵션 | 설명 |
| ---- | ---- |
| BESU_IDENTITY | 노드의 이름. 지정된 경우 일부 Ethereum 네트워크 탐색기에서 제공하는 클라이언트ID의 두 번째 섹션 |
| BESU_NETWORK | 구성할 네트워크 옵션 |
| BESU_NETWORK_ID | P2P 네트워크 식별자 옵션 |
| BESU_LOGGING | 로깅 수준 정의하는 옵션 |
| BESU_COLOR_ENABLED | 콘솔에 대한 색상 출력을 활성화 여부 |
| BESU_SYNC_MODE | 피어들 간의 동기화 모드 옵션 |
| BESU_GENESIS_FILE | 제네시스 파일을 사용하여 네트워크 구성 |
| BESU_BOOTNODES | 부트 노드로 사용할 피어 설정 옵션 |
| BESU_RPC_HTTP_ENABLED | HTTP JSON-RPC API 서비스 사용 여부 옵션 |
| BESU_RPC_HTTP_HOST | HTTP JSON-RPC API 수신 대기하는 HOST IP |
| BESU_RPC_HTTP_PORT | HTTP JSON-RPC 수신 포트(TCP) |
| BESU_RPC_HTTP_API | 블록체인의 기능을 제공하는 옵션으로 HTTP JSON-RPC API를 통해서 설정한 기능들을 사용할 수 있다 |
| BESU_HOST_ALLOWLIST | JSON-RPC API에 접근 또는 요청할 수 있도록 설정하는 옵션 |
| BESU_RPC_HTTP_CORS_ORIGINS | CORS 유효성 검사를 위한 도메인 URL 목록이며, 설정된 도메인만 JSON-RPC를 사용하여 노드에 접근할 수 있다. |
| BESU_RPC_TX_FEECAP | JSON-RPC API인 eth_sendRawTransaction를 통해서 invoke할 경우 트랜잭션 수수료가 발생하는데 거기에 대한 기본 수수료 설정 옵션이며, 0으로 설정 시 옵션 무시하여 상한선이 적용되지 않음. |
| BESU_RPC_HTTP_MAX_ACTIVE_CONNECTIONS | 허용 되는 최대 HTTP JSON-RPC 연결 수 해당 수 이상 도달하면 들어오는 연결이 거부당함 |
| BESU_P2P_ENABLED | 모든 P2P 통신을 활성화 여부 옵션 |
| BESU_P2P_HOST | P2P 통신할 수 있는 외부에서 노드에 접근할 수 있는 범위의 Host IP |
| BESU_P2P_PORT | P2P 수신 포트(UDP 및 TCP) |
| BESU_P2P_INTERFACE | 실행중인 export Besu가 특정 네트워크 인터페이스에 바인딩할 경우 지정 |
| BESU_MIN_GAS_PRICE | 거래(트랜잭션)가 채굴된 블록에 포함시키기 위해 제안되는 최소 가격 |
| BESU_MINER_ENABLED | 노드 구성 시 채굴 활성화 여부 |
| BESU_MINER_COINBASE | 채굴 성공 시 채굴 보상을 지급 받는 계정 |

**블록체인 옵션 파일 생성**
```console
mkdir core && cd core && vi node-core
```
```bash
export BESU_IDENTITY=${HOYA_PEER_NAME}

export BESU_NETWORK=MAINNET

export BESU_NETWORK_ID=20221026

export BESU_LOGGING=${HOYA_LOGGING}

export BESU_COLOR_ENABLED=${HOYA_BESU_COLOR_ENABLED}

export BESU_SYNC_MODE=FULL

# 노드1 Node-1, 노드2 Node-2, 노드3 Node-3, 노드4 Node-4 각자 경로에 맞게 설정
export BESU_DATA_PATH=/QBFT-Network/${HOYA_NODE}/data

export BESU_GENESIS_FILE=/QBFT-Network/genesis.json

# 위에서 로그에 찍혔던 부트노드값들 여기에 설정 !!
# 127.0.0.1 -> dns값인 peer1 또는 peer2로 설정
export BESU_BOOTNODES=enode://af3697380bce736e3c4ad3e6be5ea572cce89c494f95ab057dc90afb41d93234e8104029b24e2b5ac40ffae6253381903d8cd53787190a28dd76f58b9a689c9f@node1:30303,enode://bd2dcdac0a017e7189aaee61ad8d10266394a7757dad2ebba10fd29c8bbd9a07ad83c3ecc848d0128f240e55d10acfa6d7d948f0c94dcb1cdc7a6d63b76032af@node2:30304

export BESU_RPC_HTTP_ENABLED=true

export BESU_RPC_HTTP_HOST=0.0.0.0

# 노드1 8545, 노드2 8546, 노드3 8547, 노드4 8548
export BESU_RPC_HTTP_PORT=${HOYA_HTTP_PORT}

export BESU_RPC_HTTP_API=ETH,QBFT,NET,WEB3

export BESU_HOST_ALLOWLIST="*"

export BESU_RPC_HTTP_CORS_ORIGINS=all

export BESU_RPC_TX_FEECAP=0
 
export BESU_RPC_HTTP_MAX_ACTIVE_CONNECTIONS=500000

export BESU_P2P_ENABLED=true 

export BESU_P2P_HOST=0.0.0.0

# 노드1 30303, 노드2 30304, 노드3 30305, 노드4 30306
export BESU_P2P_PORT=${HOYA_P2P_PORT}

export BESU_P2P_INTERFACE=0.0.0.0

export BESU_MIN_GAS_PRICE=0

export BESU_MINER_ENABLED=true

# 블로그에선 노드1 계정 설정하도록 진행
export BESU_MINER_COINBASE=11549197c99e4b886ed7d6ed2843f534a4f367fc
```

**entrypoint 파일 생성**
```console
cd / && vi entrypoint.sh
```
```bash
#!/bin/bash
set -e

source /QBFT-Network/core/node-core
/usr/local/bin/besu-22.10.0/bin/besu --Xdns-enabled=true --Xdns-update-enabled=true

exec "$@"
```

이제 블록체인 구동에 필요한 설정은 다 끝났다. 하이퍼레저 비수(이더리움 기반) 구축(4)에서 도커를 이용하여 4개의 노드를 구동해볼 것 이다.
**최종 디렉토리 구조**는 아래와 같이 나와야한다.
!["최종디렉토리구조"](/assets/img/blockchain/%EC%B5%9C%EC%A2%85%EB%94%94%EB%A0%89%ED%86%A0%EB%A6%AC%EA%B5%AC%EC%A1%B0.png)  

## 참조  
[^option]: [하이퍼레저 비수 옵션](https://besu.hyperledger.org/en/stable/public-networks/reference/cli/options/)  
