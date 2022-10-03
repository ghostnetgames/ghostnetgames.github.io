---
title: OpenApi Architecture
permalink: mydoc_api_architecture.html
keywords: api
sidebar: mydoc_sidebar
folder: mydoc
---

## OpenApi Architecture

OpenApi는 Release된 GhostNet에 사용된 Core를 분리하여 제작하였습니다. 의존성을 붐리하여 개발한 덕분에 쉽게 분리하여 첫번째 버전을 생성하였으나 구조 개선과 사용성 개선을 위해 개발중입니다.

### OpenApi 구조

OpenApi는 GhostNet 구조와 크게 다르지 않습니다. 다만 클라이언트다보니 Distributed Network Layer가 존재하지 않습니다.

- Network Layer
- Application Layer
    - Blockchain 
    - GhostNetRobby 
    - Manager

### Network Layer
OpenApi는 UDP를 기본으로 합니다. UDP는 Packet 전송과 순서를 보장하지 않기 때문에 이를 염두해 사용해야합니다. 

### Application Layer
Application Layer는 블록체인 모듈과 멀티사용자 통신을 위한 Robby, 그리고 이를 관리하믄 매니저로 나뉩니다.

#### 블록체인 
기본적인 Block Parsing과 트랜잭션 생성, 암호화 등에 사용됩니다.

#### GhostNetRobby
멀티사용자를 위한 Robby기능입니다. Robby는 마스터 노드에서 생성되며 Robby를 기준으로 사용자의 연결이 관리됩니다. 마스터 노드에서 생성된 Robby는 일정 시간 응답이 없을 경우 자동으로 삭제됩니다. 

#### Manager
OpenApi를 초기화하고 생성된 오브젝트들을 관리하는 역활을 합니다. 

{% include links.html %}
