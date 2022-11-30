---
title: 하이퍼레저 베수(이더리움 기반) 구축(4)
author: cotes
categories: [blockchain, ethereum]
tags: [blockchain, hyperledger besu, ethereum, 블록체인, 하이퍼레저베수, 이더리움]
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true
pin: true
render_with_liquid: false
---

## QBFT 네트워크 기반 하이퍼레저 베수 구축
**작업영역 생성**
```console
mkdir besu && cd besu && mkdir info
```

**도커(컨테이너 -> 로컬)파일 가져오기**
```console
docker cp {containerID}:/QBFT-Network ./
```

**도커 이미지 업데이트**
```console
docker stop {containerID} && docker commit {containerID} ms_peer:hoya
```

**도커 환경변수 설정**
```console
vi .env
```
```bash
# common settins
HOYA_NODE_IMAGE="ms_peer:hoya"
HOYA_COMMAND="/entrypoint.sh"
HOYA_LOGGING="INFO"
HOYA_BESU_COLOR_ENABLED=true

# node1 settings
NODE1_HOYA_PEER_NAME="node1"
NODE1_HOYA_NODE="Node-1"
NODE1_HOYA_HTTP_PORT=8545
NODE1_HOYA_P2P_PORT=30303

# node2 settings
NODE2_HOYA_PEER_NAME="node2"
NODE2_HOYA_NODE="Node-2"
NODE2_HOYA_HTTP_PORT=8546
NODE2_HOYA_P2P_PORT=30304

# node3 settings
NODE3_HOYA_PEER_NAME="node3"
NODE3_HOYA_NODE="Node-3"
NODE3_HOYA_HTTP_PORT=8547
NODE3_HOYA_P2P_PORT=30305

# node4 settings
NODE4_HOYA_PEER_NAME="node4"
NODE4_HOYA_NODE="Node-4"
NODE4_HOYA_HTTP_PORT=8548
NODE4_HOYA_P2P_PORT=30306
```

**docker-compose.yml 구성**
```console
vi docker-compose.yml
```
```yaml
version: "3"
services:
  node1:  
    container_name: ${NODE1_HOYA_PEER_NAME}
    image: ${HOYA_NODE_IMAGE}
    environment:
      HOYA_PEER_NAME: ${NODE1_HOYA_PEER_NAME}
      HOYA_LOGGING: ${HOYA_LOGGING}
      HOYA_BESU_COLOR_ENABLED: ${HOYA_BESU_COLOR_ENABLED}
      HOYA_NODE: ${NODE1_HOYA_NODE}
      HOYA_HTTP_PORT: ${NODE1_HOYA_HTTP_PORT}
      HOYA_P2P_PORT: ${NODE1_HOYA_P2P_PORT}
    entrypoint: ${HOYA_COMMAND}
    ports: 
      - "${NODE1_HOYA_HTTP_PORT}:${NODE1_HOYA_HTTP_PORT}"
    networks: 
      - hoya_net
    volumes: ['./QBFT-Network/${NODE1_HOYA_NODE}/data:/QBFT-Network/${NODE1_HOYA_NODE}/data']
    restart: always
  node2:  
    container_name: ${NODE2_HOYA_PEER_NAME}
    image: ${HOYA_NODE_IMAGE}
    environment:
      HOYA_PEER_NAME: ${NODE2_HOYA_PEER_NAME}
      HOYA_LOGGING: ${HOYA_LOGGING}
      HOYA_BESU_COLOR_ENABLED: ${HOYA_BESU_COLOR_ENABLED}
      HOYA_NODE: ${NODE2_HOYA_NODE}
      HOYA_HTTP_PORT: ${NODE2_HOYA_HTTP_PORT}
      HOYA_P2P_PORT: ${NODE2_HOYA_P2P_PORT}
    entrypoint: ${HOYA_COMMAND}
    ports: 
      - "${NODE2_HOYA_HTTP_PORT}:${NODE2_HOYA_HTTP_PORT}"
    networks: 
      - hoya_net
    volumes: ['./QBFT-Network/${NODE1_HOYA_NODE}/data:/QBFT-Network/${NODE1_HOYA_NODE}/data']
    restart: always
  node3:  
    container_name: ${NODE3_HOYA_PEER_NAME}
    image: ${HOYA_NODE_IMAGE}
    environment:
      HOYA_PEER_NAME: ${NODE3_HOYA_PEER_NAME}
      HOYA_LOGGING: ${HOYA_LOGGING}
      HOYA_BESU_COLOR_ENABLED: ${HOYA_BESU_COLOR_ENABLED}
      HOYA_NODE: ${NODE3_HOYA_NODE}
      HOYA_HTTP_PORT: ${NODE3_HOYA_HTTP_PORT}
      HOYA_P2P_PORT: ${NODE3_HOYA_P2P_PORT}
    entrypoint: ${HOYA_COMMAND}
    ports: 
      - "${NODE3_HOYA_HTTP_PORT}:${NODE3_HOYA_HTTP_PORT}"
    networks: 
      - hoya_net
    volumes: ['./QBFT-Network/${NODE1_HOYA_NODE}/data:/QBFT-Network/${NODE1_HOYA_NODE}/data']
    restart: always
  node4:  
    container_name: ${NODE4_HOYA_PEER_NAME}
    image: ${HOYA_NODE_IMAGE}
    environment:
      HOYA_PEER_NAME: ${NODE4_HOYA_PEER_NAME}
      HOYA_LOGGING: ${HOYA_LOGGING}
      HOYA_BESU_COLOR_ENABLED: ${HOYA_BESU_COLOR_ENABLED}
      HOYA_NODE: ${NODE4_HOYA_NODE}
      HOYA_HTTP_PORT: ${NODE4_HOYA_HTTP_PORT}
      HOYA_P2P_PORT: ${NODE4_HOYA_P2P_PORT}
    entrypoint: ${HOYA_COMMAND}
    ports: 
      - "${NODE4_HOYA_HTTP_PORT}:${NODE4_HOYA_HTTP_PORT}"
    networks: 
      - hoya_net
    volumes: ['./QBFT-Network/${NODE1_HOYA_NODE}/data:/QBFT-Network/${NODE1_HOYA_NODE}/data']
    restart: always

networks: 
  hoya_net: 
    driver: bridge
```

**docker-compose 구동**
```console
docker-compose up -d

docker logs -f {node1~node4}
```
![블록체인정상구동](/assets/img/blockchain/%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8%EC%A0%95%EC%83%81%EA%B5%AC%EB%8F%99.png)

**정상적으로 블록체인 구성되었는지 요청해보기**
```console
curl -X POST --data '{"jsonrpc":"2.0","method":"qbft_getValidatorsByBlockNumber","params":["latest"], "id":1}' localhost:8545
```
![curl테스트](/assets/img/blockchain/curl%ED%85%8C%EC%8A%A4%ED%8A%B8.png)

원래는 [시스템 아키텍처](https://leehh312.github.io/posts/hyperledger-besu-1/)대로 구성하려고 했으나 기본적인 노드4개만 구성하고 해당 컨텐츠는 끝내려고 합니다. 나머지들은 블록체인 문서를 보면 쉽게 플러그 또는 기능들을 추가할 수 있기 때문에 아래 링크에 타고 가셔서 직접 해보는것도 좋다고 생각합니다.  

## 참고자료  
[호야 블로그 Hyperledger Besu 샘플 저장소](https://github.com/leehh312/hyperledger-besu).  
[Hyperledger Besu Ethereum Client 공식 사이트](https://besu.hyperledger.org/en/stable/).  
