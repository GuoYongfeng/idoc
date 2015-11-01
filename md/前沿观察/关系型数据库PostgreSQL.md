# PostgreSQL基础入门知识

> PostgreSQL是以加州大学伯克利分校计算机系开发的 POSTGRES 版本 4.2 为基础的对象关系型数据库管理系统（ORDBMS）。 POSTGRES 领先的许多概念只是在非常迟的时候才出现在商业数据库中。

PostgreSQL 是最初伯克利的代码的一个开放源码的继承人。 它支持大部分 SQL 标准并且提供了许多其他现代特性：

- 复杂查询
- 外键
- 触发器
- 视图
- 事务完整性
- 多版本并发控制

同样，PostgreSQL 可以用许多方法扩展，比如， 通过增加新的：

- 数据类型
- 函数
- 操作符
- 聚集函数
- 索引方法
- 过程语言

并且，因为许可证的灵活，任何人都可以以任何目的免费使用，修改，和分发 PostgreSQL， 不管是私用，商用，还是学术研究使用。

## 安装

- 在Liunux服务器上安装

安装PostgreSQL客户端:
```
sudo apt-get install postgresql-client
```
安装PostgreSQL服务器:
```
sudo apt-get install postgresql
```

- 在Windows上安装

去[postgresql](http://www.postgresql.org/download/windows/)下载安装包
再安装一个管理工具[pgadmin](www.pgadmin.org)
