---
title: Golang + gRPC 사용하기 
permalink: mydoc_go_grpc.html
keywords: golang, go, grpc, server
sidebar: mydoc_sidebar
folder: mydoc
---

### Golang + gRPC 환경 설정 

golang 1.16 이상 버전이 필요하다고 한다. golang update를 한다.  
왠지 모르겠지만 apt-get은 잘 안되는 것 같다. 해결하기 보단 구글링해서 사람들이 많이 하는 방식으로 했다.
```bash
# 기존 golang 삭제
$ sudo apt remove golang
$ sudo apt autoremove
# 새로운 golang 설치 
$ git clone https://github.com/udhos/update-golang
$ cd update-golang
$ sudo ./update-golang.sh
```
그리고 protobuf-compiler를 설치해야한다.
```bash
$ apt install -y protobuf-compiler
$ protoc --version  # Ensure compiler version is 3+
```
#### Proto 컴파일해보기 
[Example 코드](https://github.com/grpc/grpc-go/tree/master/examples/helloworld) 공식 guthub에서 받을 수 있는 소스 코드를 기반으로 분석한다.  
```proto
syntax = "proto3";

option go_package = "google.golang.org/grpc/examples/helloworld/helloworld";
option java_multiple_files = true;
option java_package = "io.grpc.examples.helloworld";
option java_outer_classname = "HelloWorldProto";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```
아래 명령어를 통해 proto 파일이 잘 컴파일 되는지 확인한다. 
```bash
$ protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative helloworld/helloworld.proto
```
그런데 `Error "protoc-gen-go: program not found or is not executable"` 메시지가 발생한다면 GOPATH가 설정이 안되었거나 다른 방법으로 설치를 제안한다.
```bash
#~/.bashrc
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
`cannot find package "google.golang.org/protobuf/cmd/protoc-gen-go"`메시지가 출력된다면 grpc를 설치해야한다. 
```bash
$ go get -u google.golang.org/protobuf/cmd/protoc-gen-go
$ go get -u google.golang.org/grpc/cmd/protoc-gen-go-grpc
# get이 안되는 버전이 있다.
$ go install google.golang.org/protobuf/cmd/protoc-gen-go
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
```
아래와 같은 메시지가 출력된다면 `go get google.golang.org/grpc` 을 수행한다. 웹사이트 가이드가 조금 탐탁치 않다. 처음부터 이 명령어를 실행하면 다 설치되는 거 아닌가...
```bash
helloworld/helloworld_grpc.pb.go:11:2: no required module provides package google.golang.org/grpc; to add it:
        go get google.golang.org/grpc
```
다시 proto 컴파일을 시도하면 잘 완료될 것이다.

### gRPC 코드 분석


{% include links.html %}
