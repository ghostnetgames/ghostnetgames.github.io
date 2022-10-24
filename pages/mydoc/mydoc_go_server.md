---
title: Web Server 만들기 (기초) &#35;1
permalink: mydoc_go_server.html
keywords: golang, go, web, server
sidebar: mydoc_sidebar
folder: mydoc
---

### Go로 Web Server를 쉽게 만들어보자
golang은 간단한 web server를 만들수 있도록 기본 패키지를 제공한다. golang 책을 보더라도 다른 언어 책과 다르게 http서버 제작 챕터가 나온다. 그만큼 언어 자체가 네트워크 기능이 강력함을 느낄 수 있다. Django로 서버구축을 하였으나 강력한 Platform을 바탕으로 지원되어 있어 python 언어외에 django 사용법을 익혀야하며 내부 원리나 인터페이스 측면에서 배움에 한계가 있어 아쉬움이 있었다. 
#### Golang 기반 tiny webserver
GhostNet은 분산 컴퓨팅이라는 큰 목표를 갖고 있는데 하나의 서버를 구축하는 것은 목표에 맞지 않다. 따라서 각 마스터 노드에 Tiny Webserver를 구축하여 트래픽을 분산해보고자 한다. golang도 배울겸 golang으로 웹서버를 만들어 보자.

```go
	http.HandleFunc("/bootstrap.js", jsHandler)
	http.ListenAndServe(":9123", nil)
```
가장 기본적인 http 함수들이다.

```go
package main

import (
	"log"
	"net/http"
	"html/template"
	"encoding/json"
	"io/ioutil"
)

type Article struct {
	Title string
}

// file들을 별도 요청하기 때문에 handler로 등록해야한다.
// 확장성을 생각하면 file 이름을 구해서 해당 파일을 읽어야하지만
// 일단 두개 파일만 등록한다.
func cssHandler(w http.ResponseWriter, r *http.Request) {
	t, err := template.ParseFiles("bootstrap.css")
	if err != nil {
		log.Fatal(err)
	}
	t.Execute(w, nil)
}

func jsHandler(w http.ResponseWriter, r *http.Request) {
	t, err := template.ParseFiles("bootstrap.js")
	if err != nil {
		log.Fatal(err)
	}
	t.Execute(w, nil)
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
	t, err := template.ParseFiles("index.html")
	if err != nil {
		log.Fatal(err)
	}
	data, err := ioutil.ReadFile("./result.json")
	if err != nil {
		log.Fatal(err)
	}
	var article []Article
	json.Unmarshal(data, &article)
	t.Execute(w, article)
}

func main() {
    // web site 첫 화면과 그것을 구성하는 css, js 파일을 Service 할 수 있는 handler를 등록한다.
    http.HandleFunc("/", indexHandler)
	http.HandleFunc("/bootstrap.css", cssHandler)
	http.HandleFunc("/bootstrap.js", jsHandler)

	log.Fatal(http.ListenAndServe(":9123", nil))
}
```
{% raw %}
```html
<html lang="en">
    <head>
        <title>GhostWebService</title>
        <link rel="stylesheet" href="/bootstrap.css">
        <script src="/bootstrap.js"></script>
    </head>
    <body>
        <table class="table">
        <thead>
            <th scope="col">#</th>
            <th scope="col">test1</th>
        </thead>
        <tbody>
            {{range .}}
            <tr>
                <td scope="row">1</td>
                <td scope="row">{{.Title}}</td>
            </tr>
            {{end}}
        </tbody>
        </table>
    </body>
</html>
```
{% endraw %}
{% include links.html %}
