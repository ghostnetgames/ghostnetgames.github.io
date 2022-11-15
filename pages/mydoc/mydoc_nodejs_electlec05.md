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


<!--
https://github.com/dtychshenko/electron-tabs-sample
https://www.npmjs.com/package/electron-tabs
https://github.com/sourcechord/electron-gridlayout-sample-->
{% include links.html %}
