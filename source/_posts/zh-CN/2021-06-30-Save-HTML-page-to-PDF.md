---
layout:       post
title:        "JavaScript | 将 HTML 网页保存成 PDF 文件"
subtitle:     "页面增加「导出 PDF 按钮」，保存页面成为 PDF 文件，使用纯 JavaScript 方式实现"
date:         2021-06-30 12:00:00
updated:      2021-06-30 16:00:00
author:       "Quan Qinle"
header-img:   "img/home-bg.webp"
multilingual: false
catalog:      true
categories:
    - JavaScript
tags:
    - JavaScript

---

我的一个静态网页需要增加在线导出为 PDF 的功能，纯 JavaScript 方式比较适合这个使用场景。本文记录下来我尝试的几种方法。

先说下结论，方法一是我最终采用的；方法二最简单，但是有一步操作需要使用者自己切换成「另存为 PDF」，对小白用户不友好；方法三应该对保持复杂样式是最有效的，但它是以图片的方式保存网页，失去了 PDF 可编辑的特性。

<!-- more -->

# printjs
https://printjs.crabbly.com/

```html
<body id="main-body">

<button id="pdfBtn" type="button" onclick="downloadPdf()">
  Print PDF
</button>

<script src="https://printjs-4de6.kxcdn.com/print.min.js"></script>
<link rel="stylesheet" type="text/css" href="https://printjs-4de6.kxcdn.com/print.min.css">

<script>
  function downloadPdf() {
    var x = document.getElementById("pdfBtn");
    x.style.display = "none";
    //printJS('main-body', 'html');
    printJS({ 
      printable: 'main-body', 
      type: 'html', 
      scanStyles: true, 
      header: '<a href="https://blog.quanqinle.com/">check this page online</a>',
      css: ['./style.css', './sheet.css'], 
      style: '.skill-tags {background-color: #eee; padding: 1px 5px; margin: 0 5px 5px 0; display: inline-block;}',
      maxWidth: 2480, 
      targetStyles: ['*'] 
    });
    x.style.display = "inline";
  };
</script>

</body>
```
注意，为了导出的 PDF 保持原页面样式：
1. 将页面用到的样式表都放在参数`css`中
2. 内联样式写在参数`style`中

# window.print()
https://www.w3schools.com/jsref/met_win_print.asp

打开打印预览弹框，将“目标打印机”改为「另存为 PDF」，默认导出页面中 body 里的所有内容。

```html
<input type="button" value="Print this page" onClick="window.print()">
```

默认页面没有样式，有多种方法可以给打印的页面设置样式：
1. 引入方式表 print.css，标记 media="print"
配置一份打印样式表 print.css，引入到 HTML 文档，在 <link> 上加上一个 media="print" 来标识这是打印机才会应用的样式表，这样打印的时候，就会默认将该样式表应用到文档中
```html
<link href="/path/print.css" media="print" rel="stylesheet" />
```
2. 当我们要修改的样式没有很多的时候，其实完全不需要重新写个样式表，只要写上一个媒介查询也可以达到同样的效果，如：
```html
@media print {
  h1 {
    font-size: 20px;
    color: red;
  }
}
```

# jsPDF
https://github.com/MrRio/jsPDF

把页面转成 canvas（图片）再保存，所以它能够很好的保留页面样式，也正因如此，和前两种方式相比，将无法复制或编辑 PDF 中的内容。我个人不喜欢这种方式。

另外，下面是基于 jsPDF 的二次封装，使用起来稍微简单一些：
https://github.com/eKoopmans/html2pdf.js
