---
title: Markdown 편집기 (3)
permalink: mydoc_nodejs_electlec04.html
keywords: nodejs electron Markdown
sidebar: mydoc_sidebar
folder: mydoc
---
### 멀티 윈도우 작업
Electron은 두가지 타입의 process가 있는데 main process와 renderer process이다.  
main process는 renderer process는 여러개를 소유할 수 있다.  
이번 코드에서는 윈도우 생성을 함수화시켜 다수의 renderer process를 쉽게 생성할 수 있도록 만들어 본다.

```typescript
// main.ts
const windows = new Set();
```
생성될 renderer window들을 저장할 구조체이다. 

```typescript
// main.ts
const createWindow = exports.createWindow = () => {
  let x, y;
  const currentWindow = BrowserWindow.getFocusedWindow();

  if (currentWindow) {
    const [currentWindowX, currentWindowY] = currentWindow.getPosition();
    x = currentWindowX + 10;
    y = currentWindowY + 10;
  }

  let newWindow = new BrowserWindow({ x, y, show: false ,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      nodeIntegration: true,
      contextIsolation: false,
      enableRemoteModule: true,
    },
  });
  newWindow.loadFile('index.html');
  newWindow.once('ready-to-show', () => {
    newWindow.show();
  });
  newWindow.on('closed', () => {
    windows.delete(newWindow);
    newWindow = null;
  });
  
  windows.add(newWindow);
  require("@electron/remote/main").enable(newWindow.webContents);
  return newWindow;
};
```
윈도우를 생성하는 함수를 만든다. `getFocusedWindow` 함수로 현재 윈도우의 객체를 받아오고 `getPosition`함수로 10px 옆으로 이동후 윈도우가 생성되도록 한다. 그렇지 않으면 동일한 위치에 윈도우가 생성되어 생성유무가 혼동된다.  
그리고 우아한 Open을 위해 생성시 Window의 속성의 Show를 false로 설정하고 `ready-to-show` 이벤트시에 보여줄 수 있도록 설정하면 깜빡임과 같은 현상을 줄일 수 있다.  
그리고 새로운 윈도우는 `Set` 구조체에 `add`한다.  

```typescript
// main.ts
app.on("ready", () => {
  createWindow();
});
```

그리고 우리가 만든 `createWindow`함수를 `ready`시에 호출할 수 있도록 한다.

```typescript
// main.ts
const openFile = exports.openFile = (targetWindow: any, file: string) => {
  // 생략
const getFileFromUser = exports.getFileFromUser = async (targetWindow: any) => {
```

그리고 export 함수들의 매개변수에 window를 받을 수 있도록 추가한다. `BrowserWindow`를 받을 것이지만 타입은 any로 전달받게 된다.

```javascript
// renderer.js
const remote = require('@electron/remote');
const mainWin = remote.require('./main');
const currentWin = remote.getCurrentWindow();

// 생략 ....

openFileButton.addEventListener('click', () => {
    mainWin.getFileFromUser(currentWin);
});
newFileButton.addEventListener('click', () => {
    mainWin.createWindow();
});
```
이전과 달라진 점은 `getFileFromUser`함수에 `currentWin` 객체를 구하여 넘기고 있다. 그리고 New File버튼을 누르면 `createWindow`가 호출되도록 변경하였다.   
이제 실행해보면 New File 버튼을 누를 때마다 새로운 창이 생길 것이다.

<!--
https://github.com/dtychshenko/electron-tabs-sample
https://www.npmjs.com/package/electron-tabs
https://github.com/sourcechord/electron-gridlayout-sample-->
{% include links.html %}
