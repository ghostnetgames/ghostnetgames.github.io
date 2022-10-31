---
title: Overview
permalink: mydoc_api_overview.html
keywords: overview, openapi
sidebar: mydoc_sidebar
folder: mydoc
---

## OpenApi 지원 플랫폼
.NET Standard Class Library 2.1을 지원하고 있습니다. 
.NET Standard 라이브러리는 .NET Core, .NET Framework, Mono / Xamarin과 같은 .NET Standard 호환 런타임에서 실행됩니다.
* Xamarin
* WPF 및 윈도우 Application
* Unity

{% include note.html content="OpenApi를 사용하여 앱을 개발하려면 .NET Standard를 잘 알고 있어야 합니다." %}

## Network 연결 방식
OpenApi는 비연결(UDP) 방식을 지향합니다. 
따라서 네트워크 상황에 따라 Packet을 잃어버릴 수 있습니다.
재전송 매카니즘은 각 앱에서 구현해야합니다.

## OpenAPi List

Function Name | Notes
--------|-----------
OpenApi | 동작에 필요한 객체들을 초기화합니다. 무조건 처음에 호출되어야 합니다.
RequestMasterNodeList | Master Node List를 요청합니다. 연결된 Master Node에게 전달합니다. Master Node들은 네트워크를 구성하고 채굴을 하는 중심 노드입니다.
RequestMasterFriendNodeList | Master Node Friend List를 요청합니다. Master Node Friend는 Broker로도 명칭하는데 Master Node와 유저를 중계하는 역할을 합니다. 
RequestSearchGhostInfo | Ghost 유저 정보를 요청합니다.
RequestSearchMasterInfo| Master 정보를 요청합니다.
CheckGhostNickname | 이미 존재하는 닉네임인지 확인합니다.
GetGhostNetTuplebyNickname | 닉네임으로 User 정보를 조회합니다. 2개 이상 존재할 수 있습니다.
GetNFTPackageList | 특정 Block의 GhostNFT List를 반환합니다.
GetUserNFTPackageList | 특정 유저의 GhostNFT List를 반환합니다.
GetDataTxIdList | 특정 Block의 특정 유저의 DataTxId List를 반환합니다.
GetBlock | 특정 Block을 다운로드합니다.
MakeNewGhostTuple | Tuple을 생성합니다.
SendGhostNetTuple | Tuple을 GhostNet에 저장합니다.




{% include links.html %}
