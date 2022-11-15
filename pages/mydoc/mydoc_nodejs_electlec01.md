---
title: Web bookmarker 만들기
permalink: mydoc_nodejs_electlec01.html
keywords: nodejs electron
sidebar: mydoc_sidebar
folder: mydoc
---

## 간단한 Web bookmarker 만들기
### 소개
아래 코드는 [Electron in Action: Chapter 2](https://livebook.manning.com/book/electron-in-action/chapter-2/)를 참고하였다.
파일 구성은 아래와 같다.
- Main Process
  - main.ts
  - preload.js
- Renderer Process
  - renderer.js
  - index.html  

electron은 `Main Process`와 `Renderer Process` 둘로 나눠지는데 Main Process에 해당하는 것은 main.ts(이것도 결국 main.js로 변환된다.)와 preload.js이고 Renderer Process에 해당하는 것은 index.html과 renderer.js이다.  
Renderer Process는 web Page를 로드하여 GUI를 구성한다.

최근 버전에서는 Renderer Process에서 보안상 이슈 때문에 system API를 접근하는 것이 허용되고 있지 않은 것 같다. 그래서 `contextBridge`를 사용하여 `ipc` 기반으로 Main Process와 Renderer Process가 통신하는 방법을 사용하는 것이 안전하다. 이것도 다룰 것이다.  

#### 개발 환경

```json
{
  "name": "electron_ex",
  "version": "1.0.0",
  "description": "",
  "main": "build/main.js",
  "type": "commonjs",
  "scripts": {
    "compile": "npx tsc && copyfiles -f index.html build",
    "start": "npm run compile && cross-env DEBUG=true electron .",
    "test": "test"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "copyfiles": "^2.4.1",
    "cross-env": "^7.0.3",
    "electron": "^21.2.0",
    "electron-builder": "^23.6.0",
    "typescript": "^4.8.4"
  },
}
```

### UI
index.html을 수정하는 것부터 시작한다.
```html
<!--index.html-->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' 'unsafe-inline';connect-src *">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Bookmarker</title>
</head>
```
`Fetch 삽질`  
header 부분에서 눈여겨 볼 것은 http-equiv field이다. 이것 때문에 일주일동안 삽질을 했다. header는 다른 파일에서 복붙하다보니 해당 tag가 누락되었다. 누락되었다기보다 content 내용이 달랐다. 그렇다보니 보안레벨이 높아졌고 renderer.js의 api들이 작동하지 않았다. 맞다. 이건 완전히 `초보자의 실수`다. 그러나 이 문제를 해결하기 위해 여기저기 찾아보면서 다양한 문서를 보게되었고 사람들의 다양한 고민들을 확인하면서 좀 더 감이 올라왔다.

```html
<!--index.html-->
<body>
  <h1>Bookmarker</h1>
  <div class="error-message"></div>
  <section class="add-new-link">
    <form class="new-link-form">
      <input type="url" class="new-link-url" placeholder="URL" size="100" required>
      <input type="submit" class="new-link-submit" value="Submit" disabled>
    </form>
  </section>
  <section class="links"></section>
  <section class="controls">
    <button class="clear-storage">Clear Storage</button>
  </section>
```
UI를 구성한다. class이름으로 Dom 요소에 접근할 것이다. 링크를 입력하고 submit 버튼을 누르면 site에서 정보를 로드하여 아래 links section에 tag들이 추가될 것이다.  
input type을 보면 `url`로 설정되어 있음을 알 수 있다. 내용이 유효한 URL패턴과 일치하지 않으면 Chromium에서 유효하지 않은 필드로 표시한다. 

```html
<!--index.html-->
  <script>
    require('./renderer.js');
   </script>
</body>
</html>
```

왠지 모르지만 typescript에서는 require 키워드를 인식하지 못한다. 여러가지 원인이 있는 것 같은데 알아보기 귀찮다. ES6라 그런것 같기도 하고... 일단 아래와 같이 수정할 수 있다. path는 사용자 환경에 맞춰서 기입하면 된다. 
```html
<script type="module" src="build/renderer.js"></script>
```

그리고 renderer.js를 로드하여 script가 동작할 수 있도록한다. 

```js
// renderer.js
const {shell} = require('electron');
const parser = new DOMParser();
const linksSection = document.querySelector('.links');
const errorMessage = document.querySelector('.error-message');
const newLinkForm = document.querySelector('.new-link-form');
const newLinkUrl = document.querySelector('.new-link-url');
const newLinkSubmit = document.querySelector('.new-link-submit');
const clearStorageButton = document.querySelector('.clear-storage');
```
html에서 정의한 dom요소의 로드하는 코드이다. 

```js
// renderer.js
newLinkForm.addEventListener('submit', (event) => {
    event.preventDefault();
    const url = newLinkUrl.value;
    fetch(url)
        .then(response => response.text())
        .then(parseResponse)
        .then(findTitle)
        .then(title => storeLink(title, url))
        .then(clearForm)
        .then(renderLinks);
});
const parseResponse = (text) => {
    return parser.parseFromString(text, 'text/html');
}
const findTitle = (nodes) => {
    return nodes.querySelector('title').innerText;
}
const storeLink = (title, url) => {
    localStorage.setItem(url, JSON.stringify({ title: title, url: url }));
}
```
`newLinkForm` 요소에 Event를 등록한다. 입력된 url 값을 기반으로 fetch를 수행한다. fetch api를 사용하여 외부 서버에 요청을 실행한다. 단순히 요청을 수행하기에 보안상 이슈가 있어 이 글을 읽어보면 도움이 될 것이다. [CORS개념](https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-CORS-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-%F0%9F%91%8F)  
url에 따라 html, json, xml등 여러가지 데이터를 요청할 수 있다. `response.text()`는 결과 값을 반환한다. `DOMParser`는 html 문자열을 분석하여 DOM 트리로 해석한다. `findTitle`은 DOM 트리를 탐색하여 `<title>` 노드를 찾는다.  
`localStorage`는 브라우저에 내장되어 있는 세션간 지속되는 간단한 KeyValue Store이다. 제목과 URL로 간단한 객체를 만들고 내장 Json 라이브러리를 사용하여 문자열로 변환한다음 URL을 키로 저장한다.

```js
// renderer.js
newLinkUrl.addEventListener('keyup', () => {
    newLinkSubmit.disabled = !newLinkUrl.validity.valid;
});
clearStorageButton.addEventListener('click', () => {
    localStorage.clear();
    linksSection.innerHTML = '';
});
linksSection.addEventListener('click', (event) => {
  if(event.target.href) {
    event.preventDefault();
    shell.openExternal(event.target.href);
  }
})
const clearForm = () => {
    newLinkUrl.value = null;
}
const getLinks = () => {
    return Object.keys(localStorage)
        .map(key => JSON.parse(localStorage.getItem(key)));
}
const convertToElement = (link) => {
    return `<div class="link"><h3>${link.title}</h3>
<p><a href="${link.url}">${link.url}</a></p></div>`;
}
const renderLinks = () => {
    const linkElements = getLinks().map(convertToElement).join('');
    linksSection.innerHTML = linkElements;
}
renderLinks();
```


### Main Process
```typescript
//main.ts
import { app, BrowserWindow } from "electron"; // ES import 

const path = require('path');

let window;

app.on("ready", () => {
  console.log('start electron app');
  window = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      nodeIntegration: true,
    },
  });
  window.webContents.openDevTools()
  window.loadFile("index.html");
});
```

```html
<meta http-equiv="Content-Security-Policy" content="default-src * self blob: data: gap:; style-src * self 'unsafe-inline' blob: data: gap:; script-src * 'self' 'unsafe-eval' 'unsafe-inline' blob: data: gap:; object-src * 'self' blob: data: gap:; img-src * self 'unsafe-inline' blob: data: gap:; connect-src self * 'unsafe-inline' blob: data: gap:; frame-src * self blob: data: gap:;">
```
혹은
```html
<meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    script-src 'self' 'unsafe-inline';
    connect-src *
    ">
```
> [stackoverflow:보안기능Off](https://stackoverflow.com/questions/31211359/refused-to-load-the-script-because-it-violates-the-following-content-security-po)  
> [추천:contextBridge를 통해 해결](https://github.com/electron-react-boilerplate/electron-react-boilerplate/issues/2957)

{% include links.html %}


