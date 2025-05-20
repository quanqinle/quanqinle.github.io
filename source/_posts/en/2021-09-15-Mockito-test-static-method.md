---
layout:       post
title:        "Test | Use Mockito to mock static method"
subtitle:     ""
date:         2022-08-11 15:00:00
updated:      2022-08-11 15:00:00
author:       "Quan Qinle"
header-img:   "img/home-bg.webp"
lang:         en
catalog:      true
categories:
    - Mockito
    - Unit Testing
tags:
    - Mockito
    - Unit Testing

---

Starting from version 3.4.0, Mockito can already support to mock static method.

Mocking static methods (since 3.4.0)
When using the inline mock maker, it is possible to mock static method invocations within the current thread and a user-defined scope. This way, Mockito assures that concurrently and sequentially running tests do not interfere. To make sure a static mock remains temporary, it is recommended to define the scope within a try-with-resources construct. In the following example, the Foo type's static method would return foo unless mocked:

 assertEquals("foo", Foo.method());
 try (MockedStatic mocked = mockStatic(Foo.class)) {
 mocked.when(Foo::method).thenReturn("bar");
 assertEquals("bar", Foo.method());
 mocked.verify(Foo::method);
 }
 assertEquals("foo", Foo.method());
 
Due to the defined scope of the static mock, it returns to its original behavior once the scope is released. To define mock behavior and to verify static method invocations, use the MockedStatic that is returned.


<!-- more -->

# Datatables Introduction
https://datatables.net/

Introduction from offical website:
> Add advanced interaction controls to your HTML tables the free & easy way

# Installment
在 HTML 中添加`DataTables`的 js 和 css。我使用的是当前最新版本 1.10.25。
```html
<link rel="stylesheet" href="https://cdn.datatables.net/1.10.25/css/jquery.dataTables.min.css">

<script src="https://cdn.datatables.net/1.10.25/js/jquery.dataTables.min.js"></script>
```
