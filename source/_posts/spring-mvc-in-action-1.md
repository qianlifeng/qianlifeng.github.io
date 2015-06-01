title: SpringMVC - 初识
date: 2015-06-01 16:06
tags: [Java]
---

写一个最小可运行且无XML配置（看到大坨大坨的XML配置就烦）的Spring MVC程序。

<!--more-->

#建立项目
首先，你需要下载Maven。关于Maven的使用，请参考我写的[这篇入门文章](/2015/05/28/maven-start/)。按照Maven的约定，我们建立如下目录结构：
```
--src
  --main
    |--java
    |  |--com
    |    |--scottqian
    |      |--controllers       /*控制器目录*/
    |      |  --HomeController.java
    |      |--Application.java  /*程序入口*/
    |--resources
       |--templates     /*模板目录*/
          |--home.html
--pom.xml
```

其中，`HomeController.java`内容如下
```
package com.scottqian.controllers;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class HomeController {

    @RequestMapping("/")
    public String Home(@RequestParam(value="name", required=false, defaultValue="World") String name, Model model) {
        model.addAttribute("name", name);
        return "home";
    }
}
```
本身结构就比较清晰了，对MVC有点了解的从字面上看都应该知道它的意思了（先抛开@Controller这种注解不谈）。唯一可能疑惑的地方是`Home`方法的返回值，这里返回的是一个字符串。这个字符串代表了视图的名字，在SpringMVC 2.x时代还可以返回`ModelAndView`，然后在`ModelAndView`里面设置`View`和`Model`，其实效果都一样。  

`Application.java`内容如下：
```
package com.scottqian;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
这个是程序的入口点，可以看到他是一个标准的Java main入口程序。得益于[Spring Boot](http://projects.spring.io/spring-boot/)项目，我们可以以如此优雅的方式启动Spring MVC程序。简单列举一下这样做的直观好处：  
1. 丢弃大段配置文件
2. 内嵌Web服务器，就地运行（忘记WAR文件吧），配合`spring-boot-maven-plugin`的`mvn spring-boot:run`直接从命令行启动程序

`home.html`内容如下：
```
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Hello from Spring MVC</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p th:text="'Hello, ' + ${name} + '!'" />
</body>
</html>
```
模板，没什么好说的了。

`pom.xml`内容如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.scottqian</groupId>
    <artifactId>JPanther</artifactId>
    <version>0.1.0</version>

    <!-- 继承spring-boot-starter-parent，这里面做了很多默认的配置-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.2.3.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
    </dependencies>

    <properties>
        <java.version>1.7</java.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <!-- 为了直接使用 mvn spring-boot:run 启动网站-->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestone</id>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>spring-milestone</id>
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>

</project>
```

#运行项目
有了maven plugin，运行项目真是再简单不过了。直接如下命令搞定：
```
mvn spring-boot:run
```
如果没什么意外，打开localhost:8080就可以看到效果了。So Easy~
