---
title: Markdown 편집기 (1)
permalink: mydoc_nodejs_electlec02.html
keywords: nodejs electron Markdown
sidebar: mydoc_sidebar
folder: mydoc
---

### markdown 편집기 기본 코드
아래 코드를 기반으로 확장해 나간다.
```json
// pacakge.json
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
  "dependencies": {
    "@electron/remote": "^2.0.8",
    "marked": "^4.2.2"
  }
}

```
```html
<!--index.html-->
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>Fire Sale</title>
    <link rel="stylesheet" href="build/style.css" type="text/css">
</head>

<body>
    <section class="controls">
        <button id="new-file">New File</button>
        <button id="open-file">Open File</button>
        <button id="save-markdown" disabled>Save File</button>
        <button id="revert" disabled>Revert</button>
        <button id="save-html">Save HTML</button>
        <button id="show-file" disabled>Show File</button>
        <button id="open-in-default" disabled>Open in Default
            ➥ Application</button>
    </section>
    <section class="content">
        <label for="markdown" hidden>Markdown Content</label>
        <textarea class="raw-markdown" id="markdown"></textarea>
        <div class="rendered-html" id="html"></div>
    </section>
</body>
<script type="text/javascript" src="build/renderer.js"></script>

</html>
```

```css
/* style.css */
html {
    box-sizing: border-box;
}

*, *:before, *:after {
    box-sizing: inherit;
}

html,
body {
    height: 100%;
    width: 100%;
    overflow: hidden;
}

body {
    margin: 0;
    padding: 0;
    position: absolute;
}

body,
input {
    font: menu;
}

textarea,
input,
div,
button {
    outline: none;
    margin: 0;
}

.controls {
    background-color: rgb(217, 241, 238);
    padding: 10px 10px 10px 10px;
}

button {
    font-size: 14px;
    background-color: rgb(181, 220, 216);
    border: none;
    padding: 0.5em 1em;
}

button:hover {
    background-color: rgb(156, 198, 192);
}

button:active {
    background-color: rgb(144, 182, 177);
}

button:disabled {
    background-color: rgb(196, 204, 202);
}

.container {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    min-width: 100vw;
    position: relative;
}

.content {
    height: 100vh;
    display: flex;
}

.raw-markdown,
.rendered-html {
    min-height: 100%;
    max-width: 50%;
    flex-grow: 1;
    padding: 1em;
    overflow: scroll;
    font-size: 16px;
}

.raw-markdown {
    border: 5px solid rgb(238, 252, 250);
    ;
    background-color: rgb(238, 252, 250);
    font-family: monospace;
}
```

```typescript
//main.ts
const { app, BrowserWindow } = require('electron'); // ES import 
const path = require('path');

let win;

app.on("ready", () => {
  console.log('start electron app');
  win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      nodeIntegration: true,
      contextIsolation: false,
    },
  });
  win.webContents.openDevTools()
  win.loadFile("index.html");
});
```

```javascript
// renderer.js
const marked = require('marked');

const markdownView = document.querySelector('#markdown');
const htmlView = document.querySelector('#html');
const newFileButton = document.querySelector('#new-file');
const openFileButton = document.querySelector('#open-file');
const saveMarkdownButton = document.querySelector('#save-markdown');
const revertButton = document.querySelector('#revert');
const saveHtmlButton = document.querySelector('#save-html');
const showFileButton = document.querySelector('#show-file');
const openInDefaultButton = document.querySelector('#open-in-default');

const renderMarkdownToHtml = (markdown) => {
    htmlView.innerHTML = marked.marked(markdown, { sanitize: true });
};

markdownView.addEventListener('keyup', (event) => {
    const currentContent = event.target.value;
    renderMarkdownToHtml(currentContent);
    console.log("key");
});
```


[ESM 문제](https://devblog.kakaostyle.com/ko/2022-04-09-1-esm-problem/)

{% include links.html %}
