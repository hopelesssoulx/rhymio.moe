---
title: hexo使用pdf.js
date: 2025-08-18 16:10:36
tags: [web, hexo, pdf.js]
---

### 环境

```json
<!-- package.json -->

"hexo": "^7.3.0",
"hexo-theme-fluid": "1.9.8",    hexo主题，应该不影响
```

```folder
├── scaffolds/
├── source/
│   ├── _drafts/
│   ├── _posts/
│       ├── hexo使用pdf-js.md/
│   ├── css/
│   ├── img/
│       ├── hexo使用pdf.js/
│           ├── demo.pdf
│   ├── js/
│       ├── pdfjs-5.4.54-legacy-dist/       pdf.js库
│           ├── build/
│           ├── web/
│           ├── LICENSE
├── _config.yml
└── package.json
```

### 准备

https://mozilla.github.io/pdf.js/
下载 Prebuilt (older browsers)

### 操作

1.  解压到路径 \source\js\pdfjs-5.4.54-legacy-dist\

2.  删除 \source\js\pdfjs-5.4.54-legacy-dist\web\compressed.tracemonkey-pldi-09.pdf 节省空间

3.  修改 \_config.yml

```yml
skip_render:
  - js/pdfjs-5.4.54-legacy-dist/**
```

### 使用

1. 准备 pdf 文件到路径 \source\img\hexo 使用 pdf.js\demo.pdf

2. 文章 md 文件 语句

```md
<iframe src='/js/pdfjs-5.4.54-legacy-dist/web/viewer.html?file=/img/hexo使用pdf.js/demo.pdf' style='width:100%;height:800px'></iframe>
```

3. 多 hexo clean 后 hexo server

### 展示

<iframe src='/js/pdfjs-5.4.54-legacy-dist/web/viewer.html?file=/img/hexo使用pdf.js/demo.pdf' style='width:100%;height:800px'></iframe>
