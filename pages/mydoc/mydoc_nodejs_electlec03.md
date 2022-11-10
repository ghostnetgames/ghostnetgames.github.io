---
title: Markdown 편집기 (2)
permalink: mydoc_nodejs_electlec03.html
keywords: nodejs electron Markdown
sidebar: mydoc_sidebar
folder: mydoc
---
### Interprocess Communication 사용하기
<!--Table로 만드는 것이 나을 듯 -->
Electron은 두가지 타입의 process가 있는데 `main process`와 `renderer process`이다.  

| main process | both process  | renderer process | 
|---|---|---|
|app, autoUpdater, BrowserWindow, contentTracing, dialog, globalShotcut, ipcMain, Menu, MenuItem, powerMonitor, powerSaveBlocker, powerSaveBlocker, protocol, session, webContents, tray|clipboard, crashReporter, nativeImage, screen, shell|desktopCapturer, ipcRenderer, remote, webFrame| 

각 process 별 사용가능한 모듈도 다르다. renderer에서 파일을 접근하지 위해서 main process와 통신이 필요한데, `IPC방식`과 `remote 방식` 두가지가 있다.  
사실 remote방식은 ipc를 추상화하고 있으므로 기초는 ipc라고 할 수 있다.  
그리고 remote 모듈은 Electron에 기본으로 포함된 기능이였으나 Electron ^16.0.5 이후부터 미포함이되었다. 그래서 npm 등을 통해 별도 Package 설치가 필요하다. 

그런 이유로 책에 나온 코드는 동작하지 않으며 아래 package를 설치해야한다.
[@electron/remote 설치](https://github.com/electron/remote/blob/main/README.md)

```typescript
//main.ts
const { app, BrowserWindow, dialog } = require('electron'); // ES import 
const path = require('path');
const fs = require('fs');
require('@electron/remote/main').initialize();
```
main에서 remote 패키지를 초기화시켜야 한다. 

```typescript
app.on("ready", () => {
  win = new BrowserWindow({
    // 생략
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      nodeIntegration: true,
      contextIsolation: false,
      enableRemoteModule: true,
    },
  });
  require("@electron/remote/main").enable(win.webContents);
```

그리고 윈도우를 생성할때 `enableRemoteModule: true`를 추가한다. 그리고 `webContents`를 등록한다. [webContents](https://www.electronjs.org/docs/latest/api/web-contents)[한글](https://tinydew4.github.io/electron-ko/docs/api/web-contents/)는 웹페이지의 렌더링과 관리를 책임진다. 


```typescript
  exports.getFileFromUser = async () => {
    const files = await dialog.showOpenDialog(win, {
      properties: ['openFile'],
      filters: [
        { name: 'Markdown Files', extensions: ['md', 'markdown'] },
        { name: 'Text Files', extensions: ['txt'] },
      ]
    });
    if (files) { openFile(files.filePaths[0]); }
```

그리고 renderer process에 의해 불려질 `getFileFromUser`를 정의한다. 외부 모듈에서 접근하기 위해서 `exports`에 변수를 생성한다. 이 함수는 파일을 열기 위한 dialog를 띄운다. dialog 속성에 따라 directory를 열 수도 있고 파일을 열수도 있다. 

```typescript
    const openFile = (file: string) => {
      const content = fs.readFileSync(file).toString();
      win.webContents.send('file-opened', file, content);
    };
```
그리고 파일을 읽고 renderer process에게 ipc를 전달한다.

```javascript
const { ipcRenderer } = require('electron');
const remote = require('@electron/remote').require('./main');
const marked = require('marked');

// 생략 ....

openFileButton.addEventListener('click', () => {
    remote.getFileFromUser();
});
ipcRenderer.on('file-opened', (event, file, content) => {
    markdownView.value = content;
    renderMarkdownToHtml(content);
});
```

이제 markdown 파일이 정상적으로 로드되는지 확인한다.


[ESM 문제](https://devblog.kakaostyle.com/ko/2022-04-09-1-esm-problem/)

{% include links.html %}
