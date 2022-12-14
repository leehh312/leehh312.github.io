---
title: 트러플 이용하여 스마트 컨트랙트 배포하기
author: cotes
categories: [blockchain, smart-contract]
tags: [blockchain, ethereum, smartcontract]
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true
pin: true
render_with_liquid: false
---

## 트러플 개념 및 설치

**트러플이란?**  
이더리움 기반 디앱을 쉽게 개발할 수 있도록 도와주는 블록체인 프레임워크이다.
스마트 컨트랙트 컴파일, 배포, 관리, 테스트 쉽게할 수 있으며 트러플은 이더리움 기반 디앱 개발에 가장 많이 사용되는 프레임워크 중 하나이다.  

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
!["solidity download"](/assets/img/blockchain/vscode-solidity.png)  

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
필자는 1_deploy_contract.js이렇게 작성하였음.
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

필자가 올린 [하이퍼레저 비수(이더리움 기반) 구축](https://leehh312.github.io/posts/hyperledger-besu-1/)을 통해서 스마트 컨트랙트 배포 테스트해볼 예정이다.
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