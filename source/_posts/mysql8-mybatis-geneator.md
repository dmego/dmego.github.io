---
layout: post
title: 'Mybatis 逆向工程中使用 Mysql 8.0 版本驱动遇到的问题'
comments: true
date: 2018-10-10 6:49:00
categories: [Exception]
tags: [Mybatis]
---

<!--more -->

 <center>
<img src="mysql8-mybatis-geneator\mybatis-logo.png" width="250px" />
</center>

## 前言

今天在使用`8.0.12`版的 mysql 驱动时遇到了各种各样的坑，在使用 JDBC 连接上遇到的问题可以参考我的上一篇[博客](https://dmego.me/2018/10/10/java-connect-mysql-error.html)。我在使用 mybatis 逆向工程生成各种 `mapper`，`pojo`，`dao`时，遇到了一个困惑我好几个小时的错误，这个错误是 

```
Result Maps collection already contains value for BaseResultMap
```

产生这个错误可能有各种原因。但是这里我只说我的原因及解决过程。

## 初步探索

我在网上查阅了大量的博客文章，对于产生这类错误的原因最多的是：生成了多次 `mapper`，`dao`以及` pojo` 文件。也就是多次运行了生成这些文件的方法。造成`XXXmapper.xml`中出现了重复的`resultmap`。但是我这里把这些文件删除后，再重新生成还是会报这个错。所以肯定不是多次生成的问题。

于是我打开了出现问题的那个`Mapper.xml`文件，搜索`BaseResultMap`发现其作为`resultMap`的`id`居然出现了三次，还有很多其他的`sql`标签的`id`也有很多重复的。我将这些重复的都删除，再次运行，成功了，没有出现错误。而且利用这些生成的 `mapper`做后面的功能也没有任何问题。这就非常奇怪，为什么会多生成这些代码呢，我继续在网上找相关的文章。

## 深入探索

好不容易找到一篇博客中提到 ：升级到 mysql 8.0 驱动后的使用 mybatis 逆向工程生成的文件或不一样，具体的怎么不一样也没有说。看到这里，我猜会不会是驱动版本造成的，于是我将 `pom.xml`里的 mysql 驱动版本调整到了 5.1.10。删干净文件，再次生成后，发现之前出错的那个 `mapper.xml`里的 以 `BaseResultMap`作为 `id` 的 `resultMap` 只有一个了，其他的`resultMap`中` id` 也是唯一的。为了检验这次生成的到底有没有用。我启动 `Tomact `运行程序。发现正常启动，后续的功能也没有问题。

## 最终解决

但是如果使用 5.0 版本的驱动连接 mysql 8.0 在项目中可能会遇到难以预料的问题，所以我并没有就此将驱动版本改变。我继续在网上通过换各种关键词来搜寻解决方案。几个小时过去了，还是没有任何结果。最后被迫去看了 `[MyBatis Generator]`的官方英文文档。中文文档已经看过了，没有找到相关的内容。在阅读英文文档中，我在[Database Specific Information](http://www.mybatis.org/generator/usage/intro.html)(使用注意事项)下面的[mysql使用注意事项](http://www.mybatis.org/generator/usage/mysql.html#)中似乎找到了相关的内容。其原内容如下：

> If you are using version 8.x of Connector/J you may notice that the generator attempts to generate code for tables in the MySql information schemas (sys, information_schema, performance_schema, etc.) This is probably not what you want! To disable this behavior, add the property "nullCatalogMeansCurrent=true" to your JDBC URL.

> For example:

>```
   <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost/my_schema"
            userId="my_user" password="my_password">
        <property name="nullCatalogMeansCurrent" value=true" />
    </jdbcConnection>
>```

上面的英文文档翻译过来就是:

>如果您使用的是Connector / J的8.x版，您可能会注意到生成器尝试为MySql信息模式（sys，information_schema，performance_schema等）中的表生成代码。这可能不是您想要的！ 要禁用此行为，请将属性“nullCatalogMeansCurrent = true”添加到JDBC 

的确，我发现使用 8.0 版的驱动比使用 5.0 版时不仅`mapper.xml`文件中多生成了好多代码，而且还多生成了一个`xxxWithBLOBs`的`pojo`文件。虽然还是不太理解上面说的问题，但是我还是添加 nullCatalogMeansCurrent 属性。然后重新生成了相关的`mapper`，`pojo`，`dao`。打开之前出现问题的`mapper.xml`文件，和使用 5.0 版的驱动生成的代码一样，以 `BaseResultMap`作为 id 的 resultMap 只有一个了。再次启动 Tomact，成功启动，没有任何问题，测试其他业务功能，也没有任何问题。

## 后记

下面是我找到的相关问题的帖子或博客，虽然没有解决我问题，但是也可能会遇到，这里记录一下

- [mysql8.0逆向工程遇到的坑](https://blog.csdn.net/qq_37718636/article/details/81413800)
- [MyBatis Generator mysql8.0版本的连接的注意点](https://blog.csdn.net/weixin_38107930/article/details/82556164)
- [mybatis(错误一) 项目启动时报“Result Maps collection already contains value forxxx”的解决方案](https://blog.csdn.net/zengdeqing2012/article/details/46340357)
- [mybatis.generator配上最新的mysql 8.0.11的一些坑](https://blog.csdn.net/qq_37350706/article/details/81429154)
- [mysql8.0 mybatis逆向工程 mybatis-generator](https://blog.csdn.net/Mint6/article/details/81869777)