---
title: MySQL数据库如何支持emoji表情
abbrlink: 44e620e3
date: 2019-01-01 11:14:55
tags:
---

最近做的一个微信公众号的项目，有一个需求是：通过openID获取到的用户基础信息并持久化到数据库中，当时也没有多想，哐哐哐就是一通写。后来测试反馈说有些用户信息无法存储，我查了一下日志，发现类似异常：`java.sql.SQLException: Incorrect string value: '\xF0\x9F\x92\x94' for column 'name' at row 1 `，这才发现自己忽略了一种情况，有些用户的昵称带有emoji表情，MySQL字符集使用的uft8，只能存储3个字节的数据，但是emoji表情是4个字节，这样数据持久化的时候就必然异常。

<!-- more --> 

解决方案也比较简单，把字符集utf8更改为utf8mb4。下面是具体的步骤：

1. 修改字段的字符集：`ALTER TABLE userinfo MODIFY COLUMN nickname varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`

2. 修改表的字符集：`ALTER TABLE userinfo charset=utf8mb4;`

3. 修改库的字符集：`SET NAMES utf8mb4;`

4. 项目中的MySQL配置需要修改一下，我这里用的是Spring boot，所以在`application.yml`文件中添加:

   ```yaml
   spring:
     datasource:
       type: com.zaxxer.hikari.HikariDataSource
       hikari:
         connection-init-sql: SET NAMES utf8mb4
   ```


