---
title: Electron VSCode로 시작하기
permalink: mydoc_nodejs_electronvscode.html
keywords: nodejs electron VisualStudioCode
sidebar: mydoc_sidebar
folder: mydoc
---

## Electron 시작하기
Electron은 Node.js기반이기 때문에 먼저 Node.js를 설치해야 한다.  
소스코드는 아래 git을 통해 받을 수 있다.
> [Electron Quick Start](https://github.com/electron/electron-quick-start)  
> [Electron + Vue 강좌](https://fkkmemi.github.io/electron/electron-00-intro/)


```bash
# Clone this repository
git clone https://github.com/electron/electron-quick-start
# Go into the repository
cd electron-quick-start
# Install dependencies
npm install
# Run the app
npm start
```

#### 수동으로 코드 만들기

```bash
# Go into the repository
$ cd electron-quick-start
# Install dependencies
$ npm init
# 정보 입력하기
package name: (electron) electronEx
Sorry, name can no longer contain capital letters.
package name: (electron) electron_ex
version: (1.0.0)
description:
entry point: (index.js) main.js
test command: test
git repository:
keywords:
author:
license: (ISC)
About to write to C:\Users\root\source\repos\nodejs\electron\package.json:

{
  "name": "electron_ex",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "scripts": {
    "test": "test"
  },
  "author": "",
  "license": "ISC"
}


Is this OK? (yes) yes
```
위 방식으로 하면 빈 폴더에서 기본적인 package.json을 생성할 수 있다. 우리에게 필요한 파일은 아래와 같다.  

- package.json - 앱에 대한 파일 정보와 속성 및 의존성을 기록합니다. vs의 project파일과 같습니다.
- main.js - 앱을 시작하고 html을 렌더링하는 브라우저 창을 만듭니다. package.json에서 파일 이름을 변경할 수 있습니다. 
- index.html - 랜더링할 웹페이지이며 앱의 렌더러 프로세스입니다.
- preload.js - 렌더러 프로세스가 로드되기 전에 실행되는 콘텐츠 스크립트입니다.

그리고 필요한 모듈인 electron을 설치합니다.
```bash
$ npm install --save-dev electron
```
그러면 json파일에 devDependencies가 추가되고 node_modules폴더와 그 하위 폴더가 생성됨을 확인할 수 있다.
```json
  "devDependencies": {
    "electron": "^21.2.0"
  }
```
그리고 Electron을 실행할 수 있도록 scripts 구성 필드에 다음과 같이 명령어를 추가한다. 아마도 npm init으로 만들었다면 이미 test 필드가 있을 것이다.
```json
{
  "scripts": {
    "start": "electron ."
  }
}
```
`start` 명령을 사용하면 개발 모드에서 앱을 열 수 있다.
```bash
$ npm start
```
아마 실행할 앱이 없다고 오류창이 뜨면 정상적으로 동작한 것이다. 

### Main
Electron 앱의 진입점은 `main` 스크립트이다. 이 스크립트는 전체 Node.js 환경에서 실행되는 기본 프로세스를 제어하며 윈도우 생성과 종료, 인터페이스를 보여주거나 렌더러 프로세스들을 관리한다.  
Electron은 앱스캐폴딩 단계에서 구성해야할 정보를 `package.json`파일의 `main`필드에서 설정된 파일을 찾는다.  
일단 빈 `main.js`파일을 생성한다. 다시 `start`를 하면 오류창이 뜨지 않으나 아무 코드도 없기 때문에 아무것도 하지 않는다.


### Web
Electron 앱이 시작하면서 로드할 콘텐츠를 만들어야한다. electron에서는 각 창은 로컬 html이나 `원격 url`에서 로드할 수 있다.  
여기서는 로컬에서 `index.html`를 생성하여 테스트한다.
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.
  </body>
</html>
```

### Browser
이제 웹페이지를 생성했을므로 응용 프로그램 창에 로드할 수 있다. 그렇게 하려면 두개의 Electron 모듈이 필요하다.  
- `app`: 앱 Event의 lifecycle를 제어하는 모듈이다.
- `BrowserWindow`: 응용 프로그램 창을 만들고 관리하는 모듈이다.  

메인 프로세스가 Node.js를 실행하기 때문에 스크립트 첫 줄에서 `CommonJS` 모듈로 가져올 수 있다.
```javascript
const { app, BrowserWindow } = require('electron')
```
그런 다음 새 인스턴스를 생성하는 `createWindow()`를 추가한다.
```js
const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600
  })

  win.loadFile('index.html')
}
```
다음으로 `createWindow()`함수를 호출하도록 해야한다.  
Electron에서 브라우저 창은 모듈의 `ready` 이벤트가 발생한 후에만 생성할 수 있다. `app.whenReady()` api를 사용하여 이 이벤트 발생시 함수를 trigger할 수 있다. 이 api는 Promise방식으로 수행된다.

```js
app.whenReady().then(() => {
  createWindow()
})
```

### Window의 Lifecycle
Electron은 다양한 OS Platform을 지원하고 있다. 앱 화면은 OS마다 다르게 동작하며 Electron은 개발자에게 차이를 해결하도록 하고 있다. 예를들어 global 변수인 `process.platform` 속성을 사용하여 특정 운영체제용 코드를 실행할 수 잇다.

#### 모든 창이 닫히면 앱 종료 (windows & linux)
Windows 및 Linux에서 모든 창을 종료하면 일반적으로 응용 프로그램이 완전히 종료된다.  
이를 구현하려면 `app`모듈의 `window-all-closed` 이벤트를 수진하면 `app.quit()`를 호출하도록한다. macOS의 경우는 종료하지 않는다. 

#### 아무 것도 열려 있지 않으면 창 열기 (macOS)
Windows와 Linux는 창이 열려있지 않으면 종료되지만 macOS 앱은 일반적으로 창이 열리지 않아도 계속 실행된다. 이 기능을 구현하려면 `activate` 이벤트를 수신하고 Window창이 있는지 확인한 뒤 열린 창이 없으면 `createWindow()` 를 호출한다.  
이벤트 전에는 창을 만들수 없기 때문에 초기화한 된 `ready` 이벤트를 수신 해야한다.

```js
app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})
```


### Preload
Renderer와 main과의 통신에서 nodeintegration에서 보안에 취약점이 생겼다.  
따라서 version 5부터는 preload.js를 통해 통신을 분리했다고 한다.  
> [관련 issue](https://stackoverflow.com/questions/57807459/how-to-use-preload-js-properly-in-electron)

```js
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency])
  }
})
```
좀 더 공부가 필요한 부분이다.   
preload를 추가했다면 main.js에도 추가해줘야할 코드가 있다.

```js
// include the Node.js 'path' module at the top of your file
const path = require('path')

// modify your existing createWindow() function
const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  win.loadFile('index.html')
}
// ...
```
- `__dirname`은 현재 실행중인 스크립트(우리 예제에서는 root)를 의미한다.
- `path.join`은 여러 경로를 결합하여 모든 플랫폼에서 작동하는 결함된 경로를 생성한다.


{% include links.html %}
