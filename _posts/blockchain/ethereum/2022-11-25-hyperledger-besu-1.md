---
title: 하이퍼레저 비수(이더리움 기반) 구축(1)
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

## 시스템 아키텍처
!["hyperledger besu system architecture"](/assets/img/blockchain/hyperledger_besu_architecture.png)  

## 시스템 요구사항
**JVM Memory**
* JVM 메모리 최소 4GB

**OS**
* 메모리 크기 6GB
* 최소 20GB의 하드디스크  

## 하이퍼레저 비수 기본 개념 간단하게 정리
**하이퍼레저 비수란?**
* 하이퍼레저 비수는 아파치2.0 라이선스에 따라서 개발되었으며, 자바로 작성된 오픈소스 이더리움 클라이언트이다. 비수는 사설 네트워크에서 안전하며 고성능 트랜잭션 처리가 가능한 엔터프라이즈 앱에 개발하기 좋은 오픈소스이다.

**합의 알고리즘**
* Pow(Ethash) : 마이닝 증명으로 블록생성, 보상 개념의 알고리즘(비트코인, 이더리움)
* PoA(Clique, IBFT2.0, QBFT) : 개인신원을 이용한 블록 유효성 검사하며, 보상받을 권리 대신 개인신원 확인만 하는 알고리즘

**IBFT란?**
* 이스탄불 [**비잔틴 장애 허용**](http://wiki.hash.kr/index.php/%EB%B9%84%EC%9E%94%ED%8B%B4_%EC%9E%A5%EC%95%A0_%ED%97%88%EC%9A%A9) 매커니즘의 줄임말
* 시스템이 멈추거나 에러 메시지를 내보내는 장애 뿐만 아니라, 잘못된 값을 다른 시스템에 전달하는 등의 좀 더 원인을 파악하기 어려운 장애들 까지 포함한 말이며, 비잔티움 장애 허용이 구현된 시스템은 미리 정해진 정도를 넘지않는 부분에서 어떠한 현태의 장애가 있더라도 정확한 값을 전달할 수 있는 시스템

## 하이퍼레저 비수 구축하기 위한 서버 환경 구성
**전제 조건**
* docker 및 docker-compose

**도커 우분투 이미지 다운로드**
```console
docker pull ubuntu
```
**도커 이미지명 및 태그 변경**
```console
docker image tag ubuntu:latest ms_peer:heno
```
**기존 우분투 이미지 삭제 및 결과 확인**
```console
docker rmi ubuntu && docker images
```
**컨테이너 실행**
```console
docker run -id --name ms_peer {이미지아아디} /bin/bash
```
**컨테이너 진입**
```console
docker exec -it ms_peer /bin/bash
```
**apt(패키지 관리 명령어 도구) 업데이트**
```console
apt-get update && apt-get upgrade
```
**하이퍼레저 비수 구성에 필요한 툴 설치**
```console
apt-get install vim unzip wget jq -y
```

**JDK 11 설치**
```console
apt-get install openjdk-11-jdk && vi ~/.bashrc
```
**JDK 환경설정 저장후 스크립트 적용**
```bash
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
export PATH=${PATH}:$JAVA_HOME/bin
```

**언어 패키지 설치**
```console
apt-get -y install language-pack-ko
```
!["언어패키지설치1"](/assets/img/blockchain/언어패키지설치_1.png)
**템플릿들로 부터 현재 시스템에 파일들 생성**
```console
locale-gen ko_KR.UTF-8
```
!["언어패키지설치2"](/assets/img/blockchain/언어패키지설치_2.png)
**설치된 패키지(locales)를 재설정**
```console
dpkg-reconfigure locales
```
!["언어패키지설치3"](/assets/img/blockchain/언어패키지설치_3.png)
**패키지 환경설정**
```console
echo export LANGUAGE=ko_KR.UTF-8 export LANG=ko_KR.UTF-8 >> ~/.bashrc && source ~/.bashrc
```  

## 참고자료  
[Hyperledger Besu Ethereum Client 공식 사이트](https://besu.hyperledger.org/en/stable/).  
