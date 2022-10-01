---
title: Quick Start
permalink: mydoc_api_quickstart.html
keywords: protocol, openapi
sidebar: mydoc_sidebar
folder: mydoc
---
## 초기화 과정
- 제작자 계정이 필요합니다.
    - 제작자의 블록체인 지갑 주소를 초기화시 정보를 전달합니다.
- 유저의 계정이 필요합니다.
    - 계정을 생성합니다.
        - Api를 통해 생성할 수도 있고 GhostNet앱으로 연결해도 됩니다.
    - 이미 계정이 있을 경우 로그인을 합니다.
- 연결할 Master Node의 리스트를 받아옵니다.
- 연결할 Master Node를 선택합니다.(Random)
- 선택된 Master Node에게 유저의 정보를 전달하며 연결 합니다.


## Master Friend(broker) Account

GhostNet에 접속하는 모든 인스턴스는 계정을 갖고 있어야 합니다. GhostNet은 Master Node와 Normal Node로 구성되는데 특별한 역할을 하는 노드가 하나 더 있는데 `Master Friend Node` 라고 합니다.  

이 노드는 Master Node와 Normal Node를 중계하는 역할을 갖고 있으며 Master Node와 **보상을 나누어 갖습니다.** 따라서 Api 초기화시 제작자의 계정을 기입해야합니다.

## OpenApi 초기화
```c#
brokerUserInfo = new GhostNetUser() { 
    PubKeyAddress = "", 
    PublicIP = new GhostIp() { ip = "xxx.xx.x.x", port = "xx" }
};
ghostPacketEventHandler = new GhostPacketEventHandler();
var ghostOpenApi = new GhostClient(brokerUserInfo, ghostPacketEventHandler);
```
`GhostClient`는 root 노드에서 Master Node List를 요청한다. 그와 함께 Master Friend Account를 함께 전달해야한다. 그리고 Event가 도착했을 때 Callback 함수를 호출하는 Handler 구조체를 등록한다.

```c#
bool ok = await ghostOpenApi.OpenApiSync();
```
`OpenApiSync()`는 Network 연결과 셋업에 필요한 정보를 요청한다. 
- Network instance를 초기화 한다.
- root 노드에 Master Node List를 요청한다.
- Callback 함수를 사용한다면 `OpenApi()`를 사용한다.
- `OpenApiSync()`함수는 동작을 모두 완료하면 Return된다.

## OpenApi 로그인
```c#
GhostNetUser ghostNetUser = await ghostOpenApi.GhostNetLoginSync(nickname, passwd)
```
`GhostNetLogin`은 GhostNet에 로그인하고 유저 정보를 Return합니다. 로그인에 실패할 경우 null 값을 반환합니다.

{% include links.html %}
