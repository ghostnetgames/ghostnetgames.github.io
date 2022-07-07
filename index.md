---
title: "GhostNet Games"
keywords: sample homepage
tags: [getting_started]
sidebar: mydoc_sidebar
permalink: index.html
summary: GhostNet Games는 블록체인 네트워크인 GhostNet 플랫폼을 기반으로 하는 멀티 온라인 게임을 의미합니다. 을 기반하여 원할하게 동작하기 위해 Open API를 제공하고 문서를 공유하고 있습니다.
---

## Introduction
GhostNet Games는 블록체인 네트워크인 GhostNet 플랫폼을 기반으로 하는 멀티 온라인 게임을 의미합니다. 
게임이나 앱이 플랫폼GhostNet 플랫폼을 확장시키기 위해 개발되고 있으며 게임은 블록체인 기반 게임의 가능성을 발굴하기위해 서로 상호 호혜적인인 관계로 발전시키켜 나아가는 것이 목적입니다.

### 1. GhostNet이란?

Ghost Net은 독자적으로 구축된 자체 블록체인 네트워크입니다. 전력을 많이 사용하는 작업증명을 사용않고 독자적인 합의 알고리즘을 갖고 있으며 데이터를 분산 저장할 수 있는 기능을 내장하고 있습니다. [고스트넷 대표 블로그](https://ghostnetblog.github.io/)에서 확인할 수 있습니다. **Window Store** 와 **Play Store**에서 [다운로드](https://ghostnetblog.github.io/download/)할 수 있습니다. App Store는 애플 정책상 NFT 거래와 관련된 심사를 반려하고 있어서 업로드 되지 못하였습니다. 

### 2. GhostNet Platform
Ghost Net App은 코인 거래와 데이터 저장 기능을 자체 내장하고 있는 블록체인 플랫폼입니다. 자체 App에서는 다음과 같은 기능을 지원합니다.
* **SNS, Messenger** GhostNet은 P2P Network기반 Serverless Mobile Messenger입니다. 중앙 서버 없이 개인과 개인끼리 통신을 지원하며 자신의 메시지는 개인 기기에만 저장이 됩니다. GhostNet의 각 Ghost들은(Node) 팔로워와 팔로잉으로 연결되어 있으며 서로의 피드를 실시간으로 확인할 수 있습니다.
* **BlockChain Network** GhostNet은 Master Node들에 의해 운영되며 누구나 Master Node가 되어 BlockChain을 관리하는 주체가 될 수 있습니다. GhostNet은 그 보상으로 코인이 지급하고 Block Chain상에서 코인 송금이나 데이터 저장, NFT 거래 서비스를 제공합니다.
* **Cloud Service** BlockChain에 데이터를 저장하고 로드할 수 있는 클라우드 서비스를 제공합니다. 각 Master Node들은 Block Chain Data를 분산 저장하며 보상으로 코인을 지급받습니다. 클라우드 서비스는 GhostNetService.com에서 이용할 수 있습니다.
* **Open Api** Open Api기능을 제공하여 위 기능들을 외부 App들이 사용할 수 있도록 지원합니다.

### 3. Language
GhostNet은 C#과 Xamarin Platform 기반으로 만들어졌습니다. Xamarin이 지원하는 모든 Platform에서 동작할 수 있도록 설계되었으며 OpenApi도 C#을 기반으로 제작하여 C#을 Script 언어로 사용하는 Unity와 호환될 수 있도록 설계하였습니다. 

## GhostNet OpenApi 구조
OpenApi는 외부에서 제작된 App이 GhostNet App과 통신하기 위해 만들어진 Api입니다. 
GhostNet Platform의 Transport 계층과 BlockChain Client 계층을 그대로 Package화하여 배포합니다.
따라서 사용자는 Transaction의 형식과 생성 과정 등을 신경쓸 필요 없이 원하는 데이터를 요청하고 주고 받을 수 있습니다.



## GhostNet Api 보상 체계

### 1. 계정 생성
OpenApi를 사용하기 위해서는 GhostNet 계정을 생성해야합니다.
GhostNet과 통신하기 위해서 필수는 아니지만 보상을 받거나 Transaction을 모아볼 때 GhostNet 계정을 기준으로 필터링 할 수 있습니다.

### 2. Broker Id
OpenApi의 Api들 중 Transaction을 생성하는 Api들은 Broker Id를 입력할 수 있습니다. 
Broker Id는 GhostNet의 Master 노드들과 보상을 나눠 갖습니다.


{% include links.html %}
