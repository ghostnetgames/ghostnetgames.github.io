---
title: Web Server 만들기 (모듈화) &#35;2
permalink: mydoc_go_server_2.html
keywords: golang, go, web, server
sidebar: mydoc_sidebar
folder: mydoc
---

### Go로 Web Server를 쉽게 만들어보자 2편
#### Module 분리하여 정의하기

```go
//http/webserver.go
package webserver

import (
	"ghostwebserver/types"
	"log"
	"net/http"
	"html/template"
	"string"
)

type HttpServer struct {
	param map[string] interface {}
}

func (httpServer *HttpServer) indexHandler(w http.ResponseWriter, r *http.Request) {
	// Ref: https://pkg.go.dev/net/url#URL
	httpServer.param["UrlParam"] = r.URL.Query()
	var filename string
	if r.URL.Path == "/" {
		filename = "index.html"
	} else {
		filename = strings.TremLeft(r.URL.Path, "/")
	}
	
	if t, err := template.ParseFiles(filename); err != nil {
		log.Fatal(err)
	} else {
		t.Execute(w, httpServer.param)
	}	
}

func (httpServer *HttpServer)StartServer(article []types.Artical) {
    // web site 첫 화면과 그것을 구성하는 css, js 파일을 Service 할 수 있는 handler를 등록한다.
    httpServer.param = make(map[string]interface{})
	httpServer.param["Article"] = article

	http.HandleFunc("/", httpServer.indexHandler) // indexHandler에 httpServer를 붙여야한다..why?
	log.Fatal(http.ListenAndServe(":9123", nil))
}
```

```go
//types/article.go
package types

type Article struct {
	Title string
}
```

```go
//main.go
package main

import (
	"ghostwebserver/types"
	"ghostwebserver/http"
	"fmt"
	"encoding/json"
	"io/ioutil"
)

func main() {
	var server webserver.HttpServer
	var article []types.Article

	if data, err := ioutil.ReadFile("./result.json"); err != nil {
		fmt.Println(err)
	} else {
		json.Unmarshal(data, &article)
	}

	server.StartServer(article)
}
```
#### Multi Param 사용하기
```go
httpServer.param["UrlParam"] = r.URL.Query()
...
httpServer.param["Article"] = article
```

#### Multi Param Html Template 보기
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
            {{range .Article}}
            <tr>
                <td scope="row">1</td>
                <td scope="row">{{.Title}}</td>
            </tr>
            {{end}}
        </tbody>
        </table>
		{{index .UrlParam.id "key"}}
    </body>
</html>
```
{% endraw %}
이전보다 Responsibility에 따라 모듈을 나눠 의존성이 개선되었다. 

{% include links.html %}
