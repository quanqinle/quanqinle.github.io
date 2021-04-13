---
layout:       post
title:        "Linux | Vim常用操作"
subtitle:     "Vim查、增、减、移动等常用操作"
date:         2011-04-06 12:00:00
updated:      2011-04-06 12:00:00
author:       "权芹乐"
header-img:   "img/home-bg.webp"
catalog:      true
categories:
    - Linux
tags:
    - Linux
    - Vim
---

# 四种模式
vi有两种模式，命令模式和插入模式，vim作为vi的增强版多了两种模式，普通模式和可视模式。

{% mermaid graph LR %}
A(普通模式) -->|i/a...| B(插入模式)
A(普通模式) -->|shift + :| C(命令模式)
A(普通模式) -->|v/V...| D(可视模式)
B -->|Esc| A
C -->|Esc| A
D -->|Esc| A
{% endmermaid %}

可以通过底部状态行看出所处的模式：
* ①普通模式 ，打开文件后所处的就是普通模式
* ②插入模式 ，编辑文件时的模式
* ③命令模式 ，可以查找、替换等十分丰富的操作
* ④可视模式 ，可以实现光标选择整块文字的操作

<!-- more -->

各模式间的转换方法:
* ②③④==>① 按esc键 （所以，没事可以多按按esc）
* ①==>② 按i/a在光标前/后插入；按I/A在行首/尾插入；按o/O在当前行的下/上新建行；按s删除光标所在字符再插入；按S删除光标所在行再插入
* ①==>③ 按分号（shift + :）
* ①==>④ 按v可视模式visual；按V可视行模式visual line；按ctrl+v/V可视列模式visual block。

> 存稿录入，未完待续……
