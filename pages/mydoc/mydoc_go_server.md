---
title: Web Server 만들기 (기초) &#35;1
permalink: mydoc_go_server.html
keywords: golang, go, web, server
sidebar: mydoc_sidebar
folder: mydoc
---

### Go로 Web Server를 쉽게 만들어보자

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
