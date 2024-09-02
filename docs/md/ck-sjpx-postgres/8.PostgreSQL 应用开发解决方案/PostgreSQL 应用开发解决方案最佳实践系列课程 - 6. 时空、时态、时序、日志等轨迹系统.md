## PostgreSQL 应用开发解决方案最佳实践系列课程 - 6. 时空、时态、时序、日志等轨迹系统           
              
### 作者              
digoal              
              
### 日期              
2021-05-09               
              
### 标签              
PostgreSQL , 解决方案 , 最佳实践 , 课程               
              
----              
              
## 背景              
本文信息量非常大.   
  
PostgreSQL 应用开发解决方案最佳实践系列课程 - 6. 时空、时态、时序、日志等轨迹系统       
              
[视频回放](xx)            
            
## 课程对象              
数据库架构师, 应用开发者, DBA.               
              
## 1、应用场景              
              
- 网约车:   
    - 实时位置, 时空轨迹回放、轨迹偏离告警、地理围栏、异步消息和告警    
- 共享单车:   
    - 实时位置, 无损和有损压缩, 冷热分离存储, 时空轨迹回放   
- 车联网:   
    - 实时位置, 无损和有损压缩, 冷热分离存储, 时空轨迹回放   
- 应用 feed log:   
    - 时序存储、无损和有损压缩、区间或点查询、区间聚合分析和存储、大范围分析   
- 物联网:   
    - 时序存储、无损和有损压缩、区间或点查询、区间聚合分析和存储、大范围分析、first value、last value、柱状图分析、触发异步消息和告警通知、流计算  
- 证券:   
    - 时序存储、无损和有损压缩、区间或点查询、区间聚合分析和存储、大范围分析、first value、last value、柱状图分析、触发异步消息和告警通知、流计算  
- 电网:   
    - 时序存储、无损和有损压缩、区间或点查询、区间聚合分析和存储、大范围分析、first value、last value、柱状图分析、触发异步消息和告警通知、流计算  
- 公安刑侦:   
    - 时空轨迹时态分析、触发异步消息和告警通知、流计算     
              
## 2、业界解决方案              
      
KV存储  
         
## 3、业界解决方案的挑战或痛点      
  
KV存储只做到:  
写入快  
易扩展  
压缩比高  
  
但是:  
  
综合能力极其简陋, 研发成本极高. 与关系数据库相差十万八千里, 要啥没啥, 开发极其痛苦, KV存储只负责高速入库, 其他什么几乎都要业务实现, 开发代价大, 周期长, 研发成本极高.   
查询性能差, 这里说的不是点查, 是区间或JOIN分析类的查询弱.   
不支持时空、空间、轨迹等多模类型, 不支持轨迹类型和轨迹运算. 无法满足GIS类时序业务需求, 如时空轨迹存储、时空分析、时态分析等.   
不支持时空索引  
不支持时序索引  
不支持流计算  
不支持异步消息触发  
不支持冷热分离存储  
不支持行、列、冷、热混合存储   
              
## 4、PG的解决方案              
    
PostgreSQL 针对时空、时态、时序、日志等轨迹系统的综合能力图谱:     
  
- ganos:   
    - 阿里云自研PostgreSQL时空多模引擎, PostGIS 增强版.  
    - https://help.aliyun.com/document_detail/107501.html  
- postgis:   
    - PostgreSQL开源时空多模引擎. 支持2D3D模型, 栅格, 拓扑, 时空轨迹等模型.      
    - http://postgis.net/docs/reference.html  
- timescaledb:   
    - PostgreSQL开源时序数据库引擎. 自动分片, 内置压缩, 支持物化(增量“流式”计算)  
    - [《PostgreSQL 时序数据库设计最佳实践 - 关联 citus,columnar,partition,timescaledb,压缩,高速写,parallel append 多分区并行查询,分区》](../202104/20210428_03.md)    
    - [《PostgreSQL 时序数据库插件 timescaleDB 部署实践(含例子 纽约TAXI数据透视分析) - PostGIS + timescaleDB => PG时空数据库》](../201801/20180129_01.md)    
    - [《PostgreSQL 按需切片的实现(TimescaleDB插件自动切片功能的plpgsql schemaless实现)》](../201711/20171102_02.md)    
    - [《时序数据库有哪些特点? TimescaleDB时序数据库介绍》](../201704/20170409_05.md)    
- citus:   
    - PostgreSQL 开源物联网、时序插件. 支持列存储, sharding扩展  
    - [《使用Postgres，MobilityDB和Citus大规模(百亿级)实时分析GPS轨迹》](../202011/20201117_01.md)    
    - [《怎么看待PostgreSQL sharding插件: citus - 对比PG 社区基于 postgres_fdw 的sharding》](../202103/20210325_02.md)    
    - [《PostgreSQL citus 发布 10版本 - 支持columner 列存储, 开放rebalance源码, 支持reference,分布式,本地表JOIN 等》](../202103/20210307_03.md)    
    - [《PostgreSQL sharding extension citus 优化器 Query Processing 之 - Distributed Query Planner、Executor (Real-time Executor, Router Executor, Task Tracker Executor)》](../201903/20190316_02.md)    
    - [《PostgreSQL sharding extensino citus 优化器 Query Processing 之 - Subquery/CTE Push-Pull Execution》](../201903/20190316_01.md)    
    - [《PostgreSQL sharding : citus 系列7 - topn 加速(```count(*) group by order by count(*) desc limit x```) (use 估值插件 topn)》](../201809/20180914_01.md)    
    - [《PostgreSQL sharding : citus 系列6 - count(distinct xx) 加速 (use 估值插件 hll|hyperloglog)》](../201809/20180913_04.md)    
    - [《PostgreSQL sharding : citus 系列5 - worker节点网络优化》](../201809/20180905_02.md)    
    - [《PostgreSQL sharding : citus 系列4 - DDL 操作规范 (新增DB，TABLE，SCHEMA，UDF，OP，用户等)》](../201809/20180905_01.md)    
    - [《PostgreSQL sharding : citus 系列3 - 窗口函数调用限制 与 破解之法(套用gpdb执行树,分步执行)》](../201809/20180902_01.md)    
    - [《PostgreSQL sharding : citus 系列2 - TPC-H》](../201808/20180829_01.md)    
    - [《PostgreSQL citus, Greenplum  分布式执行计划 DEBUG》](../201808/20180828_01.md)    
    - [《PostgreSQL sharding : citus 系列1 - 多机部署（含OLTP(TPC-B)测试）- 含Citus MX模式》](../201808/20180824_02.md)    
- 并行计算:   
    - PG内置并行计算, 解决时序数据实时分析性能问题   
    - [《PostgreSQL 并行计算解说 汇总》](../201903/20190318_05.md)    
- 函数丰富(例如旋转门):   
    - 内置plpgsql, 可扩展支持plpython , pljava等编成语言. move code比move data的效率高太多了.   
    - [《SQL流式案例 - 旋转门压缩(前后计算相关滑窗处理例子)》](../201801/20180107_01.md)    
    - [《旋转门数据压缩算法在PostgreSQL中的实现 - 流式压缩在物联网、监控、传感器等场景的应用》](../201608/20160813_01.md)    
- 压缩存储:   
    - PG支持TOAST压缩技术, 支持扩展压缩算法.   
    - [《PostgreSQL 14 preview - TOAST 支持 lz4 压缩算法 - --with-lz4 , 新增GUC default_toast_compression》](../202103/20210320_01.md)    
    - [《PostgreSQL appendonly 压缩 存储引擎 - pg_cryogen》](../202003/20200324_10.md)    
- 冷热分离存储:   
    - 阿里云RDS PG支持OSS_FDW OSS冷存储功能, 允许用户将冷数据存储在OSS, 通过PG SQL可直接读写冷存储中的表数据.   
    - [《PostgreSQL+MySQL 联合解决方案 - 第6课视频 - PG外部表妙用 - mysql_fdw, oss_fdw（直接读写mysql、冷热分离、归档存储）》](../202001/20200108_01.md)    
    - [《阿里云RDS PostgreSQL OSS 外部表实践 - (dblink异步调用封装并行) 从OSS并行导入数据》](../201804/20180427_01.md)    
    - [《阿里云RDS PostgreSQL OSS 外部表实践 - (dblink异步调用封装并行) 数据并行导出到OSS》](../201709/20170906_01.md)    
- OSS冷数据函数计算, 在PG内可以直接调用plpython, pljava等实现OSS 函数计算的调用:   
    - 冷数据存储在阿里云OSS后, 可以利用OSS的函数计算功能, 进行分布式计算. 在PG实例中使用plpython直接调用函数计算, 结果直接返回到PG实例. 打破了数据存储孤岛, 实现PG(时序OLTP数据库)+OSS(OLAP函数计算)的无缝融合, 例如分析结果直接进入PG数据库, 不需要再走一次数据同步. 这是云原生能力的增强.    
    - [《使用PostgreSQL plpythonu实现一个推荐引擎 - Building a recommendation engine inside Postgres with Python and Pandas》](../202103/20210311_02.md)    
    - [《AWS redshift->hdb pg(Greenplum)， plpython, pljava UDF 以及upload library》](../201810/20181017_04.md)    
    - [《[未完待续] PostgreSQL 流式fft傅里叶变换 (plpython + numpy + 数据库流式计算)》](../201803/20180307_01.md)    
    - [《在PostgreSQL中使用 plpythonu 调用系统命令》](../201710/20171023_01.md)    
- 堆行存储:   
    - 堆存储支持高速写入, 单实例最高可支持每秒百万行以上的写入速度.   
    - [《HTAP数据库 PostgreSQL 场景与性能测试之 24 - (OLTP) 物联网 - 时序数据并发写入(含时序索引BRIN)》](../201711/20171107_25.md)    
- 列存储:   
    - 列存储支持高效率压缩, 支持更好的计算性能, 提高大范围数据分析性能    
    - [《PostgreSQL 时序数据库设计最佳实践 - 关联 citus,columnar,partition,timescaledb,压缩,高速写,parallel append 多分区并行查询,分区》](../202104/20210428_03.md)    
- 行、列、冷、热、混合分区:   
    - PG支持分区表功能, 支持混合分区, 例如历史数据分区采用列存储或冷存储, 最近的数据分区使用堆行存储和热存储.   
    - [《PostgreSQL 时序数据库设计最佳实践 - 关联 citus,columnar,partition,timescaledb,压缩,高速写,parallel append 多分区并行查询,分区》](../202104/20210428_03.md)    
- JSON和SQL/JSON:   
    - 内置JSON功能, 支持jsonpath语法, 对于开发者非常友好. 使用JSON在物联网领域大幅度降低开发代价.    
    - [《不懂jsonpath的话等于JSON没入门》](../202105/20210507_04.md)    
    - [《PostgreSQL 史上最强JSON功能 - PG 12 jsonpath 完全超越oracle, mysql, sql server的sql json标准覆盖率》](../202010/20201013_01.md)    
- batch insert:   
    - 支持批量写入接口, 单实例最高可支持每秒百万行以上的写入速度.   
    - [《PostgreSQL jdbc batch insert》](../201703/20170329_03.md)    
- 异步消息:   
    - 内置listen/notify的异步消息接口, 在物联网、证券、电网、公安行政场景中, 可以大幅提升流式数据实时预警功能. 避免采用任务调度形式的间歇式查询导致的大量无用功和时延问题(因为大多数情况下预警数据只占数据总量的0.1%, 任务调度形式的预警解决方案, 大量SQL查询是无用功).    
    - [《PostgreSQL 流式处理应用实践 - 二手商品实时归类(异步消息notify/listen、阅后即焚)》](../201807/20180713_03.md)    
    - [《PostgreSQL 异步消息实践 - Feed系统实时监测与响应(如 电商主动服务) - 分钟级到毫秒级的实现》](../201711/20171111_01.md)    
    - [《从电波表到数据库小程序之 - 数据库异步广播(notify/listen)》](../201701/20170116_01.md)    
    - [《从微信小程序 到 数据库"小程序" , 鬼知道我经历了什么》](../201701/20170113_03.md)    
- 时序索引:   
    - 支持时序索引(brin, block range index), 大幅度降低索引的空间占用, 大幅度降低索引带来的写入性能影响. 10亿条记录的brin时序索引仅占用几MB空间, 而使用传统Btree索引需要至少几十GB空间.   
    - [《PostgreSQL 聚集存储 与 BRIN索引 - 高并发行为、轨迹类大吞吐数据查询场景解说》](../201702/20170219_01.md)    
    - [《HTAP数据库 PostgreSQL 场景与性能测试之 24 - (OLTP) 物联网 - 时序数据并发写入(含时序索引BRIN)》](../201711/20171107_25.md)    
- 模拟任意维度的聚集索引组织表存储:   
    - 我们知道时序数据的写入使用堆存储时是乱序存储, 如果要查询某个用户某个时间段的轨迹, 可能会导致SCAN大量的blocks. 而按ID+时间戳的聚集存储可以大幅度减少scan blocks. 但是索引组织表只有一个聚集维度, 而且必须是按PK组织的, 没有实际使用意义. PG内置的include index功能打破了索引组织表的问题, 同时打破了堆存储的问题. 鱼与熊掌可以兼得, 大幅度提升任意维度的时序轨迹查询性能.   
    - [《PostgreSQL index include - 类聚簇表与应用(append only, IoT时空轨迹, 离散多行扫描与返回)》](../201905/20190503_03.md)    
- 时态分析:   
    - ganos, postgis都内置了大量时空类型、计算、索引功能. 时态分析可以根据轨迹计算伴随、相遇、相遇时间等, 广泛应用于公安刑侦.   
    - [《PostgreSQL + PostGIS 时态分析》](../201806/20180607_01.md)    
    - https://help.aliyun.com/document_detail/107501.html  
    - http://postgis.net/docs/reference.html#Temporal  
- 递归(skip index scan)SQL, 快速获取first, last value:   
    - 常用的时序查询, 任意数据量毫秒级返回.   
    - [《PostgreSQL - 时序、IoT类场景 - first_value , last_value , agg , cte , window , recursive》](../202104/20210429_02.md)    
    - [《PostgreSQL 排序去重limit查询优化 - 递归 vs group分组 (loop降到极限, block scan降到极限)》](../202005/20200515_01.md)    
    - [《时序数据合并场景加速分析和实现 - 复合索引，窗口分组查询加速，变态递归加速》](../201611/20161128_01.md)     
- 柱状图SQL:   
    - 常用的时序分析.  
    - [《PostgreSQL - 时序、IoT类场景 - 自定义histogram函数, 绘制数据分布柱状图 - cte window range width_bucket format plpgsql》](../202104/20210429_01.md)     
- time_bucket SQL:  
    - 常用的按时间范围分片分析, 如按时间片进行柱状图绘制.  
    - [《PostgreSQL - 时序、IoT类场景 - time_bucket 分析函数 - 内置 date_bin》](../202104/20210429_03.md)    
- madlib:   
    - PostgreSQL 内置的开源机器学习引擎, 广泛应用于时序OLAP场景.     
    - [《PostgreSQL 机器学习插件 MADlib 发布1.18 , 大量深度学习, 自动机器学习等新功能和增强.》](../202104/20210415_02.md)    
    - [《一张图看懂MADlib能干什么》](../201511/20151111_01.md)    
    - [《PostgreSQL MADlib 图(Graph)相关机器学习算法介绍》](../202005/20200525_03.md)    
    - [《PostgreSQL 多元线性回归 - 1 MADLib Installed in PostgreSQL 9.2》](../201307/20130731_01.md)    
              
## 5、PG的解决方案原理              
      
              
## 6、PG的解决方案 VS 业界解决方案              
      
PG 的时序解决方案堪称完美, 要啥有啥.    
        
## 7、DEMO              
              
准备工作              
ECS , Linux + PostgreSQL 客户端软件              
阿里云 RDS PostgreSQL 13       
        
              
### 7.1、时空、时态、时序、日志等轨迹系统               
          
知识量太大, 只挑几个功能点和场景进行DEMO介绍.  
  
1、海量写入性能  
2、模拟任意维度聚集索引与高效率轨迹查询  
3、使用轨迹类型和高效率时态分析  
4、对比时序索引和普通btree索引  
5、行、列、冷、热、混合分区  
6、并行计算  
7、递归获取first,last value  
8、绘制柱状图  
9、异步消息与触发告警  
  
  
              
## 8、知识点回顾              
              
brin 时序索引  
ganos 时空引擎  
postgis 时空引擎  
轨迹数据类型  
轨迹应用之时态分析  
流计算  
异步消息  
including index 类聚簇表   
column store 列存储  
compress 压缩存储  
partition table 行、列、冷、热 混合存储  
oss_fdw 冷热分离外部表   
oss 函数计算  
madlib 机器学习  
  
              
## 9、参考文档              
[《PostgreSQL 15大惊奇应用场景实践 - 直播预告》](../202009/20200903_02.md)              
          
https://help.aliyun.com/document_detail/95580.html      
  
https://help.aliyun.com/document_detail/44461.htm    
        
[《PostgreSQL 时序最佳实践 - 证券交易系统数据库设计 - 阿里云RDS PostgreSQL最佳实践》](../201704/20170417_01.md)    
  
[《泛电网系统(智能电表、智能燃气表等) 海量实时计算+OLTP+OLAP DB设计 - 阿里云(RDS、HybridDB) for PostgreSQL最佳实践》](../201708/20170826_01.md)   
  
[《PostgreSQL IoT，车联网 - 实时轨迹、行程实践 1》](../201812/20181207_01.md)    
  
[《PostgreSQL IoT，车联网 - 实时轨迹、行程实践 2 - (含index only scan类聚簇表效果)》](../201812/20181209_01.md)    
  
[《PostgreSQL index include - 类聚簇表与应用(append only, IoT时空轨迹, 离散多行扫描与返回)》](../201905/20190503_03.md)    
  
[《PostgreSQL 实时位置跟踪+轨迹分析系统实践 - 单机顶千亿轨迹/天》](../201712/20171231_01.md)    
  
[《菜鸟末端轨迹 - 电子围栏(解密支撑每天251亿个包裹的数据库) - 阿里云RDS PostgreSQL最佳实践》](../201708/20170803_01.md)    
  
[《PostgreSQL 物流轨迹系统数据库需求分析与设计 - 包裹侠实时跟踪与召回》](../201704/20170418_01.md)    
  
[《PostgreSQL 聚集存储 与 BRIN索引 - 高并发行为、轨迹类大吞吐数据查询场景解说》](../201702/20170219_01.md)    
      
[《PostgreSQL 时序数据库设计最佳实践 - 关联 citus,columnar,partition,timescaledb,压缩,高速写,parallel append 多分区并行查询,分区》](../202104/20210428_03.md)    
  
[《PostgreSQL 排序去重limit查询优化 - 递归 vs group分组 (loop降到极限, block scan降到极限)》](../202005/20200515_01.md)    
  
[《时序数据合并场景加速分析和实现 - 复合索引，窗口分组查询加速，变态递归加速》](../201611/20161128_01.md)    
         
[《PostgreSQL jdbc batch insert》](../201703/20170329_03.md)    
  
[《PostgreSQL 流式处理应用实践 - 二手商品实时归类(异步消息notify/listen、阅后即焚)》](../201807/20180713_03.md)    
  
[《PostgreSQL 异步消息实践 - Feed系统实时监测与响应(如 电商主动服务) - 分钟级到毫秒级的实现》](../201711/20171111_01.md)    
  
[《从电波表到数据库小程序之 - 数据库异步广播(notify/listen)》](../201701/20170116_01.md)    
  
[《PostgreSQL - 时序、IoT类场景 - time_bucket 分析函数 - 内置 date_bin》](../202104/20210429_03.md)    
  
[《PostgreSQL - 时序、IoT类场景 - first_value , last_value , agg , cte , window , recursive》](../202104/20210429_02.md)    
  
[《PostgreSQL - 时序、IoT类场景 - 自定义histogram函数, 绘制数据分布柱状图 - cte window range width_bucket format plpgsql》](../202104/20210429_01.md)    
    
[《PostgreSQL+MySQL 联合解决方案 - 第6课视频 - PG外部表妙用 - mysql_fdw, oss_fdw（直接读写mysql、冷热分离、归档存储）》](../202001/20200108_01.md)    
  
[《阿里云RDS PostgreSQL OSS 外部表实践 - (dblink异步调用封装并行) 从OSS并行导入数据》](../201804/20180427_01.md)    
  
[《阿里云RDS PostgreSQL OSS 外部表实践 - (dblink异步调用封装并行) 数据并行导出到OSS》](../201709/20170906_01.md)    
  
[《PostgreSQL 时序数据库插件 timescaleDB 部署实践(含例子 纽约TAXI数据透视分析) - PostGIS + timescaleDB => PG时空数据库》](../201801/20180129_01.md)    
  
[《PostgreSQL 按需切片的实现(TimescaleDB插件自动切片功能的plpgsql schemaless实现)》](../201711/20171102_02.md)    
  
[《时序数据库有哪些特点? TimescaleDB时序数据库介绍》](../201704/20170409_05.md)    
  
[《使用Postgres，MobilityDB和Citus大规模(百亿级)实时分析GPS轨迹》](../202011/20201117_01.md)    
  
https://help.aliyun.com/document_detail/95580.html  
  
http://postgis.net/docs/reference.html#Temporal  
  
https://help.aliyun.com/document_detail/107501.html  
  
[《PostgreSQL + PostGIS 时态分析》](../201806/20180607_01.md)    
  
[《SQL流式案例 - 旋转门压缩(前后计算相关滑窗处理例子)》](../201801/20180107_01.md)    
  
[《旋转门数据压缩算法在PostgreSQL中的实现 - 流式压缩在物联网、监控、传感器等场景的应用》](../201608/20160813_01.md)    
  
[《不懂jsonpath的话等于JSON没入门》](../202105/20210507_04.md)    
  
[《PostgreSQL 史上最强JSON功能 - PG 12 jsonpath 完全超越oracle, mysql, sql server的sql json标准覆盖率》](../202010/20201013_01.md)    
  
[《怎么看待PostgreSQL sharding插件: citus - 对比PG 社区基于 postgres_fdw 的sharding》](../202103/20210325_02.md)    
  
[《PostgreSQL citus 发布 10版本 - 支持columner 列存储, 开放rebalance源码, 支持reference,分布式,本地表JOIN 等》](../202103/20210307_03.md)    
  
[《PostgreSQL sharding extension citus 优化器 Query Processing 之 - Distributed Query Planner、Executor (Real-time Executor, Router Executor, Task Tracker Executor)》](../201903/20190316_02.md)    
  
[《PostgreSQL sharding extensino citus 优化器 Query Processing 之 - Subquery/CTE Push-Pull Execution》](../201903/20190316_01.md)    
  
[《PostgreSQL sharding : citus 系列7 - topn 加速(```count(*) group by order by count(*) desc limit x```) (use 估值插件 topn)》](../201809/20180914_01.md)    
  
[《PostgreSQL sharding : citus 系列6 - count(distinct xx) 加速 (use 估值插件 hll|hyperloglog)》](../201809/20180913_04.md)    
  
[《PostgreSQL sharding : citus 系列5 - worker节点网络优化》](../201809/20180905_02.md)    
  
[《PostgreSQL sharding : citus 系列4 - DDL 操作规范 (新增DB，TABLE，SCHEMA，UDF，OP，用户等)》](../201809/20180905_01.md)    
  
[《PostgreSQL sharding : citus 系列3 - 窗口函数调用限制 与 破解之法(套用gpdb执行树,分步执行)》](../201809/20180902_01.md)    
  
[《PostgreSQL sharding : citus 系列2 - TPC-H》](../201808/20180829_01.md)    
  
[《PostgreSQL citus, Greenplum  分布式执行计划 DEBUG》](../201808/20180828_01.md)    
  
[《PostgreSQL sharding : citus 系列1 - 多机部署（含OLTP(TPC-B)测试）- 含Citus MX模式》](../201808/20180824_02.md)    
  
[《PostgreSQL 并行计算解说 汇总》](../201903/20190318_05.md)    
  
[《PostgreSQL 14 preview - TOAST 支持 lz4 压缩算法 - --with-lz4 , 新增GUC default_toast_compression》](../202103/20210320_01.md)    
  
[《PostgreSQL appendonly 压缩 存储引擎 - pg_cryogen》](../202003/20200324_10.md)    
  
[《使用PostgreSQL plpythonu实现一个推荐引擎 - Building a recommendation engine inside Postgres with Python and Pandas》](../202103/20210311_02.md)    
  
[《AWS redshift->hdb pg(Greenplum)， plpython, pljava UDF 以及upload library》](../201810/20181017_04.md)    
  
[《[未完待续] PostgreSQL 流式fft傅里叶变换 (plpython + numpy + 数据库流式计算)》](../201803/20180307_01.md)    
  
[《在PostgreSQL中使用 plpythonu 调用系统命令》](../201710/20171023_01.md)    
  
[《HTAP数据库 PostgreSQL 场景与性能测试之 24 - (OLTP) 物联网 - 时序数据并发写入(含时序索引BRIN)》](../201711/20171107_25.md)    
  
[《PostgreSQL 机器学习插件 MADlib 发布1.18 , 大量深度学习, 自动机器学习等新功能和增强.》](../202104/20210415_02.md)    
  
[《一张图看懂MADlib能干什么》](../201511/20151111_01.md)    
  
[《PostgreSQL MADlib 图(Graph)相关机器学习算法介绍》](../202005/20200525_03.md)    
  
[《PostgreSQL 多元线性回归 - 1 MADLib Installed in PostgreSQL 9.2》](../201307/20130731_01.md)    
  
[《从微信小程序 到 数据库"小程序" , 鬼知道我经历了什么》](../201701/20170113_03.md)    
  
  
#### [PostgreSQL 许愿链接](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216")
您的愿望将传达给PG kernel hacker、数据库厂商等, 帮助提高数据库产品质量和功能, 说不定下一个PG版本就有您提出的功能点. 针对非常好的提议，奖励限量版PG文化衫、纪念品、贴纸、PG热门书籍等，奖品丰富，快来许愿。[开不开森](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216").  
  
  
#### [9.9元购买3个月阿里云RDS PostgreSQL实例](https://www.aliyun.com/database/postgresqlactivity "57258f76c37864c6e6d23383d05714ea")
  
  
#### [PostgreSQL 解决方案集合](https://yq.aliyun.com/topic/118 "40cff096e9ed7122c512b35d8561d9c8")
  
  
#### [德哥 / digoal's github - 公益是一辈子的事.](https://github.com/digoal/blog/blob/master/README.md "22709685feb7cab07d30f30387f0a9ae")
  
  
![digoal's wechat](../pic/digoal_weixin.jpg "f7ad92eeba24523fd47a6e1a0e691b59")
  
