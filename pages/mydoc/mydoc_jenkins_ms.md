---
title: Jenkins + MSBuild
permalink: mydoc_jenkins_ms.html
keywords: jenkins, MSBuild
sidebar: mydoc_sidebar
folder: mydoc
---

## Jenkins + MSBuild Setup


github에 계정과 비번으로 연결할 경우 github에서 거부한다.
그러나 계정과 비번으로 안하면 Project의 credentials에서 아무리 추가해도 보이지 않는다. 

https://stackoverflow.com/questions/45482507/how-do-i-install-nunit-3-console-on-windows-and-run-tests 에서 참조하여 해결
https://github.com/nunit/nunit-console/releases 에서 msi파일을 받아서 설치한다.
https://charliepoole.org/technical/nunit-engine-version-conflicts-in-visual-studio.html
에 보면 버전을 맞출수 있으나 실패
dotnet test로 수행하니까 잘된다.
https://stackoverflow.com/questions/43284881/jenkins-integration-for-dotnet-test

그러나 git push가 일어나도 build가 수행되지 않는다.
webhook 설정이 되지 않아서 그런것 같다.

윈도우이므로 openssh를 설치해야한다.

{% include links.html %}
