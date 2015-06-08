title: Nodejs与Java的性能比较
category: Java
date: 2015-06-03 15:41:17
---

最近遇到一个项目需要提供这样一种RESTful的查询接口：
1. 接口比较独立，内部没有复杂的业务逻辑，基本就是查询数据库
2. 接口查询量会非常大，所以希望能有很好的吞吐量（每秒查询数）

因为接口比较独立，没有业务逻辑负担。所以我们可以选择的方向就比较多，其中两个方向是Nodejs和Java。Java是目前项目的主要开发语言，考虑Nodejs是看中了他的非阻塞的异步IO。
而本篇文章的目的就是模拟这两种语言在我们真实使用场景下的表现。

<!-- more -->

#一些规则
在进行比较之前，我们需要先确定一些规则：
* 双方尽量使用最原生最简单的实现，排除第三方库的性能干扰。
* 在同一台机器上进行测试
* 对同一数据集进行查询

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
    connection.query("select * from purch_sku_master where sku_id = 'sku-000ba89a-4817-4835-ba55-7606ec7b54ed'", function(err, rows, fields) {
        if (err) throw err;
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

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@SuppressWarnings("serial")
public class ServletDemo extends HttpServlet {

    public void doGet(HttpServletRequest req, HttpServletResponse res) throws IOException {
        PrintWriter pw = res.getWriter();
        res.setContentType("text/plain");
        try {
            Class.forName("com.mysql.jdbc.Driver"); //这里注意要将mysql-connector-java-5.1.35-bin.jar拷贝到WEB-INF/lib下面
            Connection con = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "", "");
            Statement stmt = con.createStatement();
            String sql = "select * from purch_sku_master where sku_id = 'sku-000ba89a-4817-4835-ba55-7606ec7b54ed'";

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
![](/images/nodejs-vs-java-performance/1.png)
然后设置一些关于线程组的变量。我们这里使用了30个线程，每个线程发送1000个请求，立即发送。相当于30000个请求一下子过来，看服务器每秒能处理多少个请求。
![](/images/nodejs-vs-java-performance/2.png)
在线程组之下，还需要设置一个HTTP请求sampler，然后在这里面设置需要请求的url
![](/images/nodejs-vs-java-performance/3.png)
最后，添加一个聚合报告，用于查看测试的结果
![](/images/nodejs-vs-java-performance/4.png)


#测试结果
| Label  | # Samples | Average | Median | 90% Line | 95% Line | 99% Line | Min | Max  | Error % | Throughput | KB/sec |
|--------|-----------|---------|--------|----------|----------|----------|-----|------|---------|------------|--------|
| java   | 31887     | 106     | 71     | 210      | 395      | 693      | 1   | 1310 | 0.16%   | 639.8      | 330.0  |
| nodejs | 31862     | 47      | 40     | 96       | 108      | 132      | 0   | 194  | 0.17%   | 642.8      | 118.6  |
| 总体   | 63749     | 76      | 55     | 125      | 210      | 559      | 0   | 1310 | 0.17%   | 1278.7     | 447.9  |