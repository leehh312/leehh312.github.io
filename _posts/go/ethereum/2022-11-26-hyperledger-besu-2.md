---
title: 하이퍼레저 비수(이더리움 기반) 구축(2)
author: cotes
categories: [blockchain, ethereum]
tags: [blockchain, hyperledger besu, ethereum, 블록체인, 하이퍼레저비수, 이더리움]
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true
pin: true
render_with_liquid: false
---

## QBFT 네트워크 기반 하이퍼레저 비수 구축
**하이퍼레저 비수 설치**
```console
cd /usr/local/bin && wget https://hyperledger.jfrog.io/hyperledger/besu-binaries/besu/22.10.0/besu-22.10.0.tar.gz
```

**하이퍼레저 압축 풀기**
```console
tar -zxvf besu-22.10.0.tar.gz
```

**하이퍼레저 비수 환경설정**
```console
echo export PATH=${PATH}:/usr/local/bin/besu-22.10.0/bin >> ~/.bashrc && source ~/.bashrc
```

**정상 설치 확인**
!["비수 정상 설치 확인"](/assets/img/blockchain/%EB%B2%A0%EC%88%98%EC%84%A4%EC%B9%98%EC%A0%95%EC%83%81%ED%99%95%EC%9D%B8.png)

**디렉토리 구성**
```console
mkdir -p /QBFT-Network/{Node-1,Node-2,Node-3,Node-4}/data && cd /QBFT-Network
```
!["비수 디렉토리 구성"](/assets/img/blockchain/%EB%B2%A0%EC%88%98%EB%94%94%EB%A0%89%ED%86%A0%EB%A6%AC%EA%B5%AC%EC%84%B1.png)

**노드 키쌍 생성**
```console
echo '{
 "blockchain": {
   "nodes": {
     "generate": true,
       "count": 4
   }
 }
}
' | jq '.' >> config.json
````

```console
besu operator generate-blockchain-config --config-file=config.json --to=keyPairs --private-key-file-name=key
```
<span style="color: #FF4848">에러는 무시해도 된다 현재 노드 키쌍만 생성하는 명령어가 존재하지않아 이러한 방법으로 키쌍 생성 진행하였음.</span>
!["키쌍생성"](/assets/img/blockchain/%ED%82%A4%EC%8C%8D%EC%83%9D%EC%84%B1.png)

**키쌍 각 노드에 배치**

<span style="color: #FF4848">아래의 이미지와 같이 최종적으로 작업 진행하면 된다.</span>
!["키쌍생성결과"](/assets/img/blockchain/%ED%82%A4%EC%8C%8D%EC%83%9D%EC%84%B1%EA%B2%B0%EA%B3%BC.png)

**각 노드 계정(Address) 파일 생성**
```console
besu public-key export-address --node-private-key-file=Node-1/data/key --to=Node-1/data/account

besu public-key export-address --node-private-key-file=Node-2/data/key --to=Node-2/data/account

besu public-key export-address --node-private-key-file=Node-3/data/key --to=Node-3/data/account

besu public-key export-address --node-private-key-file=Node-4/data/key --to=Node-4/data/account
```

**Extra Data 파일 생성**
```console
jq --null-input \
  --arg node1 $(cat Node-1/data/account) \
  --arg node2 $(cat Node-2/data/account) \
  --arg node3 $(cat Node-3/data/account) \
  --arg node4 $(cat Node-4/data/account) \
  '[$node1, $node2, $node3, $node4]' >> extra_config.json
```

**Extra Data RLP 인코딩**
이더리움 네트워크에서 노드에 데이터 구조를 저장하거나, 혹은 노드끼리 데이터 구조를 주고 받으려면 통일된 형식이 필요하며, 이더리움에서는 RLP 인코딩 방식을 선택한것이다.[^rlp]

<span style="color: #FF4848">출력된 RLP인코딩 값 가지고 있다 제네시스에 설정해야한다.</span>
```console
besu rlp encode --from=extra_config.json --type=QBFT_EXTRA_DATA
```

**제네시스 파일 구성**
```
vi /QBFT-Network/genesis.json
```
블록체인의 첫 번째 블록을 제네시스 블록이라고 하며, 제네시스 파일은 블록체인 자체에 대한 규칙뿐만 아니라 블록체인의 첫 번째 블록에 있는 데이터를 정의한다. 또 프라이빗이나 퍼블릭 상관없이 새로운 노드가 참가하게 되면 이 제네시스 블록이 새로 생성된다.[^genesis-file]

<span style="color: #FF4848">alloc필드 내부에 있는 속성들 수정 진행 해야하며, 앞에서 생성한 계정 및 개인키에 보면 prefix로 0x가 붙어있는데 제거해서 수정 진행 </span>

```json
{
  "config" : {
    "chainId" : 20221126,
    "muirglacierblock": 0,
    "qbft" : {
      "blockperiodseconds" : 4,
      "epochlength" : 30000,
      "requesttimeoutseconds" : 6
    },
    "contractSizeLimit" : 2147483647
  },
  "evmStackSize" : 5000000,
  "messageQueueLimit" : 5000000,
  "duplicateMessageLimit" :1000,
  "futureMessagesLimit" : 5000000,
  "nonce" : "0x0",
  "timestamp" : "0x58ee40ba",
  "gasLimit" : "0x1fffffffffffff",
  "difficulty" : "0x1",
  "mixHash" : "0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365",
  "coinbase" : "0x0000000000000000000000000000000000000000",
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "alloc" : {
    "11549197c99e4b886ed7d6ed2843f534a4f367fc" : {
      "privateKey" : "691193c87afa8c0fe94c5ef32a5b223879f4db1c956f6f070e31a4ae3519d996",
      "balance" : "0xad78ebc5ac6200000"
    },
    "66283b796406855fcb763d60742e922c0143deef" : {
      "privateKey" : "cd43e18391ab9eb9851538b8e5b1f5602ab81059cebcf282ad7cced859c94513",
      "balance" : "0xad78ebc5ac6200000"
    },
    "b7bb70ad693cc227d45507d5579a7077caa5f426" : {
      "privateKey" : "ca0196e07feeb2d7912bb96e6a73c79ee00928e53dc9e5b8cd886c6897ea3068",
      "balance" : "0xad78ebc5ac6200000"
    },
    "edad1970f705d11f59130b29d76c37332772e24c" : {
      "privateKey" : "2da52730a784fe5e1a0b4c127409e24b6e14f2c6ba9ef2ccb063907d8855db83",
      "balance" : "0xad78ebc5ac6200000"
    }
  },
  "extraData" : "0xf87aa00000000000000000000000000000000000000000000000000000000000000000f8549411549197c99e4b886ed7d6ed2843f534a4f367fc9466283b796406855fcb763d60742e922c0143deef94b7bb70ad693cc227d45507d5579a7077caa5f42694edad1970f705d11f59130b29d76c37332772e24cc080c0"
}
```  

## 참조  
[^rlp]: [RLP 개념 및 원리](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp/)  
[^genesis-file]: [제네시스 파일 옵션 및 개념](https://besu.hyperledger.org/en/stable/public-networks/concepts/genesis-file/)  