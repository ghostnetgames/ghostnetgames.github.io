---
title: Electron을 typescript로 확장하기
permalink: mydoc_nodejs_electypescript.html
keywords: nodejs electron typescript
sidebar: mydoc_sidebar
folder: mydoc
---

{% include note.html content="이전 글에 이어서 작성된 글입니다." %}

이제 기본 설정이 완료되었다면 typescript를 설치한다.
```bash
$ npm i -D typescript
$ npx tsc --init
```
그러면 tsconfig.json 파일이 생성된다. 아래 key를 검색하여 주석을 풀고 build 폴더를 output 디렉토리로 지정해준다.  
```json
{
	...
	"outDir" : "./build"
}
```
그리고 build 폴더에 copy할 수 있도록 라이브러리를 추가한다.  
- copyfiles => file copy for cross platform
- cross-env => setting environment variable for cross platform  

```bash
$ npm i -D copyfiles cross-env
```
그리고 package.json 파일을 열어 아래와 같이 수정해준다.
```json
{
	"main": "build/main.js",
	"scripts" : {
    "compile": "tsc && copyfiles -f index.html build",
		"start": "npm run compile && cross-env DEBUG=true electron .",
	}	
}
```
위 과정은 Typescript를 javascript로 변환하고 그 결과물을 build 밑으로 복사하여 electron을 실행한다. 
main.js를 main.ts로 변경 후 아래 코드를 입력한다.  

```ts
import { app, BrowserWindow } from "electron"; // ES import 

let window;

app.on("ready", () => {
  window = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
    },
  });
  window.loadFile("index.html");
});
```

그리고 아래 명령을 실행한다
```bash
$ npm start
```

### Test Button Click Event 만들어보기 
index.html에서 button을 생성하고 클릭할 경우 alert 창을 띄워보자.  
```html
<body>
  <h1>Hello World!</h1>
  We are using Node.js <span id="node-version"></span>,
  Chromium <span id="chrome-version"></span>,
  and Electron <span id="electron-version"></span>.
  <h1>GhostWorld</h1>
  <button id="btnTest">Current Directory</button>
  <script src="./renderer.js"></script>
</body>
```
button과 renderer.js script를 추가하였다. renderer.js는 root 폴더 밑에 작성한다.
```js
// renderer.js
const onClick = (sel, fn) => document.querySelector(sel).addEventListener('click', fn);
onClick('#btnTest', () => alert(__dirname));
```
그리고 `npm start`를 실행하고 버튼을 클릭해본다.  
최신버전을 사용하고있다면 alert창이 뜨지 않을 수 있다. 아래와 같이 옵션을 변경해야한다.
```ts
window = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
    nodeIntegration: true,
    contextIsolation: false,
    },
});
```
최신 버전에서는 보안 문제로 contextIsolation의 default 값이 true로 되어 있다. 이 경우 main process와 접근은 다이렉트로 허용도지 않고 ipc를 통해서만 가능하다고 한다.
[Stackoverflow 관련Issue](https://stackoverflow.com/questions/66455289/unable-to-use-node-js-apis-in-renderer-process)

나중에 안 사실인데 옵션을 주지 않아도 아래와 같이 html를 수정해도 된다... ㅜㅜ  
이것 또한 보안에 취약하다고 한다.
```html
<meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    script-src 'self' 'unsafe-inline';
    connect-src *
    ">
```

{% include links.html %}