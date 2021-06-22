---
layout:       post
title:        "JUnit | 在 Spring Boot2 低版本中使用 JUnit 5"
subtitle:     "Spring Boot 2.2.0 开始支持 JUnit 5，之前的版本需要手动引入依赖"
date:         2020-06-12
updated:      2020-06-12
author:       "权芹乐"
header-img:   "img/post-bg-beach1.webp"
multilingual: false
catalog:      true
categories:
    - Code
tags:
    - JUnit
    - Spring Boot
    - Unit Testing

---

先说结论：如果你用的`Spring Boot`版本是`2.2.0`或更新，那么，`spring-boot-starter-test`已自带`JUnit 5`，本文对你无用，你可以直接在你的工程中编码了。

下文是`Spring Boot 2`版本低于`2.2.0.RELEASE`时的配置方法。

<!-- more -->

我的环境：
+ Java 8+
+ Maven 3
+ Spring Boot 2.0.7.RELEASE
+ 想使用 JUnit 5，且没有 JUnit 4 的历史用例

配置步骤不复杂，简单来说，从`spring-boot-starter-test`中去掉`JUnit 4`，再引入`JUnit 5`。  
pom.xml 需要修改的部分如下：
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
  <exclusions>
    <exclusion>
      <!-- JUnit updates to version 5 since springboot 2.2-->
      <groupId>JUnit</groupId>
      <artifactId>JUnit</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>org.JUnit.jupiter</groupId>
  <artifactId>JUnit-jupiter</artifactId>
  <version>5.6.2</version>
  <scope>test</scope>
</dependency>


<plugin>
  <!-- unit test: Need at least 2.22.0 to support JUnit 5 -->
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.22.2</version>
</plugin>
```

Tip：`JUnit-jupiter`已包含`JUnit-jupiter-engine`、`JUnit-jupiter-api`、`JUnit-jupiter-params`，所以，只引入它就够了。
