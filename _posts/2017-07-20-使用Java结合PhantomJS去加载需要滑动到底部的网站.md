---
layout: post
title: "使用Java结合PhantomJS去加载需要滑动到底部的网站"
date: 2017-07-20
category: Java
tag: PhantomJS
---

## 背景

有一些网站,滑动条往下滑动的时候页面才加载或者再次获取数据

## 使用PhantomJS来实现自动加载,以www.toutiao.com为例

### 安装phantomjs

把phantomjs放在工程phantomjs文件夹下

### maven需要依赖的包

```xml
        <dependency>
            <groupId>com.github.detro</groupId>
            <artifactId>phantomjsdriver</artifactId>
            <version>1.2.0</version>
        </dependency>
```

### Java代码
```java
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.phantomjs.PhantomJSDriver;
import org.openqa.selenium.phantomjs.PhantomJSDriverService;
import org.openqa.selenium.remote.DesiredCapabilities;

import java.io.File;
import java.util.concurrent.TimeUnit;

/**
 * 测试PhantomJS框架函数
 */
public class PhantomJST {

    public static void main(String[] args) throws Exception{
        String currDir = System.getProperty("user.dir");
        String driverPath = currDir + File.separator + "phantomjs/phantomjs";

        DesiredCapabilities capabilities = DesiredCapabilities.phantomjs();
        capabilities.setCapability(PhantomJSDriverService.PHANTOMJS_EXECUTABLE_PATH_PROPERTY, driverPath);

        PhantomJSDriverService service = PhantomJSDriverService.createDefaultService(capabilities);
        WebDriver webDriver = new PhantomJSDriver(service, capabilities);
        webDriver.manage().deleteAllCookies();
        webDriver.manage().timeouts().pageLoadTimeout(5, TimeUnit.SECONDS);
        webDriver.manage().timeouts().setScriptTimeout(5, TimeUnit.SECONDS);
        webDriver.get("http://www.toutiao.com");
        JavascriptExecutor jse = (JavascriptExecutor) webDriver;

        Long position = (Long)jse.executeScript("return window.scrollY;");

        System.out.println(webDriver.getPageSource());
        System.out.println("***" + position + "***");
        System.out.println("------------------------------------------");


        jse.executeScript("window.scrollTo(0, document.body.scrollHeight);");
        position = (Long)jse.executeScript("return window.scrollY;");
        Thread.sleep(3000);

        System.out.println(webDriver.getPageSource());
        System.out.println("***" + position + "***");
        System.out.println("------------------------------------------");


        jse.executeScript("window.scrollTo(0, document.body.scrollHeight);");
        position = (Long)jse.executeScript("return window.scrollY;");
        Thread.sleep(3000);

        System.out.println(webDriver.getPageSource());
        System.out.println("***" + position + "***");
        System.out.println("------------------------------------------");

        webDriver.quit();
    }
}

```
