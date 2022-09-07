---
title: Supported features
tags:
  - getting_started
keywords: "features, capabilities, scalability, multichannel output, dita, hats, comparison, benefits"
last_updated: "July 16, 2016"
summary: "Ghost Network와 통신할 수 있는 기능과 기본적인 Transaction을 주고받는 기능을 지원합니다. 이를 통해 회원가입과 Database, 홀더등을 저장할 수 있습니다."
published: true
sidebar: mydoc_sidebar
permalink: mydoc_supported_features.html
folder: mydoc
---

GhostNet Api의 기술 문서를 보기 전에 이 기술 문서가 요구 사항을 충족하는지 확인하기 위해 필요한 몇 가지 기능의 지원 여부를 제공합니다. 
아래에 나열되어 있는 정보들은 GhostApi의 Api list가 아닌 기능 관점에서 정리된 항목입니다.

## Supported features

Features | Supported | Notes
--------|-----------|-----------
 계정생성| Yes | GhostNet의 계정 생성 기능을 지원합니다. GhostNet 계정을 생성시 트랜잭션이 생성되며 OpenApi를 사용하는 앱의 GhostNet 계정에게 보상이 돌아갑니다.|
계정조회 | Yes | 로그인에 사용할 수 있습니다. |
과금 | No | OpenApi 사용에 대한 어떠한 과금도 존재하지 않습니다.|
계정정보 | Yes | GhostNet은 실명, 이메일주소와 같은 개인 정보를 저장하지 않습니다. 오로지 계정의 주소와 닉네임, 프로필 사진, 소개와 같은 공개된 데이터만 제공합니다.
코인송금 | No | 보안상 이유로 코인 송금과 관련된 Transaction 생성은 제공하지 않습니다. |
홀더생성 |  Yes | 랜덤하게 생성할 수 있는 GhostNet 홀더 생성 기능을 제공합니다. |
홀더조회 | Yes | 계정이 보유한 홀더를 불러올 수 있습니다. 홀더의 레벨에 따라 앱은 아이템을 제공할 수 있습니다. |
데이터 저장 | Yes | GhostNet은 10mb의 공간을 제공합니다. 모든 데이터는 블록체인에 저장됨으로 공개되어 있습니다. Private 데이터는 별도의 암호화를 통해 저장해야합니다. |


{% include links.html %}
