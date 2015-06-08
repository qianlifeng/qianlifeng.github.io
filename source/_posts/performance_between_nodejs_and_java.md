title: Nodejs与Java的性能比较
category: Java
date: 2015-06-03 15:41:17
---

最近遇到一个项目需要提供这样一种RESTful的查询接口：
1. 接口比较独立，内部没有复杂的业务逻辑，基本就是查询数据库
2. 接口查询量会非常大，所以希望能有很好的吞吐量

因为接口比较独立，没有业务逻辑负担。所以我们可以选择的方向就比较多，其中两个方向是Nodejs和Java。Java是目前项目的主要开发语言，考虑Nodejs是看中了他的非阻塞的异步IO。
而本篇文章的目的就是模拟这两种语言在我们真实使用场景下的表现。

<!-- more -->

#吞吐量与PV
开始之前，我们需要先了解几个名词，这对我们后面分析比较数据会有一定帮助。
* **吞吐量**：每秒钟完成的请求数。
* **PV**：PV是page view的简写。PV是指页面的访问次数，每打开或刷新一次页面，就算做一个pv。

他们之间存在如下的简单换算：  
```
每台服务器每秒处理请求的数量=((80% × 总PV量)/(24小时 × 60分 × 60秒 × 40%)) / 服务器数量。
```
其中关键的参数是80%、40%。表示一天中有80%的请求发生在一天的40%的时间内。24小时的40%是9.6小时，有80%的请求发生一天的9.6个小时当中。
所以说，如果你的网站每秒能处理600个请求，那么你大概能抗住每日2500W的PV请求。当然这里只是理想情况下的简单换算，真实环境可能会与这个值有出入。

#一些规则
在进行比较之前，我们需要先确定一些规则：
* 双方尽量使用最原生最简单的实现，排除第三方库的性能干扰。
* 在同一台机器上进行测试（Intel i7 4710MQ，16G）
* 对同一数据集进行查询 (包含200W记录的表)
* 测试流程为：请求来了之后，程序会从10个候选的ID中随机抽取一个进行数据库查询，并将查询的结果返回到页面

#准备数据库
首先，使用如下python脚本fake出我们的测试数据。我这里生成了一张200W行数据的表。后面就针对该表进行查询
```
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

#Nodejs
接着编写Nodejs端的测试代码，先看目录结构：
```
--nodejs
    --package.json
    --httpserver.js
```
`package.json`里面定义了该项目的一些信息与依赖，因为我们使用的是mysql数据库，所以这里会依赖一个mysql的包：
```
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
下面是Nodejs的测试代码：
```
var http = require("http");
var mysql      = require('mysql');
var connection = mysql.createConnection({
    host     : 'localhost',
    user     : '',
    password : '',
    database: 'test'
});

http.createServer(function(request, response) {
    query(function(rows){
        response.writeHead(200, {"Content-Type": "text/plain"});
        response.write(rows[0].sku_id);
        response.end();
    });
}).listen(8888);

function query(callback){
    var skus = [
        'sku-0000bcd5-c1f7-4460-aaf8-7ee1ab10d409',
        'sku-000f1583-8026-407f-92e2-b6b67c5447e5',
        'sku-000fa572-8f67-4e37-9f2f-52f9bb82a229',
        'sku-0017c850-818b-4bb8-922b-59988b279679',
        'sku-0019ca10-fdb8-490c-a1c7-971b840d0d5f',
        'sku-0019ef1e-0363-41b8-87df-f0f4f51453d0',
        'sku-001a1dd6-cc26-4bfe-a81c-ee952bf024d1',
        'sku-001a475f-45df-4df2-b8bb-643e022b4c72',
        'sku-001ab267-18c6-43b4-915f-104eef32c336',
        'sku-0020b18b-8e09-4d3d-8920-97b27e2d11c4'
    ];

    var sku = skus[Math.floor(Math.random() * 6) + 1];
    connection.query("select * from purch_sku_master where sku_id = '"+sku+"'", function(err, rows, fields) {
        callback(rows);
    });
}
```

#Java
根据第一条规则，我这里没有使用SpringMVC类似的框架，而是直接使用了Servlet进行测试。直接看我实现的Servlet类：
```
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
下面是web.xml的配置：
```
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
最后将上面的部分放到Tomcat8下面运行即可。

#性能测试工具Jmeter
我是用了Jmeter工具进行性能测试。首先，需要建立一个线程组：
![](/Images/nodejs-vs-java-performance/1.png)
然后设置一些关于线程组的变量。我们这里使用了30个线程，每个线程发送1000个请求，立即发送。相当于30000个请求一下子过来，看服务器每秒能处理多少个请求。
![](/Images/nodejs-vs-java-performance/2.png)
在线程组之下，还需要设置一个HTTP请求sampler，然后在这里面设置需要请求的url
![](/Images/nodejs-vs-java-performance/3.png)
最后，添加一个聚合报告，用于查看测试的结果
![](/Images/nodejs-vs-java-performance/4.png)


#测试结果
| Label  | # Samples | Average | Median | 90% Line | 95% Line | 99% Line | Min | Max  | Error % | Throughput | KB/sec |
|--------|-----------|---------|--------|----------|----------|----------|-----|------|---------|------------|--------|
| java   | 31887     | 106     | 71     | 210      | 395      | 693      | 1   | 1310 | 0.16%   | 639.8      | 330.0  |
| nodejs | 31862     | 47      | 40     | 96       | 108      | 132      | 0   | 194  | 0.17%   | 642.8      | 118.6  |
| 总体   | 63749     | 76      | 55     | 125      | 210      | 559      | 0   | 1310 | 0.17%   | 1278.7     | 447.9  |

先解释一下上面的几个名词：
* **Samples**：表示你这次测试中一共发出了多少个请求
* **Average**：平均响应时间
* **Median**：中位数，也就是 50％ 用户的响应时间
* **90% Line**：90％ 用户的响应时间
* **Min**：最小响应时间
* **Max**：最大响应时间 
* **Error%**：本次测试中出现错误的请求的数量/请求的总数
* **Throughput**：吞吐量——默认情况下表示每秒完成的请求数（Request per Second）
* **KB/Sec**：每秒从服务器端接收到的数据量

这里主要看Throughput（吞吐量）这一列，可以看到nodejs和java在测试机器上每秒差不多都能处理640个请求。这个结果比较意外，因为我测试这个的目的实际上想证明Nodejs在这方面会比Java的性能好一点的:(  
但是值得注意的是nodejs的平均响应时间比java小了一倍多（那为啥吞吐量最终还差不多呢？）
