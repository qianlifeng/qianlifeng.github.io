---
title: Nodejs与Java的性能比较
date: 2015-06-03 15:41:17
---

最近遇到一个项目需要提供这样一种 RESTful 的查询接口：

1. 接口比较独立，内部没有复杂的业务逻辑，基本就是查询数据库
2. 接口查询量会非常大，所以希望能有很好的吞吐量

因为接口比较独立，没有业务逻辑负担。所以我们可以选择的方向就比较多，其中两个方向是 Nodejs 和 Java。Java 是目前项目的主要开发语言，考虑 Nodejs 是看中了他的非阻塞的异步 IO。
而本篇文章的目的就是模拟这两种语言在我们真实使用场景下的表现。

# 吞吐量与 PV

开始之前，我们需要先了解几个名词，这对我们后面分析比较数据会有一定帮助。

- **吞吐量**：每秒钟完成的请求数。
- **PV**：PV 是 page view 的简写。PV 是指页面的访问次数，每打开或刷新一次页面，就算做一个 pv。

他们之间存在如下的简单换算：

```
每台服务器每秒处理请求的数量=((80% × 总PV量)/(24小时 × 60分 × 60秒 × 40%)) / 服务器数量。
```

其中关键的参数是 80%、40%。表示一天中有 80%的请求发生在一天的 40%的时间内。24 小时的 40%是 9.6 小时，有 80%的请求发生一天的 9.6 个小时当中。
所以说，如果你的网站每秒能处理 600 个请求，那么你大概能抗住每日 2500W 的 PV 请求。当然这里只是理想情况下的简单换算，真实环境可能会与这个值有出入。

# 一些规则

在进行比较之前，我们需要先确定一些规则：

- 双方尽量使用最原生最简单的实现，排除第三方库的性能干扰。
- 在同一台机器上进行测试（CentOS, 130G 内存，Xeno E5-2640 v3 @ 2.6GHz 32 核）
- 对同一数据集进行查询 (包含 200W 记录的表)
- 测试流程为：请求来了之后，程序会从 10 个候选的 ID 中随机抽取一个进行数据库查询，并将查询的结果返回到页面
- 测试环境都是默认配置

# 准备数据库

首先，使用如下 python 脚本 fake 出我们的测试数据。我这里生成了一张 200W 行数据的表。后面就针对该表进行查询

```python
# encoding: utf-8
#!/usr/bin/python

import MySQLdb
import uuid


def InsertRow(db):
    cursor = db.cursor()
    sql = "insert into purch_sku_master (mem_id,mem_code,sku_id,sku_code,item,sku_name,sku_abbr) values ('main','main','sku-{}','SKU0000009837','ITEM{}','卫衣外套_MQF1213020_瓷蓝_{}','半开襟线衫{}');".format(
        str(uuid.uuid4()),
        str(uuid.uuid4()),
        str(uuid.uuid4()),
        str(uuid.uuid4())
    )
    cursor.execute(sql)


db = MySQLdb.connect("localhost","","","test" )
for i in xrange(1,2000000):
    print i
    InsertRow(db)
    if i % 1000 == 0:
        db.commit()
db.commit()
db.close()
```

# Nodejs

接着编写 Nodejs 端的测试代码，先看目录结构：

```
--nodejs
    --package.json
    --httpserver.js
```

`package.json`里面定义了该项目的一些信息与依赖，因为我们使用的是 mysql 数据库，所以这里会依赖一个 mysql 的包：

```json
{
  "name": "test",
  "version": "0.1.0",
  "description": "A testing package",
  "dependencies": {
    "mysql": "2.7.0"
  },
  "main": "index",
  "scripts": {
    "start": "node HttpServer.js"
  }
}
```

下面是 Nodejs 的测试代码：

```js
var http = require("http");
var mysql = require("mysql");
var connection = mysql.createConnection({
  host: "localhost",
  user: "",
  password: "",
  database: "test"
});

http
  .createServer(function(request, response) {
    query(function(rows) {
      response.writeHead(200, { "Content-Type": "text/plain" });
      response.write(rows[0].sku_id);
      response.end();
    });
  })
  .listen(8888);

function query(callback) {
  var skus = [
    "sku-0000bcd5-c1f7-4460-aaf8-7ee1ab10d409",
    "sku-000f1583-8026-407f-92e2-b6b67c5447e5",
    "sku-000fa572-8f67-4e37-9f2f-52f9bb82a229",
    "sku-0017c850-818b-4bb8-922b-59988b279679",
    "sku-0019ca10-fdb8-490c-a1c7-971b840d0d5f",
    "sku-0019ef1e-0363-41b8-87df-f0f4f51453d0",
    "sku-001a1dd6-cc26-4bfe-a81c-ee952bf024d1",
    "sku-001a475f-45df-4df2-b8bb-643e022b4c72",
    "sku-001ab267-18c6-43b4-915f-104eef32c336",
    "sku-0020b18b-8e09-4d3d-8920-97b27e2d11c4"
  ];

  var sku = skus[Math.floor(Math.random() * 6) + 1];
  connection.query("select * from purch_sku_master where sku_id = '" + sku + "'", function(err, rows, fields) {
    callback(rows);
  });
}
```

# Java

根据第一条规则，我这里没有使用 SpringMVC 类似的框架，而是直接使用了 Servlet 进行测试。直接看我实现的 Servlet 类：

```java
package com.scottqian.javaPerformanceTest;

import java.io.IOException;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@SuppressWarnings("serial")
public class ServletDemo extends HttpServlet {

    private List<String> skus;

    public ServletDemo() {
        skus = new ArrayList<String>();
        skus.add("sku-0000bcd5-c1f7-4460-aaf8-7ee1ab10d409");
        skus.add("sku-000f1583-8026-407f-92e2-b6b67c5447e5");
        skus.add("sku-000fa572-8f67-4e37-9f2f-52f9bb82a229");
        skus.add("sku-0017c850-818b-4bb8-922b-59988b279679");
        skus.add("sku-0019ca10-fdb8-490c-a1c7-971b840d0d5f");
        skus.add("sku-0019ef1e-0363-41b8-87df-f0f4f51453d0");
        skus.add("sku-001a1dd6-cc26-4bfe-a81c-ee952bf024d1");
        skus.add("sku-001a475f-45df-4df2-b8bb-643e022b4c72");
        skus.add("sku-001ab267-18c6-43b4-915f-104eef32c336");
        skus.add("sku-0020b18b-8e09-4d3d-8920-97b27e2d11c4");
    }

    private String getRandomSku() {
        return skus.get(randInt(0, 9));
    }

    public static int randInt(int min, int max) {
        Random rand = new Random();
        return rand.nextInt((max - min) + 1) + min;
    }

    public void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        PrintWriter pw = res.getWriter();
        res.setContentType("text/plain");
        try {
            Class.forName("com.mysql.jdbc.Driver");
            Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "", "");
            Statement stmt = con.createStatement();

            String sql = "select * from purch_sku_master where sku_id = '" + getRandomSku() + "'";

            ResultSet rs = stmt.executeQuery(sql);
            while (rs.next()) {
                pw.println(rs.getString("sku_id"));
            }

            stmt.close();
            con.close();
        } catch (Exception e) {
            pw.write(e.getMessage());
        }
        pw.close();
    }
}
```

下面是 web.xml 的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <servlet>
        <servlet-name>HelloWorldServlet</servlet-name>
        <servlet-class>com.scottqian.javaPerformanceTest.ServletDemo</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloWorldServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

最后将上面的部分放到 Tomcat8 下面运行即可。

# 性能测试工具 Jmeter

我是用了 Jmeter 工具进行性能测试。首先，需要建立一个线程组：
![](/Images/nodejs-vs-java-performance/1.png)
然后设置一些关于线程组的变量。
![](/Images/nodejs-vs-java-performance/2.png)
在线程组之下，还需要设置一个 HTTP 请求 sampler，然后在这里面设置需要请求的 url
![](/Images/nodejs-vs-java-performance/3.png)
最后，添加一个聚合报告，用于查看测试的结果
![](/Images/nodejs-vs-java-performance/4.png)

# 测试结果

### 100 并发

| Label  | # Samples | Average | Median | 90% Line | 95% Line | 99% Line | Min | Max  | Error % | Throughput | KB/sec |
| ------ | --------- | ------- | ------ | -------- | -------- | -------- | --- | ---- | ------- | ---------- | ------ |
| java   | 9369      | 224     | 200    | 290      | 345      | 529      | 79  | 1371 | 0.25%   | 182.5      | 40.0   |
| nodejs | 9322      | 315     | 297    | 367      | 426      | 580      | 51  | 967  | 0.30%   | 185.1      | 35.8   |
| 总体   | 18691     | 269     | 287    | 338      | 392      | 542      | 51  | 1371 | 0.27%   | 364.1      | 75.1   |

### 200 并发

| Label  | # Samples | Average | Median | 90% Line | 95% Line | 99% Line | Min | Max   | Error % | Throughput | KB/sec |
| ------ | --------- | ------- | ------ | -------- | -------- | -------- | --- | ----- | ------- | ---------- | ------ |
| java   | 11376     | 330     | 246    | 420      | 556      | 1581     | 34  | 10959 | 0.15%   | 252.1      | 54.8   |
| nodejs | 11278     | 401     | 327    | 506      | 632      | 1542     | 13  | 9968  | 0.13%   | 264.2      | 50.1   |
| 总体   | 22654     | 365     | 312    | 485      | 601      | 1548     | 13  | 10959 | 0.14%   | 502.0      | 102.1  |

### 300 并发

| Label  | # Samples | Average | Median | 90% Line | 95% Line | 99% Line | Min | Max  | Error % | Throughput | KB/sec |
| ------ | --------- | ------- | ------ | -------- | -------- | -------- | --- | ---- | ------- | ---------- | ------ |
| java   | 17017     | 403     | 374    | 568      | 662      | 1252     | 21  | 1964 | 0.09%   | 246.3      | 53.2   |
| nodejs | 16850     | 430     | 400    | 565      | 624      | 945      | 55  | 4008 | 0.14%   | 278.7      | 52.8   |
| 总体   | 33867     | 417     | 390    | 566      | 637      | 1066     | 21  | 4008 | 0.12%   | 490.3      | 99.4   |

### 500 并发

| Label  | # Samples | Average | Median | 90% Line | 95% Line | 99% Line | Min | Max  | Error % | Throughput | KB/sec |
| ------ | --------- | ------- | ------ | -------- | -------- | -------- | --- | ---- | ------- | ---------- | ------ |
| java   | 20785     | 801     | 798    | 1121     | 1188     | 1385     | 84  | 2388 | 0.24%   | 243.5      | 53.4   |
| nodejs | 20609     | 664     | 647    | 859      | 924      | 1076     | 7   | 1999 | 0.29%   | 274.5      | 53.0   |
| 总体   | 41394     | 733     | 705    | 1031     | 1127     | 1297     | 7   | 2388 | 0.26%   | 484.8      | 99.9   |

先解释一下上面的几个名词：

- **Samples**：表示你这次测试中一共发出了多少个请求
- **Average**：平均响应时间
- **Median**：中位数，也就是 50％ 用户的响应时间
- **90% Line**：90％ 用户的响应时间
- **Min**：最小响应时间
- **Max**：最大响应时间
- **Error%**：本次测试中出现错误的请求的数量/请求的总数
- **Throughput**：吞吐量——默认情况下表示每秒完成的请求数（Request per Second）
- **KB/Sec**：每秒从服务器端接收到的数据量

![](/Images/nodejs-vs-java-performance/compare.png)
从上面的分析可以得到一下一些结论：

1. nodejs 的平均吞吐量比 java 多了**8%**左右。
2. 在大并发情况下，nodejs 的错误率会比 java 略微高一点
3. 上了 200 并发之后，java 的吞吐量有略微下降趋势。猜想后面并发如果更大，java 的吞吐量应该会更差

另外，有一个有趣的现象是当刚开始运行测试的时候，nodejs 的吞吐量是一下子窜到 200 多。而 java 则是慢慢一点一点涨上去的。nodejs 的异步非阻塞模型应该帮了不少忙。

最后，测试工程[下载](/Images/nodejs-vs-java-performance/nodejsVSjava.zip)
