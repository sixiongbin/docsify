# pg数据库学习路线及教程，自我学习提升 🔥

>pg数据库学习提升

# 学习路线

# PostgreSQL新手指引

## 1. 首先应该准备的资料

### 1.1 把官方手册的中文版下载到电脑中，以备需要时查看

PostgreSQL中一些SQL的使用、函数的使用、参数的使用我们不可能都能记下来。当记不住时，就需要到官方手册中查询。所以我们在学习PostgreSQL的第一步就是先把官方手册的CHM版本下载在自己的电脑中，当记不清楚一些SQL或函数的语法时，可以随时查看。

下载链接为： [PostgreSQL官方手册（包括中英文）下载列表](http://www.pgsql.tech/article_101_10000023)

可以下载最近的两个大版本的CHM手册。

### 1.2 买几本书：

- 《PostgreSQL修炼之道 从小工到专家》，一本由浅入深的书，通过阅读此书可以让你快速全面的掌握PostgreSQL的知识，成为一个合格的DBA。如果你完全没有任何数据库的基础，还可以买《PostgreSQL即学即用》、《PostgreSQL9.6从零开始学》。
- 如果没有SQL的基础，可以买本《SQL必知必会》来学习。如果SQL想进阶则可以买《SQL解惑(第2版)》，《SQL沉思录》、《SQL编程风格》、《SQL权威指南》。当然也可以直接在网上学习：http://www.w3school.com.cn/sql/index.asp
- DBA最佳实践的书: 《PostgreSQL实战》、《PostgreSQL 9X之巅》，《数据架构师的PostgreSQL修炼》、《PostgreSQL 9 Administration Cookbook 中文版》，通过学习这几本书和与日常运维工作的结合，可以让你成为一个有丰富实践经验的DBA。另注意《PostgreSQL 9X之巅》实际上是《PostgreSQL 9.0性能调校》这本书的第二版。
- 如果做PostgreSQL存储过程和函数的开发：《PostgreSQL服务器编程》。

## 2. 网上的资料

入门的资料：

- https://www.tutorialspoint.com/postgresql/
- w3school中的SQL入门： http://www.w3school.com.cn/sql/index.asp

### 2.1 社区

社区如下：

- PostgreSQL中文社区：http://www.postgres.cn/
- PostgreSQL中文技术：http://www.pgsql.tech/
- PostgreSQL全球官方网站： http://www.postgresql.org/ https://www.postgresql.org/docs/
- ChinaUnix的PostgreSQL的论坛，上面有一些帖子：http://bbs.chinaunix.net/forum-18-1.html
- 阿里云的云栖社区：https://yq.aliyun.com/ 在此社区中有很多PostgreSQL相关的文章，而且可以搜索，如搜索“postgres”就可以把所有PostgreSQL相关的文章搜索出来。
- 一个国外入门学习网站（全英文） http://www.postgresqltutorial.com/
- PostgreSQL中国社区的微博：https://weibo.com/postgresqlchina
- PostgreSQL中国社区的QQ群：
  PostgreSQL专业群① 5276420
  PostgreSQL专业群② 3336901
  PostgreSQL专业群③ 100910388
  PostgreSQL专业群④ 254622631
  PostgreSQL专业群⑤ 150657323
  PostgreSQL专业群⑥ 461170054
- 全球PG社区峰会，事件：
  https://wiki.postgresql.org/wiki/Events

### 2.2 国内的blog

- 德哥写的文章： https://github.com/digoal/blog 这里面有很多文章，可以慢慢学习。
- osdba的博客： http://blog.osdba.net/
- francs的博客：https://postgres.fun/
- 君羊的博客：https://my.oschina.net/Kenyon

### 2.3 国外BLOG(或FAQ社区)

- http://stackoverflow.com/questions/tagged/postgresql
- 社区大管家的blog: [http://momjian.us](http://momjian.us/)
- [http://www.pgexperts.com](http://www.pgexperts.com/)
- http://blog.2ndquadrant.com/en
- https://github.com/dhamaniasad/awesome-postgres

### 2.4 学习视频

- PostgreSQL 数据库优化 培训视频 3天 http://pan.baidu.com/s/1pKVCgHX
- 阿里云：PostgreSQL数据库从入门到精通 http://click.aliyun.com/m/39417/

### 2.5 可下载的资料

- 历次PostgreSQL中国用户大会的资料
  [http://www.postgres.cn/news/typelist/1/%E4%BC%9A%E8%AE%AE%E8%B5%84%E6%96%99](http://www.postgres.cn/news/typelist/1/会议资料)

## 3. 内核开发

### 3.1 内核入门者必读

想成为PostgreSQL的内核开发者，请先阅读：

- https://wiki.postgresql.org/wiki/Developer_FAQ/zh

这篇文章详细讲解了如何提交代码到PostgreSQL内核中。

从下面这个地方可以看到PostgreSQL的Todo列表：

- https://wiki.postgresql.org/wiki/Todo
  如果你想成为PostgreSQL的内核开发者，可以从这个列表中挑选相应的功能做开发，当然在开发之前，需要发邮件与PostgreSQL内核社区的人做沟通。

在线查看代码:

- http://doxygen.postgresql.org/

查看一些提交列表可以见：

- https://commitfest.postgresql.org/

git仓库：

- [http://git.postgresql.org](http://git.postgresql.org/)

### 3.2 学习PostgreSQL内核方面的书籍

- 《PostgreSQL 数据库内核分析》作者: 彭智勇,彭煜玮
- 《PostgreSQL查询引擎源码技术探析》
- 《PostgreSQL技术内幕:查询优化深度探索》
- 《数据库查询优化器的艺术》

### 3.2 内核方面的学习资料

- 不错的讲解PostgreSQL技术内幕的网站：The Internals of PostgreSQL: http://www.interdb.jp/pg/
- 《Introduction to Hacking PostgreSQL》
- 《Introduction PostgreSQL WAL》
- http://www.postgresql.org/developer/backend/
- http://wiki.postgresql.org/wiki/Backend_flowchart
- PostgreSQL源代码的结构：https://wiki.postgresql.org/wiki/Pgsrcstructure
- PPT：《从PostgreSQL实现Flashback功能模块-谈小白如何开始内核定制开发》：

## 4. PostgreSQL的相关配套软件和插件的下载

### 4.1 驱动

- PostgreSQL JDBC 驱动
  [http://jdbc.postgresql.org](http://jdbc.postgresql.org/)
  http://jdbc.postgresql.org/development/privateapi/
- PostgreSQL ODBC 驱动
  http://www.postgresql.org/ftp/odbc/versions/src/

### 4.2 插件

- 有很多插件：pgfoundry： http://pgfoundry.org/,
- 有很多插件：pgxn: https://pgxn.org/
- pg_pathman: https://github.com/postgrespro/pg_pathman
- PostGIS: 最著名的开源GIS系统，为PostgreSQL数据库中的一个插件
  http://www.postgis.org/
  https://learn.boundlessgeo.com/series/postgis
  http://www.opengeospatial.org/

### 4.3 高可用或连接池软件

- 连接池软件pgbouncer : http://pgbouncer.github.io/
- 高可用及连接池pgpool-II:
  http://www.pgpool.net/mediawiki/index.php/Main_Page
  http://www.pgpool.net/docs/latest/en/html/index.html

### 4.4 基于PostgreSQL源码改造的数据库

- 基于PostgreSQL的MPP数据库Greenplum: https://github.com/greenplum-db/gpdb
- 把greenplum的SQL执行移到hadoop上：http://hawq.apache.org/
- AntDB：基于PostgreSQL的分布式数据库，适合于OLTP场景，是基于原先Postgres-XC的增强
  https://github.com/ADBSQL/AntDB
- Postgres-XL: 基于PostgreSQL的分布式数据库，基于原先Postgres-XC的增强，想同时支持OLTP和OLAP:
  https://www.postgres-xl.org/
- Postgres-XC: 基于PostgreSQL的分布式数据库，适合于OLTP场景，目前已不太活跃
  http://postgres-xc.sourceforge.net/docs/1_2_1/
  https://github.com/postgres-x2/postgres-x2

### 4.5 小工具

自动生成通用优化参数，PostgreSQL中还有很多其他可以调整的参数这个小工具没有调整，但这个小工具可以当成入门：
http://pgtune.leopard.in.ua/

在线SQL格式化工具：

- http://sqlformat.darold.net/

可以练习SQL的网站（可能要翻墙）：
https://pgexercises.com/gettingstarted.html

