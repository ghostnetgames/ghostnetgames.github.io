---
title: Overview
permalink: mydoc_api_overview.html
keywords: overview, openapi
sidebar: mydoc_sidebar
folder: mydoc
---

## OpenApi 지원 플랫폼
.Net Standard Class Library 2.1을 지원하고 있습니다. 
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
RequestMasterNodeList | Master Node List를 요청합니다. 연결된 Master Node에게 전달합니다. Master Node들은 네트워크를 구성하고 채굴을 하는 중심 노드입니다.
RequestMasterFriendNodeList | Master Node Friend List를 요청합니다. Master Node Friend는 Broker로도 명칭하는데 Master Node와 유저를 중계하는 역할을 합니다. 
RequestSearchGhostInfo | Ghost 유저 정보를 요청합니다.

{% include links.html %}
