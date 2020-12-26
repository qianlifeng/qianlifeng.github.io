---
title: 使用log4jdbc-log4j2记录完整的SQL执行语句
date: 2015-11-17 16:55:17
---

平常在使用 Hibernate 的 show_sql 开关的时候，打印出来的 sql 不是真实的 sql，参数都是?。在调试性能的时候比较蛋疼。利用[log4jdbc-log4f2](https://code.google.com/p/log4jdbc-log4j2/)可以达到打印完整 SQL 的要求，甚至他还可以打印出执行的结果集

**注意，我这边使用的 logback(slf4j)作为日志记录组件，如果你不是使用的 logback，那么如下方法可能对你不适用，请查看官网文档**

- 加入如下 Maven 包：

  ```xml
  <dependency>
    <groupId>org.bgee.log4jdbc-log4j2</groupId>
    <artifactId>log4jdbc-log4j2-jdbc4</artifactId>
    <version>1.16</version>
  </dependency>

  <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>${slf4j.version}</version>
  </dependency>

  <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>${logback.version}</version>
  </dependency>

  <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>log4j-over-slf4j</artifactId>
      <version>${slf4j.version}</version>
  </dependency>
  <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <version>${slf4j.version}</version>
  </dependency>
  <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jul-to-slf4j</artifactId>
      <version>${slf4j.version}</version>
  </dependency>
  ```

- 更改数据库连接，由原来的

```xml
 jdbc:mysql://localhost:1527
```

改为

```xml
 jdbc:log4jdbc:mysql://localhost:1527
```

- 更改 JDBC Driver，由原来的

```xml
com.mysql.jdbc.Driver
```

改为

```xml
net.sf.log4jdbc.sql.jdbcapi.DriverSpy
```

- 添加 log4jdbc.log4j2.properties 配置文件

```ini
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
```

- 最后在 logback.xml 加入如下 appender

  ```xml
  <logger name="jdbc.sqltiming" level="DEBUG" />
  <logger name="jdbc.resultsettable" level="DEBUG" />
  <logger name="jdbc.resultset" level="WARN" />
  <logger name="jdbc.audit" level="WARN" />
  ```

  log4jdbc-log4j2 支持如下几种 appender

| logger              | description                                                                                                                                                                               |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| jdbc.sqlonly        | Logs only SQL. SQL executed within a prepared statement is automatically shown with it's bind arguments replaced with the data bound at that position, for greatly increased readability. |
| jdbc.sqltiming      | Logs the SQL, post-execution, including timing statistics on how long the SQL took to execute.                                                                                            |
| jdbc.audit          | Logs ALL JDBC calls except for ResultSets. This is a very voluminous output, and is not normally needed unless tracking down a specific JDBC problem.                                     |
| jdbc.resultset      | Even more voluminous, because all calls to ResultSet objects are logged.                                                                                                                  |
| jdbc.resultsettable | Log the jdbc results as a table. Level debug will fill in unread values in the result set.                                                                                                |
| jdbc.connection     | Logs connection open and close events as well as dumping all open connection numbers. This is very useful for hunting down connection leak problems.                                      |
