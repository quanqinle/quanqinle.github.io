---
layout:      post
title:       "Selenium | webdrivermanager简化driver配置管理"
subtitle:    "自动管理driver，例如chromedriver, geckodriver等"
date:         2021-05-11 12:00:00
updated:      2021-05-12 16:00:00
author:      "Quan Qinle"
header-img:  "img/home-bg.webp"
multilingual: false
catalog:      true
categories:
    - Selenium
tags:
    - Selenium
---

在使用Selenium之前，需要确保已经有了和被测浏览器版本一致的driver，如果没有，就要到官网下载所需的driver。因为现代浏览器更新频繁，下载driver也就变得频繁。这个过程就变成了繁琐、无趣的事情。如果你以hub-node模式运行在很多浏览器的环境，这事儿就更让人难受了。

最近发现一个项目很好地解决了这个问题。

<!-- more -->

# 作用
`WebDriverManager` 具有以下作用：
+ 自动下载所需版本的driver
+ 省去设置driver路径的环境变量，即，无需显示写出`System.setProperty("webdriver.chrome.driver", "C:\\chromedriver.exe")`

# 使用

## pom.xml
Maven项目（JDK 8+）
```xml
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>4.4.3</version>
    <scope>test</scope>
</dependency>
```

## in tests
在`new driver()`前设置
```java
WebDriverManager.chromedriver().setup();

WebDriver driver = new ChromeDriver();
```

其他driver的设置方法
```java
WebDriverManager.chromedriver().setup();
WebDriverManager.firefoxdriver().setup();
WebDriverManager.edgedriver().setup();
WebDriverManager.operadriver().setup();
WebDriverManager.phantomjs().setup();
WebDriverManager.iedriver().setup();
WebDriverManager.chromiumdriver().setup();
```

但有时我们希望创建driver的过程可以适应不同的浏览器，即参数化，可以这样：
```java
import static io.github.bonigarcia.wdm.DriverManagerType.CHROME;

import org.openqa.selenium.WebDriver;
import io.github.bonigarcia.wdm.WebDriverManager;

// ...

DriverManagerType driverType = DriverManagerType.CHROME;

WebDriverManager.getInstance(driverType).setup();
Class<?> driverTypeClass =  Class.forName(driverType.browserClass());
WebDriver driver = (WebDriver) driverTypeClass.getDeclaredConstructor().newInstance();
```

# 更多功能

以上只是对我来说最有用的功能，`WebDriverManager`还有很多其他丰富的功能，列举几个：
+ 设置proxy
+ 设置agent
+ 指定浏览器和driver版本
+ 等等……

更多信息，请查阅官网：[WebDriverManager](https://github.com/bonigarcia/webdrivermanager)
