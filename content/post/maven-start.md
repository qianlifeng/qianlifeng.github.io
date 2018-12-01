---
title: 使用Maven构建项目
date: 2015-05-28 15:43:00
tags: [Java]
---

自动化构建，从 Maven 开始~

先看官方定义，Maven 被定义为一套可用于管理项目的构建，报告和文档的工具。使用自动构建工具有利于项目的
部署，测试等，让开发人员从繁琐的编译发布中解脱出来。

```
Apache Maven is a software project management and comprehension tool. Based on the concept of a project object model (POM), Maven can manage a project's build, reporting and documentation from a central piece of information.
```

# 建立项目

本着约定甚于配置的原则，Maven 有一套默认的约定项目结构。作为例子，这里创建如下目录结构：

```
--src
  --main
    --java
      --hello
```

在 hello 目录下面，建立两个文件。分别为 HelloWorld.java 和 Greeter.java

```java
package hello;

public class HelloWorld {
    public static void main(String[] args) {
        Greeter greeter = new Greeter();
        System.out.println(greeter.sayHello());
    }
}
```

```java
package hello;

public class Greeter {
    public String sayHello() {
        return "Hello world!";
    }
}
```

# 安装 Maven

安装这部分其实没什么可说的，就是从[Maven 官网](https://maven.apache.org/download.cgi)下载最新的版本，解压到一个地方后，
将此位置加入到系统环境变量里面就可以了。加完了以后，在命令行输入如下命令验证一下即可。

```
mvn -v
```

# 定义 Maven 构建信息

安装 Maven 完成之后你需要在项目的根目录下面定义一个 pom.xml 的文件，此文件里面包含了所有构建这个项目所需要的信息。这里是一个例子：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework</groupId>
    <artifactId>gs-maven</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>hello.HelloWorld</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

- `<modelVersion>` POM 的版本信息，基本都是 4.4.0 不变
- `<groupId>` 项目所属组织的 Id，一般都是类似`com.domain.xxx`的域名倒写
- `<artifactId>` 此项目的 Id
- `<version>` 项目版本号，推荐[Semantic Version](http://semver.org)
- `<packaging>` 项目打包类型

# 编译项目

上面把项目结构和 Maven 都弄好了，下面就是编译了。在项目根目录打开命令行，并执行如下命令：

```
mvn compile
```

如果不出意外，你应该会看到如下的输出信息

```
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building gs-maven 0.1.0
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ gs-maven -
--
[WARNING] Using platform encoding (GBK actually) to copy filtered resources, i.e
. build is platform dependent!
[INFO] skip non existing resourceDirectory d:\github\java\maven_hello_world\src\
main\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ gs-maven ---
[INFO] Nothing to compile - all classes are up to date
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.674 s
[INFO] Finished at: 2015-05-28T15:40:29+08:00
[INFO] Final Memory: 11M/308M
[INFO] ------------------------------------------------------------------------
```

# 依赖包

一个稍微复杂点的项目或多或少会用到第三方的库，如果使用 Maven 来管理这些依赖?  
这里以 Spring 为例，只需要在 pom.xml 的`<dependencies>`声明依赖即可：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.1.6.RELEASE</version>
    </dependency>
</dependencies>
```

下次你再使用`mvn compile`的时候，Maven 会自动帮你下载这些依赖库。
