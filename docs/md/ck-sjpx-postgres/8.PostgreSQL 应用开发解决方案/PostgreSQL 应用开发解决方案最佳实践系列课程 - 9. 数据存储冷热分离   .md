## PostgreSQL 应用开发解决方案最佳实践系列课程 - 9. 数据存储冷热分离             
                    
### 作者                    
digoal                    
                    
### 日期                    
2021-05-10                     
                    
### 标签                    
PostgreSQL , 解决方案 , 最佳实践 , 课程                     
                    
----                    
                    
## 背景                    
        
PostgreSQL 应用开发解决方案最佳实践系列课程 - 9. 数据存储冷热分离   
                    
[视频回放](xx)                  
                  
## 课程对象                    
数据库架构师, 应用开发者, DBA.                     
                    
## 1、应用场景                    
       
通用场景  
用户feed log表历史数据转储  
应用日志表历史数据转储  
云原生, OLTP与OLAP实时联动分析  
  
                    
## 2、业界解决方案      
  
由于行业合规需求, 历史数据不能删, 客户被迫花大价钱购买专业存储设备, 采用大存储, 一个数据库上百TB.  
  
或者是把业务库和历史库分开. 通过ETL程序把历史数据同步到专门的历史库.    
  
## 3、业界解决方案的挑战或痛点        
  
历史数据存储在OLTP数据库中耗费大量数据库存储空间, 成本高     
  
数据库存在数据库中, 备份慢、恢复慢    
  
数据在oltp+olap中需要同时存若干份  
  
对历史表进行查询或分析的结果, 要回流到oltp库时, 需要传输一次结果数据      
  
如果既有历史表访问需求, 又有其他业务表需要关联查询时, 需要在业务层合并数据, 性能差.   
                    
## 4、PG的解决方案          
      
阿里云RDS PostgreSQL 支持OSS外部表存储. 用户可以将历史数据转储到OSS, 通过SQL依旧可以直接查询OSS的内容.  
数据在OSS后, 可以利用OSS的函数计算功能, 实现schema less计算.  
计算可以通过plpython直接调用OSS函数完成, 同时结果可以直接返回给RDS PG.  
历史数据存储在OSS, 数据库瘦身后备份恢复效率得以提高.   
数据存储在OSS, 与云上其他产品数据打通, 因为大多数云产品都可以读写OSS, 使得一份数据可以发挥最大的计算价值, 而不需要为每一个要用这些数据的产品同步一份数据到产品中. 成本大幅度降低, 使用效率大幅度提升.     
    
    
## 5、PG的解决方案原理         
    
  
                    
## 6、PG的解决方案 VS 业界解决方案                    
           
成本更低  
效率更高  
关系数据库使用OSS外部表功能, 无缝访问历史数据  
云原生, 打通云上多款产品, 一份数据发挥最大价值  
              
## 7、DEMO                    
                    
准备工作                    
ECS , Linux + PostgreSQL 客户端软件                    
阿里云 RDS PostgreSQL 13             
              
                    
### 7.1、 数据存储冷热分离  
  
建表  
写入测试数据  
将历史数据写入OSS外部表  
外部表使用时间分区表管理  
访问OSS外部表  
使用OSS函数计算功能  
使用plpython直接调用oss函数计算  
  
## 8、知识点回顾     
    
oss_fdw  
dblink  
异步并行任务  
plpython  
oss 函数计算  
                    
## 9、参考文档                    
[《PostgreSQL 15大惊奇应用场景实践 - 直播预告》](../202009/20200903_02.md)       
    
https://help.aliyun.com/document_detail/44461.htm   
    
[《PostgreSQL+MySQL 联合解决方案 - 第6课视频 - PG外部表妙用 - mysql_fdw, oss_fdw（直接读写mysql、冷热分离、归档存储）》](../202001/20200108_01.md)    
  
[《阿里云RDS PostgreSQL OSS 外部表实践 - (dblink异步调用封装并行) 从OSS并行导入数据》](../201804/20180427_01.md)    
  
[《阿里云RDS PostgreSQL OSS 外部表实践 - (dblink异步调用封装并行) 数据并行导出到OSS》](../201709/20170906_01.md)    
  
[《ApsaraDB的左右互搏(PgSQL+HybridDB+OSS) - 解决OLTP+OLAP混合需求》](../201701/20170101_02.md)    
  
[《强制数据分布与导出prefix - 阿里云pg, hdb pg oss快速数据规整外部表导出实践案例》](../201801/20180109_01.md)    
  
[《PostgreSQL 分析型SQL优化case一例 - 分析业务逻辑,分区,dblink异步并行》](../202002/20200215_01.md)    
  
[《PostgreSQL 11 相似图像搜索插件 imgsmlr 性能测试与优化 2 - 单机分区表 (dblink 异步调用并行) (4亿图像)》](../201809/20180904_03.md)    
  
[《PostgreSQL dblink异步调用实践,跑并行多任务 - 例如开N个并行后台任务创建索引, 开N个后台任务跑若干SQL》](../201809/20180903_01.md)    
  
[《在PostgreSQL中跑后台长任务的方法 - 使用dblink异步接口》](../201806/20180621_03.md)    
  
[《PostgreSQL 批量导入性能 (采用dblink 异步调用)》](../201804/20180427_03.md)    
  
[《阿里云RDS PostgreSQL OSS 外部表实践 - (dblink异步调用封装并行) 从OSS并行导入数据》](../201804/20180427_01.md)    
  
[《PostgreSQL 变态并行拉取单表的方法 - 按块并行(按行号(ctid)并行) + dblink 异步调用》](../201804/20180410_03.md)    
  
[《PostgreSQL 相似搜索分布式架构设计与实践 - dblink异步调用与多机并行(远程 游标+记录 UDF实例)》](../201802/20180205_03.md)    
  
[《PostgreSQL dblink异步调用实现 并行hash分片JOIN - 含数据交、并、差 提速案例 - 含dblink VS pg 11 parallel hash join VS pg 11 智能分区JOIN》](../201802/20180201_02.md)    
  
[《惊天性能！单RDS PostgreSQL实例 支撑 2000亿 - 实时标签透视案例 (含dblink异步并行调用)》](../201712/20171223_01.md)    
  
[《阿里云RDS PostgreSQL OSS 外部表实践 - (dblink异步调用封装并行) 数据并行导出到OSS》](../201709/20170906_01.md)    
  
  
#### [PostgreSQL 许愿链接](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216")
您的愿望将传达给PG kernel hacker、数据库厂商等, 帮助提高数据库产品质量和功能, 说不定下一个PG版本就有您提出的功能点. 针对非常好的提议，奖励限量版PG文化衫、纪念品、贴纸、PG热门书籍等，奖品丰富，快来许愿。[开不开森](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216").  
  
  
#### [9.9元购买3个月阿里云RDS PostgreSQL实例](https://www.aliyun.com/database/postgresqlactivity "57258f76c37864c6e6d23383d05714ea")
  
  
#### [PostgreSQL 解决方案集合](https://yq.aliyun.com/topic/118 "40cff096e9ed7122c512b35d8561d9c8")
  
  
#### [德哥 / digoal's github - 公益是一辈子的事.](https://github.com/digoal/blog/blob/master/README.md "22709685feb7cab07d30f30387f0a9ae")
  
  
![digoal's wechat](../pic/digoal_weixin.jpg "f7ad92eeba24523fd47a6e1a0e691b59")
  
