---
title: Markdown 편집기 (4)
permalink: mydoc_nodejs_electlec05.html
keywords: nodejs electron Markdown file
sidebar: mydoc_sidebar
folder: mydoc
---
### 파일을 읽고 저장하자.
#### 파일 원본 보존하기
파일이 새로 열리게 되면 위치와 내용을 저장해둔다. 이는 수정 후 저장하지 않을 경우 복구하기 위함이다. `markdownView`는 text를 입력할 수 있는 `textarea`의 id이다. textarea에 내용을 업데이트하고 markdown을 html로 변환하는 함수인 `renderMarkdownToHtml`을 호출한다. 

```js
// renderer.js
ipcRenderer.on('file-opened', (event, file, content) => {
    filePath = file; 
    originalContent = content;

    markdownView.value = content;
    renderMarkdownToHtml(content);
    updateUserInterface();
});
```
#### 윈도우 창 제목 변경하기
새로운 파일이 Open되면 파일 이름으로 윈도우 창 이름을 변경한다. `currentWin`는 main process에게 얻은 BrowserWindow 인스턴스이다.

```js
// renderer.js
const updateUserInterface = (isEdited) => {
    let title = 'Fire Sale';
    if (filePath) { title = `${path.basename(filePath)} - ${title}`; }
    if (isEdited) { title = `${title} (Edited)`; }

    currentWin.setTitle(title);
    currentWin.setDocumentEdited(isEdited);

    saveMarkdownButton.disabled = !isEdited; 
    revertButton.disabled = !isEdited; 
};
```
`path.basename`은 긴 파일 path에서 파일명만 추출해주는 함수이다. 추출된 파일이름과 앱이름을 조합하여 `title`을 생성한다. 그리고 `setTitle` 함수를 통해 제목을 설정한다. 그리고 파일 수정이 없었으므로 save button을 disable 한다.

```js
// renderer.js
markdownView.addEventListener('keyup', (event) => {
    const currentContent = event.target.value;
    renderMarkdownToHtml(currentContent);
    updateUserInterface(currentContent !== originalContent);
});
```
`keyup` event를 등록한다. 키보드의 버튼이 입력될때마다 markdown은 html로 변환되고 `edited ` flag를 업데이트한다. 그러면 save 버튼이 활성화 된다.

#### 파일 저장하기

파일을 저장할 수 있는 권한은 main process에 있다. 따라서 저장할 데이터를 renderer process에서 main process에 전달하고 main process는 해당 데이터를 저장한다. 여기서는 markdown과 html 두가지 타입으로 저장하는 방법을 제공한다. 

```ts
// main.ts
const saveHtml = exports.saveHtml = async (targetWindow: any, content: string) => {
  const files = await dialog.showSaveDialog(targetWindow, {
    title: 'Save HTML',
    defaultPath: app.getPath('documents'),
    filters: [
      { name: 'HTML Files', extensions: ['html', 'htm'] }
    ]
  });
  if (!files) return;
  fs.writeFileSync(files.filePath, content);
};
```  
`showSaveDialog`는 저장을 위한 dialog이다.  
다음 옵션을 지정할 수 있다.
- title: dialog의 제목을 지정한다. 하지만 macOS는 나타나지 않는다.
- defaultPath: 저장을 위한 기본 디렉토리를 지정한다.
- buttonLabel: 저장 버튼에 커스텀 Text를 지원한다.
- filters: 파일 종류를 선택하여 해당 타입만 보이도록 한다.

`app.getPath`는 os에서 제공하는 기본 path를 의미한다. `'documents'`는 윈도우에서는 내 폴더, mac에서는 Documents를 의미한다. 그 외에도 추가 path를 제공한다.

```js
// renderer.js
saveHtmlButton.addEventListener('click', () => {
    mainWin.saveHtml(currentWin, htmlView.innerHTML);
});
```

renderer process에서는 html파일을 저장하기 위해 `htmlView`에서 contents를 가져와 main process에게 전송한다.  
markdown 파일도 크게 다르지 않다.

```ts
// main.ts
const saveMarkdown = exports.saveMarkdown = async (targetWindow: any, file: string, content: string) => {
  if (!file) {
    const files = await dialog.showSaveDialog(targetWindow, {
      title: 'Save Markdown',
      defaultPath: app.getPath('documents'),
      filters: [
        { name: 'Markdown Files', extensions: ['md', 'markdown'] }
      ]
    });

    if (!files) return;
    fs.writeFileSync(files.filePath, content);
    openFile(targetWindow, files.filePath);
  }
};
```  

다른점은 `openFile`을 재호출하고 있는데 이는 os의 최근 문서나 현재 열린 문서에 대한 정보를 업데이트하기 위해 호출한다. 

```js
// renderer.js
saveMarkdownButton.addEventListener('click', () => {
    mainWin.saveMarkdown(currentWin, filePath, markdownView.value);
});
```

#### Reverting files
Revert를 하면 처음 Open했었을 때 저장한 Contents Data를 불러온다.  
수정중에 아차 싶으면 다시 Revert 버튼을 누른다. Undo는 아니고 초기화에 가까운 기능이다.

```js
// renderer.js
revertButton.addEventListener('click', () => {
    markdownView.value = originalContent;
    renderMarkdownToHtml(originalContent);
});
```

#### Drag and Drop
파일을 Open할 때 메뉴를 이용하여 Open하는 방법이 일반적이지만 가끔 파일을 드레그하여 Open하는 경우도 있다. 이 기능을 구현하도록한다.  
grag&drop 관련 Event를 지원하고 있기 때문에 해당 Event에 연결만 하면 된다.

- dragstart
- dragover
- dragleave
- drop

먼저 window 창의 top event를 비활성화 해야한다. 기본 액션은 웹브라우저를 기준으로 하고 있기 때문이다. 

```js
// renderer.js
document.addEventListener('dragstart', event => event.preventDefault());
document.addEventListener('dragover', event => event.preventDefault());
document.addEventListener('dragleave', event => event.preventDefault());
document.addEventListener('drop', event => event.preventDefault());
```

그리고 style.css를 통해 visual feedback을 제공한다. 파일을 드래그하여 해당 창위에 올려놨을 때 테두리나 배경색을 변경하여 맞는 타입의 파일을 드래그하고 있는 지의 정보를 제공한다. 

```css
/* style.css */
.raw-markdown.drag-over {
    background-color: rgb(181, 220, 216);
    border-color: rgb(75, 160, 151);
}

.raw-markdown.drag-error {
    background-color: rgba(170, 57, 57, 1);
    border-color: rgba(255, 170, 170, 1);
}
```

Helper 함수를 작성한다.  
Event를 받아 `dataTransfer` 구조체를 통해 원하는 정보를 구한다. 배열 첫번째로 되어 있는 이유는 다수의 파일을 드레그할 수 있기 때문이다. 이 예제에서는 다수의 파일에 대해서 처리하지 않는다. 

```js
// renderer.js
const getDraggedFile = (event) => event.dataTransfer.items[0];
const getDroppedFile = (event) => event.dataTransfer.files[0];
const fileTypeIsSupported = (file) => {
    return ['text/plain', 'text/markdown'].includes(file.type);
};
```

이제 `dragover`와 `dragleave` event를 작성한다. dragover는 마우스로 파일을 윈도우 창에 올리는 행동을 한 경우이다. 이 경우 앞서 작성한 style 태그가 적용될 수 있도록 한다. 그리고 dragleave이벤트는 떠났을 때이고 이 때  style 태그를 제거한다. dom객체의 classList는 style 태그의 속성을 가리킨다.

```js
// renderer.js
markdownView.addEventListener('dragover', (event) => {
    const file = getDraggedFile(event);
    if (fileTypeIsSupported(file)) {
        markdownView.classList.add('drag-over');
    } else {
        markdownView.classList.add('drag-error');
    }
});
markdownView.addEventListener('dragleave', () => {
    markdownView.classList.remove('drag-over');
    markdownView.classList.remove('drag-error');
});
```

그리고 `dragdrop` 이벤트를 작성한다. 파일 형식이 맞다면 해당 파일을 Open하도록 하고 그렇지 않다면 alert창을 통해 알려준다. 

```js
// renderer.js
markdownView.addEventListener('drop', (event) => {
    const file = getDroppedFile(event);
    if (fileTypeIsSupported(file)) {
        mainWin.openFile(currentWin, file.path);
    } else {
        alert('That file type is not supported');
    }
    markdownView.classList.remove('drag-over');
    markdownView.classList.remove('drag-error');
});
```

#### 파일 수정 감지하기 
파일을 수정하다보면 다른 tool에 의해 파일이 수정되는 경우가 발생한다. 그럴 경우 수정된 파일로 업데이트를 한다. 그런데 현재 본인이 직접 수정하고 있는데 다른 앱으로 인해 수정되어 덮어쓰여지는 사고가 발생할 수 있다. 그럴땐 alert 창으로 확인하는 코드를 넣으면 된다. 일단 그냥 업데이트 하는 코드를 살펴 본다.

```ts
// main.ts
const openFiles = new Map();
const startWatchingFile = (targetWindow: any, file: string) => {
  stopWatchingFile(targetWindow);
  const watcher = fs.watchFile(file, (event: any) => {
    if (event === 'change') {
      const content = fs.readFileSync(file);
      targetWindow.webContents.send('file-opened', file, content);
    }
  });
  openFiles.set(targetWindow, watcher);
};

const stopWatchingFile = (targetWindow: any) => {
  if (openFiles.has(targetWindow)) {
    openFiles.get(targetWindow).stop();
    openFiles.delete(targetWindow);
  }
};
```

`stopWatchingFile`은 `createWindow` 함수의 closed event시 호출되도록 한다. 그리고 `openFile` 함수에서 `startWatchingFile`을 호출하도록한다.

```ts
// main.ts
  newWindow.on('closed', () => {
    windows.delete(newWindow);
    stopWatchingFile(newWindow);
    newWindow = null;
  });
```

#### 변경사항을 버리기 전에 물어보기

우리가 저장하지 않았는데 저장했다는 착각을 하고 실수로 종료 버튼을 누르면 그 동안 수정했던 문서가 날아갈 수 있다. 따라서 한번 더 물어보는 성의를 보여주자.
`close` 이벤트가 발생했을 때 `showMessageBox`를 통해 확인한다. 이전과 다른점은 `close`와 `closed`라는 점이다. 이 두 이벤트는 트리거 되는 시점이 다르다.

```ts
// main.ts
  newWindow.on('close', (event: any) => {
    if (newWindow.isDocumentEdited()) {
      event.preventDefault();
      dialog.showMessageBox(newWindow, {
        type: 'warning',
        title: 'Quit with Unsaved Changes?',
        message: 'Your changes will be lost if you do not save.',
        buttons: [
          'Quit Anyway', 'Cancel',
        ],
        defaultId: 0,
        cancelId: 1
      }).then((result: any) => {
        if (result.response === 0) newWindow.destroy();
      });      
    }
  });
```

`showMessageBox`는 버튼들을 커스텀할 수 있으며 각 배열에 따라 버튼이 보여지게 된다. 해당 버튼을 클릭하게 되면 버튼의 index id가 리턴된다.  
다음으로 renderer.js를 조금 리펙토링 해보자.

```js
// renderer.js
const renderFile = (file, content) => {
    filePath = file;
    originalContent = content;
    markdownView.value = content;
    renderMarkdownToHtml(content);
    updateUserInterface(false);
};
```

`renderFile`함수는 랜더링에 필요한 함수들을 모아두었다. 이 함수는 `file-opened`와 `file-changed` 이벤트에서 호출하도록 할 것이다. 

```js
// renderer.js
ipcRenderer.on('file-opened', (event, file, content) => {
    if (currentWindow.isDocumentEdited()) {
        const result = remote.dialog.showMessageBox(currentWindow, {
            type: 'warning',
            title: 'Overwrite Current Unsaved Changes?',
            message: 'Opening a new file in this window will overwrite your unsaved changes.Open this file anyway?',
            buttons: [
                'Yes',
                'Cancel',
            ],
            defaultId: 0,
            cancelId: 1
        });
        if (result === 1) { return; }
    }
    renderFile(file, content);
});

ipcRenderer.on('file-changed', (event, file, content) => {
    const result = remote.dialog.showMessageBox(currentWindow, {
        type: 'warning',
        title: 'Overwrite Current Unsaved Changes?',
        message: 'Another application has changed this file. Load changes?',
        buttons: [
            'Yes',
            'Cancel',
        ],
        defaultId: 0,
        cancelId: 1
    });
    renderFile(file, content);
});
```

이것으로 마무리한다. 

{% include links.html %}
