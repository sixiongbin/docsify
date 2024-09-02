## PostgreSQL 应用开发解决方案最佳实践系列课程 - 1. 中文分词与模糊查询  
  
### 作者  
digoal  
  
### 日期  
2021-05-02   
  
### 标签  
PostgreSQL , 解决方案 , 最佳实践 , 课程   
  
----  
  
## 背景  
PostgreSQL 应用开发解决方案最佳实践系列课程 - 1. 中文分词与模糊查询  
   
课程介绍  
PostgreSQL是一款非常流行的关系数据库，截至2020年, 荣获3届DBEngine年度最佳数据库称号. PostgreSQL的特点包括覆盖最新的SQL标准, 支持NoSQL特性, 支持并行计算, 支持多达10种不同的索引结构, 可以兼容非常丰富的数据模型, 同时PostgreSQL支持扩展插件, 可以与应用更好的结合提高业务效率，例如PostGIS插件支持GIS的数据类型和计算函数, 操作符, 索引等. hll插件, 支持近似分析. 开源的插件有数千种之多. 所以PostgreSQL 也被认为是很难达到精通的一款数据库.   
随着云计算的发展，管好一款数据库已经成为过去式，用好一款数据库才是未来, 而如果你认为会简单的SQL操作就掌握了数据库的使用就大错特错了, 本次课程将从场景出发，介绍数据库与应用如何结合才能更加高效解决业务问题。  
人类统治地球的秘密是高效的掌握了先进工具, 每一次革命实际上都是因为人类掌握了更先进的工具. PostgreSQL就是当下最先进的数据库工具。  

课程收益  
学习PostgreSQL的中文分词、模糊搜索功能. 通过短视频业务场景, 学习掌握PostgreSQL的HLL近似搜索功能. 通过人脸识别场景, 学习PG的向量索引能力. 通过出行系统、配送系统、轨迹系统, 学习PG的时空调度功能.
通过标签系统, 学习PG的位图类型、多值类型与倒排索引的应用. 通过裂变时传播业务系统, 掌握PG的递归应用. 通过海量数据存储系统, 掌握PG的冷热分离存储技术.   
掌握这些技巧, 你的应用可以在节省成本的情况下, 得到数倍甚至数万倍的性能提升.   
  
## 课程对象  
数据库架构师, 应用开发者, DBA.   
  
[视频回放](xx)  
  
## 1、应用场景  
  
1、电商  
商品名搜索  
店铺名搜索  
商品描述搜索  
  
2、论坛、社交APP  
标题搜索  
内容搜索  
昵称搜索  
  
## 2、业界解决方案  
应用连数据库  
数据库增量订阅到MQ  
MQ同步到搜索引擎ES  
应用连ES搜索  
涉及异构关联查询, 应用既要访问数据库又要访问ES, 并在应用中后合并结果  
  
## 3、业界解决方案的挑战或痛点  
开发成本增加  
维护成本增加  
软硬件成本增加  
同步延时增加, 无法实时写入实时查询  
同步错误, 导致查询结果不一致, 需定期刷全量或定期检查和修复差异  
跨库查询代价高, 例如业务查询既有文本搜索的条件, 也有其他字段的筛选条件, 需要同时访问ES和数据库.   
  
## 4、PG的解决方案  
分词  
全文检索和模糊搜索  
倒排索引    
  
## 5、PG的解决方案原理  
分词原理  
倒排索引  
  
## 6、PG的解决方案 VS 业界解决方案   
  
## 7、DEMO  
  
准备工作  
ECS , Linux + PostgreSQL 客户端软件  
阿里云 RDS PostgreSQL 13  
  
### 7.1、中文分词搜索  
  
创建插件  
  
建表和索引  
  
生成数据  
  
查询  
  
查看执行计划  
  
rum 支持索引实现 ranking 排序  
  
tsquery 词组距离过滤  
  
rum 支持多值+其他字段索引精准过滤  
  
查看pending list, 索引build实时性, autovacuum后台进程负责异步合并pending list  
  
自定义词组测试  
  
### 7.2、模糊搜索  
  
创建插件  
  
建表和索引  
  
生成数据  
  
查询  
  
查看执行计划  
  
查看pending list, 索引build实时性, autovacuum后台进程负责异步合并pending list  
  
关于wchar, 双、单字      
[《PostgreSQL 模糊查询增强插件pgroonga , pgbigm (含单字、双字、多字、多字节字符) - 支持JSON模糊查询等》](../202003/20200330_01.md)    
[《PostgreSQL 模糊查询最佳实践 - (含单字、双字、多字模糊查询方法)》](../201704/20170426_01.md)    
  
### 7.3、组合搜索  
  
多字段(含模糊或中文分词)的组合搜索  
  
### 7.4、性能  
  
### 7.5、扩展性  
垂直扩展  
水平扩展   
只读实例  
  
  
## 8、知识点回顾  
  
  
多值类型  
文本向量  
中文分词  
自定义分词  
ranking  
zhparser  
倒排索引  
rum索引  
pg_trgm与模糊搜索  
  
## 9、参考文档  
[《PostgreSQL 15大惊奇应用场景实践 - 直播预告》](../202009/20200903_02.md)    
  
https://help.aliyun.com/document_detail/142340.html  
  
https://help.aliyun.com/document_detail/186778.html  
  
  
  
#### [PostgreSQL 许愿链接](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216")
您的愿望将传达给PG kernel hacker、数据库厂商等, 帮助提高数据库产品质量和功能, 说不定下一个PG版本就有您提出的功能点. 针对非常好的提议，奖励限量版PG文化衫、纪念品、贴纸、PG热门书籍等，奖品丰富，快来许愿。[开不开森](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216").  
  
  
#### [9.9元购买3个月阿里云RDS PostgreSQL实例](https://www.aliyun.com/database/postgresqlactivity "57258f76c37864c6e6d23383d05714ea")
  
  
#### [PostgreSQL 解决方案集合](https://yq.aliyun.com/topic/118 "40cff096e9ed7122c512b35d8561d9c8")
  
  
#### [德哥 / digoal's github - 公益是一辈子的事.](https://github.com/digoal/blog/blob/master/README.md "22709685feb7cab07d30f30387f0a9ae")
  
  
![digoal's wechat](../pic/digoal_weixin.jpg "f7ad92eeba24523fd47a6e1a0e691b59")
  
