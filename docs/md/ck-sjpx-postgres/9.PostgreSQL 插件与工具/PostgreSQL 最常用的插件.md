## PostgreSQL 最常用的插件    
    
### 作者    
digoal    
    
### 日期    
2020-09-15    
    
### 标签    
PostgreSQL , 插件    
    
----    
    
## 背景    
    
## 监控、优化、诊断    
cpu, io消耗监控    
https://github.com/powa-team/pg_stat_kcache    
    
where条件过滤性统计    
https://github.com/powa-team/pg_qualstats    
    
cgroup , /proc 系统级指标统计    
https://github.com/CrunchyData/pgnodemx    
    
pg_logging, log buffer read    
价值: 读pip管道中的log内容, 不需要写双份日志    
https://github.com/postgrespro/pg_logging    
    
当前正在执行sql的backend的状态统计    
https://github.com/postgrespro/pg_query_state    
    
执行计划固化和篡改    
https://github.com/postgrespro/sr_plan    
    
等待事件采样统计    
https://github.com/postgrespro/pg_wait_sampling    
    
系统级监控统计    
https://github.com/EnterpriseDB/system_stats    
    
pgmetrics    
https://github.com/digoal/blog/blob/master/201810/20181001_03.md    
    
csvlog 分析, pgbadger    
http://pgbadger.darold.net/    
    
《PostgreSQL DBA最常用SQL》    
https://github.com/digoal/blog/blob/master/202005/20200509_02.md    
    
《PostgreSQL 活跃会话历史记录插件 - pgsentinel 类似performance insight》    
https://github.com/digoal/blog/blob/master/202003/20200324_25.md    
https://github.com/digoal/blog/blob/master/201901/20190125_02.md    
    
pg_hint_plan, 执行计划hint    
http://pghintplan.osdn.jp/pg_hint_plan.html    
    
自动执行计划优化, pg_plan_advsr    
https://github.com/digoal/blog/blob/master/202003/20200324_33.md    
    
PostgreSQL dba常用扩展函数库 - pg_cheat_funcs    
https://github.com/digoal/blog/blob/master/202003/20200324_41.md    
    
数据库job功能 pg_cron    
https://github.com/digoal/blog/blob/master/201305/20130531_01.md    
    
pg_pathman, 分区表性能提升, 自动分区, 自动增加分区等    
https://github.com/postgrespro/pg_pathman    
    
## 功能    
postgis , 全球最流行的GIS扩展功能模块    
http://postgis.org/    
    
h3模块    
https://github.com/dlr-eoc/pgh3    
    
pgrouting, 图路由功能    
https://pgrouting.org/    
    
《PostgreSQL MOD数据库 MobilityDB - GIS 移动对象处理数据库》    
https://github.com/digoal/blog/blob/master/202003/20200324_24.md    
    
开源图像识别, imgsmlr    
https://github.com/digoal/blog/blob/master/201811/20181129_01.md    
https://github.com/postgrespro/imgsmlr    
    
pg_roaringbitmap, 用户圈选、实时画像、相似用户扩选功能 . 分析场景功能插件，应用于实时精准营销系统    
https://github.com/digoal/blog/blob/master/201911/20191118_01.md    
    
timescaledb 时序数据库引擎    
https://github.com/digoal/blog/blob/master/201804/20180420_01.md    
https://github.com/timescale/timescaledb    
    
pg_prometheus, IoT常用, 监控产品集成到timescaledb和prometheus    
https://github.com/timescale/pg_prometheus    
    
机器学习 madlib    
http://madlib.apache.org/    
    
rdkit 化学插件, 化学分析行业应用    
https://github.com/digoal/blog/blob/master/202003/20200326_06.md    
https://github.com/digoal/blog/blob/master/201911/20191125_01.md    
    
文本处理插件 pg_bigm, PGroonga. 任意内容模糊搜索(包括单字和双字的模糊查询)    
https://github.com/digoal/blog/blob/master/202003/20200330_01.md    
[《[直播]在数据库中跑全文检索、模糊查询SQL会不会被开除?》](../202009/20200913_01.md)      
    
pg_trgm, 实时模糊查询、相似文本查询    
https://github.com/digoal/blog/blob/master/201809/20180904_01.md    
    
pg_jieba 中文分词    
https://github.com/digoal/blog/blob/master/201607/20160725_02.md    
    
pg_scws 中文分词    
https://github.com/jaiminpan/pg_scws    
    
zhparser 中文分词    
https://github.com/digoal/blog/blob/master/201603/20160310_01.md    
    
smlar, 多值类型相似查询, 支持cosine, overlap等相似算法, 例如 标签交叉率相似度, 用户画像相似圈选    
http://sigaev.ru/git/gitweb.cgi?p=smlar.git;a=summary    
    
hll, 近似HASH值, 高效UNIQUE近似比对、计算, 滑窗分析. 思想上类似bloom算法, 每个value映射到hash值的某些bit上, 用若干bit位的占位是否为1表示这个value是否存在.    
https://github.com/citusdata/postgresql-hll    
[《[视频直播]亿级用户量的实时推荐数据库到底要几毛钱?》](../202009/20200910_02.md)      
    
orafce, 常用Oracle兼容类型、函数、包    
https://github.com/digoal/blog/blob/master/201603/20160324_01.md    
    
pgcrypto, 数据加密模块    
https://www.postgresql.org/docs/current/pgcrypto.html    
    
采样查询, 随机数据查询, 比offset random性能提升1万倍    
tsm_system_rows    
tsm_system_time    
https://github.com/digoal/blog/blob/master/202005/20200509_01.md    
    
rum, 加强版倒排索引, 多值类型(数组、文本向量等)检索加速, 任意字段组合搜索索引过滤    
https://github.com/postgrespro/rum    
https://github.com/digoal/blog/blob/master/201907/20190706_01.md    
    
zombodb: 创建基于es的索引, 加速搜索, 一种全新解决方案, 降低用户开发成本, 简化应用架构, 解决搜索数据同步问题、延迟等痛点.    
https://github.com/zombodb/zombodb    
https://github.com/digoal/blog/blob/master/201710/20171001_06.md    
    
阿里云pase, 图像识别, 向量检索, 相似特征圈选. 千万级别毫秒响应    
https://github.com/digoal/blog/blob/master/202004/20200424_01.md    
    
    
sql审计sql audit    
https://github.com/digoal/blog/blob/master/201505/20150515_01.md    
    
    
## 连接池    
pgbouncer     
http://www.pgbouncer.org/    
    
《PostgreSQL 多线程连接池 - Scalable PostgreSQL connection pooler odyssey》    
https://github.com/digoal/blog/blob/master/201906/20190624_01.md    
    
pgagroal    
https://github.com/digoal/blog/blob/master/202003/20200320_02.md    
    
## 外部访问    
阿里云oss_fdw冷热分离存储    
https://github.com/digoal/blog/blob/master/202001/20200108_01.md    
    
其他fdw外部表汇总    
https://wiki.postgresql.org/wiki/Foreign_data_wrappers    
    
外部数据源访问 dblink    
pg, oracle, mysql, sqlite dblink plus 打通多种数据库, 解决大型企业跨数据源访问问题(传统企业尤为普遍)    
https://github.com/digoal/blog/blob/master/202003/20200324_07.md    
    
外部数据访问 udf, 直接使用sql操作其他数据库    
memcached    
https://github.com/ohmu/pgmemcache    
    
redis    
https://github.com/digoal/blog/blob/master/202003/20200326_09.md    
    
## sharding    
plproxy    
https://plproxy.github.io/    
    
citus    
https://github.com/citusdata/citus    
    
## 读写分离    
pgpool    
http://www.pgpool.org/    
《阿里云RDS PostgreSQL 12 + pgpool 的读写分离配置》    
    
《PostgreSQL druid 多个读节点的jdbc loadbalance负载均衡连接串配置》    
https://github.com/digoal/blog/blob/master/202002/20200214_03.md    
    
《PostgreSQL libpq|jdbc 驱动层 load balance 与 failover》    
https://github.com/digoal/blog/blob/master/201910/20191027_01.md    
    
《PostgreSQL jdbc multi-host 配置与简单HA、Load Balance实现》    
https://github.com/digoal/blog/blob/master/201806/20180614_02.md    
    
    
## 异构迁移    
oracle 2 pg    
http://www.ora2pg.com/    
    
《PostgreSQL pgloader - 一键迁移 MySQL SQLite MS SQL Redshift, csv dbf ixf archive》    
https://github.com/digoal/blog/blob/master/202003/20200324_02.md    
    
    
## 日常维护    
在线收缩膨胀空间    
https://github.com/reorg/pg_repack    
    
    
    
## 存储    
    
zedstore, 行列混合存储, 更好的支持分析场景(cpu vector 批量计算, 列存储, 压缩效率)    
https://github.com/greenplum-db/postgres/tree/zedstore/src/backend/access/zedstore    
    
undam, UNDO存储引擎, 更好的支持update, 减轻膨胀    
https://github.com/postgrespro/undam    
    
zheap, UNDO存储引擎, 更好的支持update, 减轻膨胀    
https://github.com/EnterpriseDB/zheap    
    
PostgreSQL appendonly 压缩 存储引擎 - pg_cryogen, 更好的支持分析场景(cpu vector 批量计算, 列存储, 压缩效率)    
https://github.com/digoal/blog/blob/master/202003/20200324_10.md    
    
PostgreSQL LSM-tree 引擎 - VidarDB (baseon rocksdb) , 更高的插入效率, 牺牲查询效率    
https://github.com/digoal/blog/blob/master/202003/20200324_03.md    
https://en.wikipedia.org/wiki/Log-structured_merge-tree    
    
    
    
    
## 内置插件    
https://www.postgresql.org/docs/current/contrib.html    
    
用得比较多的插件:    
    
auth_delay, 防止暴力破解    
auto_explain, 将慢sql的执行计划写入日志    
btree_gin, 普通字段支持gin倒排索引    
btree_gist, 普通字段支持gist索引    
citext, 忽略大小写的数据类型    
dblink, 不解释    
earthdistance, 轻量化地球模型类型    
file_fdw, 以外部表的形式读写文件    
fuzzystrmatch, 语音模糊搜索    
intagg, 整型聚合功能扩展    
intarray, 整型数组GIST索引扩展功能    
lo, 大对象处理    
pageinspect, 内窥数据库BLOCK的内容    
passwordcheck, 密码复杂度检测    
pg_buffercache, 统计数据库shared buffer的内容    
pgcrypto, 加密插件    
pg_freespacemap, 观察数据库fsm文件内容    
pgrowlocks, 行锁统计    
pgstattuple, 记录级别统计信息观察    
pg_trgm, 模糊查询, 相似文本查询    
pg_visibility, 观察数据库block的vm标签值(all visibility, frozen, dirty等)    
postgres_fdw, postgresql外部表    
spi, 一些常用的跟踪触发器函数, 例如最后变更事件跟踪    
test_decoding, 逻辑复制decoder    
tsm_system_rows, 采样扩展模块    
tsm_system_time, 采样扩展模块    
uuid-ossp, UUID生成模块    
xml2, XML类型模块    
pg_stat_statements, 观察数据库的sql运行统计信息, 例如top sql    
cube, cube类型, 支持多种距离排序算法, 也可以用于相似向量搜索    
ltree, 树类型    
pg_prewarm, buffer预热功能    
tablefunc, 行列变换插件    
    
## 其他请参考    
[《未来数据库方向 - PostgreSQL 有价值的插件、可改进功能、开放接口 (202005)》](../202005/20200527_06.md)      
    
[其他参考信息](../README.md)    
    
    
    
  
#### [PostgreSQL 许愿链接](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216")
您的愿望将传达给PG kernel hacker、数据库厂商等, 帮助提高数据库产品质量和功能, 说不定下一个PG版本就有您提出的功能点. 针对非常好的提议，奖励限量版PG文化衫、纪念品、贴纸、PG热门书籍等，奖品丰富，快来许愿。[开不开森](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216").  
  
  
#### [9.9元购买3个月阿里云RDS PostgreSQL实例](https://www.aliyun.com/database/postgresqlactivity "57258f76c37864c6e6d23383d05714ea")
  
  
#### [PostgreSQL 解决方案集合](https://yq.aliyun.com/topic/118 "40cff096e9ed7122c512b35d8561d9c8")
  
  
#### [德哥 / digoal's github - 公益是一辈子的事.](https://github.com/digoal/blog/blob/master/README.md "22709685feb7cab07d30f30387f0a9ae")
  
  
![digoal's wechat](../pic/digoal_weixin.jpg "f7ad92eeba24523fd47a6e1a0e691b59")
  
