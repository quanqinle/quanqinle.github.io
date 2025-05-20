---
layout:       post
title:        "Selenium | Use JavaScript to solve the difficult page operations"
subtitle:     "When element operations don't work as expected, consider using JavaScript"
date:         2020-09-08
updated:      2021-07-20
author:       "Quan Qinle"
lang:         en
catalog:      true
tags:
    - Selenium
    - JavaScript

---

# JavascriptExecutor
It is Javascript executor.

`arguments[i]` is a placeholder for JS script to pass arguments, and `i` starts from 0.

<!-- more -->

# Scroll webpage

Several ways of scrolling pages:
```java
JavascriptExecutor jse = (JavascriptExecutor)driver;

// Scroll to a element (tips: especially useful when elements are invisible or blocked)
jse.executeScript("arguments[0].scrollIntoView();", element);

// Scroll down
jse.executeScript("window.scrollBy(0, 500)", "");
jse.executeScript("scroll(0, 500);");

// Scroll to buttom
jse.executeScript("window.scrollTo(0, document.body.scrollHeight)");

// height of dynamic element
double ImageHeight = eachtile.getSize().getHeight();
double f = 1.04*ImageHeight;
((JavascriptExecutor)driver).executeScript("window.scrollBy(0,arguments[0]);", -f);
```


# Get text inside elements
```HTLM
<div class="item-price">100 Yuan</div>

<div class="item-price">
  <span>200 Yuan</span>
  300 Yuan
</div>
```

div may or may not have span, anyway I don't want the text inside span, that is, it is expected to get `100 Yuan` and `300 Yuan`.
```java
WebElement element = driver.findElement(By.xpath("//div[contains(@class, 'item-price')]"));

JavascriptExecutor jse = (JavascriptExecutor)driver;
jse.executeScript("return arguments[0].lastChild.textContent;", element);
```


# Click invisible item from a very long list
```java
public void selectListByJS() {
    JavascriptExecutor js = (JavascriptExecutor) driver;
    String css;
    css = "document.querySelectorAll('.select-menu')[0].querySelector('.select-option:nth-child(24)').click();";
    js.executeScript(css);
}


// Another Solution:
// modify attribute to make it visible
// tip: I haven't check this solution out yet :)
String strJs = "document.getElementsByClassName('arguments[0]').style.height='auto'; document.getElementsByClassName('arguments[0]').style.visibility='visible';";
```

# Click element
```java
/**
 * tips: Suitable for Click () failure
 *
 * @param element
 */
public void clickByJS(WebElement element) {
	JavascriptExecutor jse = (JavascriptExecutor)driver;
	jse.executeScript("arguments[0].click()", element);
}
```

# Reference
https(:)//www(.)guru99(.)com/execute-javascript-selenium-webdriver.html
