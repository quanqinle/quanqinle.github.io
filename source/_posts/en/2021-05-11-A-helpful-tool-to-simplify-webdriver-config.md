---
layout:      post
title:       "Selenium | Webdrivermanager helps to manange and set driver"
subtitle:    "Automate the management of the drivers, such as chromedriver, geckodriver etc."
date:         2021-05-11 12:00:00
updated:      2021-06-18 15:00:00
author:      "Quan Qinle"
header-img:  "img/home-bg.webp"
lang:         en
catalog:      true
categories:
    - Selenium
tags:
    - Selenium

---

Before using Selenium, we need to make sure we already have a driver that matches the browser version under test, if not, we have to go ahead to the official website and download the driver we need. However, the browsers are updated so frequently in nowdays, downloading drivers become frequent, so it turns to be a chore. If you're running in hub-node mode with many browsers, this is even more uncomfortable.

I recently found a project which can solve these problems well.

<!-- more -->

# Function

`WebDriverManager` has the following functions:
+ Automatically download the required version of the `driver`
+ Save setting the environment variable for the driver path, that is, no need to write explicitly `System.setProperty("webdriver.chrome.driver", "C:\\chromedriver.exe")`

# Usage

## pom.xml
For Maven project with JDK 8+
```xml
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>4.4.3</version>
    <scope>test</scope>
</dependency>
```

## in tests
Before `new driver()`, do setting like this:
```java
WebDriverManager.chromedriver().setup();

WebDriver driver = new ChromeDriver();
```

The drivers managers for various browsers can be used as follows:
```java
WebDriverManager.chromedriver().setup();
WebDriverManager.firefoxdriver().setup();
WebDriverManager.edgedriver().setup();
WebDriverManager.operadriver().setup();
WebDriverManager.phantomjs().setup();
WebDriverManager.iedriver().setup();
WebDriverManager.chromiumdriver().setup();
```

But sometimes we want the process of creating a driver to be adaptable to different browsers, ie parameterized, which can be like this:
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

# More details

The above is only the most basic and useful functions for me, `WebDriverManager` has still many other rich features, such as:
+ Set proxy
+ Set agent
+ Set specific browser and driver version
+ and so on...

For more information, please visit the offical website [WebDriverManager](https://github.com/bonigarcia/webdrivermanager)
