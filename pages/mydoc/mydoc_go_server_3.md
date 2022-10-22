---
title: Web Server 만들기 (header, footer)) &#35;3
permalink: mydoc_go_server_3.html
keywords: golang, go, web, server
sidebar: mydoc_sidebar
folder: mydoc
---

### Go로 Web Server를 쉽게 만들어보자 3편
Web App을 만들다 보면 header와 footer와 같이 모든 페이지에 공통적으로 들어가는 코드가 있다. 이것을 별도의 파일에 정의하여 include를 통해 외부에 link를 하고 값을 로드해서 사용하는 형태로 구성한다.  
html/template에서도 해당 기능을 지원하고 있다.  

```go
// t, _ := template.ParseFiles(filename)
t, _ := template.ParseFiles(filename, "header.html")
t.Execute(w, article)
```
2편에서 parameter 하나만 추가하면 된다.
{% raw %}
```html
<html lang="en">
    <head>
        {{template "header.html"}}
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
header.html의 내용은 아래와 같다.
```html
<title>GhostWebService</title>
<link rel="stylesheet" href="/bootstrap.css">
<script src="/bootstrap.js"></script>
```
{% endraw %}

{% include links.html %}
