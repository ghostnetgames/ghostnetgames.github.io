---
title: Golang + httptest 만들기
permalink: mydoc_go_servertest.html
keywords: golang, go, web, server
sidebar: mydoc_sidebar
folder: mydoc
---

### Testing Http
go의 http 패키지를 테스트 해보려니 감이 잡히지 않는다. request와 response는 어떻게 생성하며 mocking은 어떻게 해야할까? 구글링 중에 httptest 패키지를 발견하였다. 역시 대단한 사람들이다. 코드는 아래 링크를 참고하였다.  
[Golang httptest Example](https://golang.cafe/blog/golang-httptest-example.html)
go httptest 패키지는 크게 두 가지 형태로 사용할 수 있다. Server와 Client이다.
- testing Go http server handlers
- testing Go http clients

#### Testing HttpServer
```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "net/url"
    "strings"
)

// Req: http://localhost:1234/upper?word=abc
// Res: ABC
func upperCaseHandler(w http.ResponseWriter, r *http.Request) {
    query, err := url.ParseQuery(r.URL.RawQuery)
    if err != nil {
        w.WriteHeader(http.StatusBadRequest)
        fmt.Fprintf(w, "invalid request")
        return
    }
    word := query.Get("word")
    if len(word) == 0 {
        w.WriteHeader(http.StatusBadRequest)
        fmt.Fprintf(w, "missing word")
        return
    }
    w.WriteHeader(http.StatusOK)
    fmt.Fprintf(w, strings.ToUpper(word))
}

func main() {
    http.HandleFunc("/upper", upperCaseHandler)
    log.Fatal(http.ListenAndServe(":1234", nil))
}
```

```go
package main

import (
    "io/ioutil"
    "net/http"
    "net/http/httptest"
    "testing"
)

func TestUpperCaseHandler(t *testing.T) {
    req := httptest.NewRequest(http.MethodGet, "/upper?word=abc", nil)
    w := httptest.NewRecorder()
    upperCaseHandler(w, req)
    res := w.Result()
    defer res.Body.Close()
    data, err := ioutil.ReadAll(res.Body)
    if err != nil {
        t.Errorf("expected error to be nil got %v", err)
    }
    if string(data) != "ABC" {
        t.Errorf("expected ABC got %v", string(data))
    }
}
```

```go
func (rw *ResponseRecorder) Result() *http.Response
```

#### Testing HttpClient

```go
package main

import (
    "io/ioutil"
    "net/http"

    "github.com/pkg/errors"
)

type Client struct {
    url string
}

func NewClient(url string) Client {
    return Client{url}
}

func (c Client) UpperCase(word string) (string, error) {
    res, err := http.Get(c.url + "/upper?word=" + word)
    if err != nil {
        return "", errors.Wrap(err, "unable to complete Get request")
    }
    defer res.Body.Close()
    out, err := ioutil.ReadAll(res.Body)
    if err != nil {
        return "", errors.Wrap(err, "unable to read response data")
    }

    return string(out), nil
}
```

```go
func NewServer(handler http.Handler) *Server
```

```go
package main

import (
    "fmt"
    "net/http"
    "net/http/httptest"
    "strings"
    "testing"
)

func TestClientUpperCase(t *testing.T) {
    expected := "dummy data"
    svr := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, expected)
    }))
    defer svr.Close()
    c := NewClient(svr.URL)
    res, err := c.UpperCase("anything")
    if err != nil {
        t.Errorf("expected err to be nil got %v", err)
    }
    // res: expected\r\n
    // due to the http protocol cleanup response
    res = strings.TrimSpace(res)
    if res != expected {
        t.Errorf("expected res to be %s got %s", expected, res)
    }
}
```

{% include links.html %}
