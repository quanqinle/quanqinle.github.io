---
layout:       post
title:        "JavaScript | Export HTML webpage as a PDF"
subtitle:     "Add a button [Export PDF], and save a webpage to PDF file, only implement with pure JavaScript"
date:         2021-07-01 10:00:00
updated:      2021-07-01 10:00:00
author:       "Quan Qinle"
header-img:   "img/home-bg.webp"
lang:         en
catalog:      true
categories:
    - JavaScript
tags:
    - JavaScript

---

One of my static web pages needed a new feature that is to export the web page as a PDF file, it is good to do this function in pure JavaScript. Here are a few ways I tried.

Let me show the conclusion first: the first one is what I ended up using; the second one is the simplest one, but there's one step that requires users to switch a edit box to [Save as PDF], which may cause confusion and not easy to use; the third one should be the most effective one for keeping the complex style of web pages, but it converts the pages as images which lead to the loss of PDF editable features.

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
In order to keep the original page style for the exported PDF:
1. `css`: Place all the style sheets used on the page in the parameter 
2. `style`: Place custom style (inline CSS and internal CSS) in the parameter

# window.print()
https://www.w3schools.com/jsref/met_win_print.asp

It will open the Print dialog, users have to change the destination printer to “Save as PDF” and hit the “Print” button and the webpage, actually which means the content in `<body>` of the webpage, will download as a PDF document.

```html
<input type="button" value="Print this page" onClick="window.print()">
```

The exported PDF has no typle by default, but several ways can help:
1. Import style sheet, and add media="print"
Configure a style sheet for printing, such as `print.css`, add it to HTML, meanwhile add `media="print"` in `<link>` which can tell the printer to use this style when printing.
```html
<link href="/path/print.css" media="print" rel="stylesheet" />
```
2. When we don't have many styles to change, there is no need to rewrite a new style sheet at all, a [media query](https://developer.mozilla.org/en-US/docs/Web/CSS/@media) can achieve the same effect
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

It converts the page as a canvas image, then exports the images to a PDF file, so it keeps the styles on the page well, but just because of this, you will not be able to copy or edit the content in the PDF as compared to the previous two methods. 
Personally, I don’t like it.

In addition, the following is a secondary development based on jsPDF, which is slightly easier to use:
https://github.com/eKoopmans/html2pdf.js
