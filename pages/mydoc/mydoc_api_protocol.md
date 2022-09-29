---
title: Open Api Protocol
permalink: mydoc_api_protocol.html
keywords: protocol, openapi
sidebar: mydoc_sidebar
folder: mydoc
---

## Open Api와 서버와의 통신
기본적으로 GhostNet의 Node들이 통신하는 기능을 모두 지원하지만 대부분이 BlockChain 관련 기능들이다.  
BlockChain 관련 프로토콜은  KeyValue Store 기능을 지원하고 있어 데이터를 저장하고 읽는 데는 Api를 사용하는데 문제가 없다.  

하지만 게임이나 채팅같은 다수의 참여자가 네트워킹하는 앱은 멀티플레이어가 모일 수 있는 로비부터 상대방의 메시지나 데이터를 동기화할 수 있는 방법이 필요하다. 특히 앱에서 사용되는 패킷 구조는 자유롭게 그 구조를 정의할 수 있기 때문에 json이나 Binary 포맷의 Serialize된 형태로 주고 받을 수 있어야 한다.   


## 로비의 구성

로비는 마스터 노드에서 생성한다.

- 로비 만들기
    - 사용자는 로비를 생성하고 로비의 Uniq한 Id를 받는 다. 
    - 로비는 제목과 내용, 비번을 생성할 수 있다.
    - Id를 통해 로비에 참여할 수 있어야 한다.
    - 로비에서 Event가 발생할 때마다 로비 참여자는 Event에 대한 메시지를 받는다.
    - 로비에 Event가 일정 시간 없는 경우 마스터 노드는 로비를 Delete 한다.



{% include links.html %}
