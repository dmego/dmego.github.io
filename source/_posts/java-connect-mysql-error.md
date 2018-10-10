---
layout: post
title: 'JDBC连接Mysql数据库出现的问题汇总'
comments: true
date: 2018-10-10 4:10:20
categories: [Exception]
tags: [Mysql,JDBC]
---

<!--more -->

<center>
<img src="java-connect-mysql-error\MySQL.png" width="250px" />
</center>

## 前言

最近安装了一个 `mysql 8.0` 版本的数据库，在程序中连接的时候可谓是状况不断。之前也会遇到一些问题，这里就对使用 JDBC 连接mysql 会出现的问题做一个汇总。

在此之前说明一下环境：

- 开发工具：IDEA

- mysql版本： 8.0.12 for Win64 on x86_64 (MySQL Community Server - GPL) 
- mysql驱动包：8.0.12

## 驱动包URL 的改变

#### 异常信息

> Loading class `com.mysql.jdbc.Driver`. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver`. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.

#### 原因

通过异常我们可以发现，新的驱动url是`com.mysql.cj.jdbc.Driver`,经过在网上查阅资料发现，从 `mysql6`开始，驱动包开始使用新的驱动 url。如果使用旧的 5.0 版本的驱动包，则不用驱动URL，但是如果使用旧的驱动可能会出现一些意想不到的问题。所以还是建议将驱动包升级，然后改变 驱动 URL 的值。

#### 解决方法

将驱动 URL 由`com.mysql.jdbc.Driver` 换成 `com.mysql.cj.jdbc.Driver`

## SSL 警告

#### 警告信息

> Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.

#### 原因

对警告信息翻译如下。

> 不建议在没有服务器身份验证的情况下建立SSL连接。根据MySQL 5.5.45+，如果未设置显式选项，则默认情况下必须建立5.6.26+和5.7.6+要求的SSL连接。对于不使用SSL的现有应用程序，ValuyServer证书属性设置为“false”。您需要通过设置useSSL=false来显式禁用SSL，或者设置useSSL=true并提供用于服务器证书验证的信任库`。

#### 解决方法

一般在开发中基本不需要使用 SSL 连接，在连接字符串后添加`useSSL=false`参数就行。但是如果真的有 SSL 连接的需要，则在驱动 URL 后添加`useSSL=true`参数。

```
jdbc:mysql://localhost:3306/dbname?characterEncoding=UTF-8&useSSL=false
```

## 时区问题

#### 异常信息

> java.sql.SQLException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.

#### 原因

同样也是由于版本升级后，新的版本数据库和系统之间有了时区差异，需要指定时区` serverTimezone`

#### 解决方法

- 连接字符串后添加参数`&serverTimezone=GMT%2B8`，最终连接字符串如下：

  ```
  jdbc:mysql://localhost:3306/dbname?characterEncoding=UTF-8&useSSL=false&serverTimezone=GMT%2B8
  ```

- 修改数据库时间。先通过命令行连上数据库，依次输入命令及其输出如下

  ```
  mysql> show variables like "%time_zone";
  +------------------+--------+
  | Variable_name    | Value  |
  +------------------+--------+
  | system_time_zone |        |
  | time_zone        | SYSTEM |
  +------------------+--------+
  2 rows in set, 1 warning (0.04 sec)
  
  mysql> set global time_zone="+8:00";
  Query OK, 0 rows affected (0.01 sec)
  ```



## XML 配置文件中 & 的转义

#### 异常信息

> org.mybatis.generator.exception.XMLParserException: XML Parser Error on line 16: 对实体 "useSSL" 的引用必须以 ';' 分隔符结尾。

#### 原因

这是我在使用` mybatis generator`时出现的错误。当时我想在连接字符串后加上`useSSL`参数，但是由于在 `XML` 文件中，`& `是被禁止的，所以需要使用 `& `的时要用它的转义`&amp;`来代替。

#### 解决方法

将连接字符串中的 `& `符号改成`&amp;`

## 详细连接字符串参考

```
jdbc:mysql://127.0.0.1:3306/dbname?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&useSSL=false&serverTimezone=GMT%2B8&verifyServerCertificate=false&autoReconnct=true&autoReconnectForPools=true&allowMultiQueries=true 
```

当然如果是使用 XML 作为配置文件，需要将 连接字符串中的  `& `符号改成`&amp;`





