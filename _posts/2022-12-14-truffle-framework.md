---
title: 트러플 이용하여 스마트 컨트랙트 배포하기
author: cotes
categories: [blockchain, smartcontract]
tags: [blockchain, ethereum, smartcontract]
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true
pin: false
render_with_liquid: false
---

## 트러플 개념 및 설치

**트러플(Truffle)이란?**  
트러플(Truffle)은 이더리움(Ethereum) 기반의 탈중앙화 애플리케이션(DApp) 개발 과정을 획기적으로 간소화하고 자동화해주는 세계에서 가장 널리 사용되는 블록체인 개발 프레임워크입니다.

웹 개발에 비유하자면, '이더리움 세계의 루비 온 레일즈(Ruby on Rails)' 나 '스프링 부트(Spring Boot)' 와 같은 역할을 합니다. 복잡한 설정 없이도 스마트 컨트랙트의 개발, 테스트, 배포에 이르는 전체 라이프사이클을 효율적으로 관리할 수 있는 강력한 도구 모음을 제공합니다.

**트러플의 핵심 역할**
* 스마트 컨트랙트 관리 (Smart Contract Management)
  * 솔리디티(Solidity)로 작성된 코드를 손쉽게 컴파일하고, 다양한 블록체인 네트워크에 배포하며, 컨트랙트의 버전과 상태를 관리합니다.
* 자동화된 테스트 (Automated Testing)
  * 단위 테스트(Unit Test)와 통합 테스트(Integration Test)를 위한 강력한 프레임워크를 내장하고 있어, 컨트랙트의 안정성과 신뢰도를 높일 수 있습니다.
* 편리한 네트워크 관리 (Convenient Network Management)
  * 로컬 개발 환경(Ganache), 이더리움 테스트넷(Sepolia 등), 그리고 실제 운영 환경인 메인넷(Mainnet)까지, 복잡한 설정 없이 원하는 네트워크에 스마트 컨트랙트를 손쉽게 배포할 수 있습니다.
* 강력한 개발 생태계 (Powerful Development Ecosystem)
  * 트러플은 Ganache(개인용 로컬 블록체인), Drizzle(프론트엔드 라이브러리) 등과 함께 '트러플 스위트(Truffle Suite)' 를 구성하여, 디앱 개발에 필요한 거의 모든 환경을 통합적으로 제공합니다.

**요구사항**  
* Node.js v14 - v18

**nvm 설치 및 nodejs 설치**
```bash
brew install nvm

# user home 디렉토리에 .nvm 파일 생성
mkdir ~/.nvm

# 편집기로 .zshrc파일 열기
vi ~/.zshrc

# .zshrc파일에 붙여넣은 후 저장
export NVM_DIR="$HOME/.nvm"
  [ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"  
  # This loads nvm
  [ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"  
  # This loads nvm bash_completio

# 저장 후 적용
source ~/.zshrc 

# 정상 설치 확인
nvm -v

# nodejs 18버전 설치
nvm install 18

# nodejs 18버전 사용
nvm use 18

# nodejs 버전 확인
node --version
```  

**트러플 설치**
```bash
npm install -g truffle

# 버전확인
truffle version
```

## VS Code IDE 이용하기
```bash
mkdir test-contract
```
!["폴더열기"](/assets/img/blockchain/vscode-folder.png)  
!["truffle init"](/assets/img/blockchain/vscode-truffle.png)
!["truffle result"](/assets/img/blockchain/vscode-directory.png)  
* contracts
솔리디티 소스 디렉토리 

* migrations
이더리움 네트워크에 배포할 때 사용되는 자바스크립트 파일 디렉토리  

* test
Application, Contract 테스트 파일 디렉토리  

* truffle.js
트러플 구성 파일  

## 간단한 스마트 컨트랙트 작성
**마켓플레이스에서 솔리디티 플러그인 다운로드**
> command+shift+x 단축키 눌러 Solidity 검색 후 다운로드  
> !["solidity download"](/assets/img/blockchain/vscode-solidity.png)  

contracts에서 파일 생성 후 작성
```solidity
//SPDX-License-Identifier: MIT
// SPDX 라이센스는 0.68 이후 부터, 솔리디티 프로그램 맨위에 명시를 요구
pragma solidity ^0.8.1;

contract TestStorage{
    struct UserInfo {
        string serial;
        string name;
        uint32 age;
        string phone_number;
    }

    mapping (string => UserInfo) private userInfoList;

    event RegUserInfo(UserInfo userinfo);

    function pingpong() public pure returns (string memory){
        return "Hello World";
    }

    function getUserInfo(string memory _serial) public view returns (UserInfo memory){
        UserInfo memory userInfo = userInfoList[_serial];
        
        return userInfo;
    }

    function setUserInfo(string memory _serial, string memory _name, uint32 _age, string memory _phone_number) public{
        UserInfo memory userinfo = UserInfo(_serial, _name, _age, _phone_number);
        userInfoList[_serial] = userinfo;

        emit RegUserInfo(userinfo);
    }
}
```

## 스크립트 배포 파일 작성  
migrations에서 1_파일명.js 작성
저는 파일은명 `1_deploy_contract.js` 이렇게 작성하였음.
```js
// require("스마트 컨트랙트명 기입")
const TestStorage = artifacts.require("TestStorage");

module.exports = async function (deployer) {
  deployer.then(function () {
    return deployer.deploy(DidStorage);
  }).then(function (DidStorage) {
    console.log(`Use this contract address. : ${DidStorage.address}`);
  });
};
```

## truffle-config.js 작성
```bash
# 자바스크립트 HD 지갑 제공자인 hdwallet-provider 설치
npm install @truffle/hdwallet-provider
```

이전에 제가 올린 [하이퍼레저 비수(이더리움 기반) 구축](https://leehh312.github.io/posts/hyperledger-besu-1/)을 이어서, 스마트 컨트랙트 배포를 테스트해볼 예정입니다.
```js
// truffle-config.js
const HDWalletProvider = require('@truffle/hdwallet-provider');

const privateKeys = ["e61d520cd0384f658256cc6da249572679575ce036ab7f96fbffc7e79889ff57"];

const url = "http://127.0.0.1:8545"

const provider = new HDWalletProvider(privateKeys, url);

module.exports = {
  networks: {
    hoya: {
      provider: () => provider,
      network_id: "*",
      gasPrice: 0,
     },
  },

  mocha: {
    // timeout: 100000
  },

  // Configure your compilers
  compilers: {
    solc: {
      version: "0.8.1",
      settings: {          // See the solidity docs for advice about optimization and evmVersion
        optimizer: {
          enabled: true,
          runs: 1000,
         },
      }
    }
  },
};
```  
## 스마트 컨트랙트 배포
```bash
truffle migirate --network hoya --to 1 -f 1 --reset
```
![배포 결과](/assets/img/blockchain/truffle_migirate.png)
![블록체인 결과](/assets/img/blockchain/truffle_bc_migirate.png)

## 참고자료  
[호야 블로그 트러플 프레임워크 샘플 저장소](https://github.com/leehh312/test-contract).  