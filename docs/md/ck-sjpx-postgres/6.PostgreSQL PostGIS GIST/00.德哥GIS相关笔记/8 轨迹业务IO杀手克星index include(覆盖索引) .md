## 重新发现PostgreSQL之美 - 8 轨迹业务IO杀手克星index include(覆盖索引)  

### 作者      
digoal      
      
### 日期      
2021-05-30      
      
### 标签      
PostgreSQL , include index , 聚集 , 轨迹业务 , 金融 , IO杀手 , IO放大        
      
----

## 背景      
视频回放: https://www.bilibili.com/video/BV1tV41177rY/       
    
场景痛点:     
- 轨迹类业务, 一个轨迹由多个点组成, 每个点的ROW写入散落到不同的PAGE, 查询一条轨迹可能要回表访问上百千个PAGE, 号称IO杀手.    
  

业务:    
- 共享单车、巡逻车、公务用车、网约车、金融行业股票数据、物联网行业传感器数据等.     
  

PG index include (覆盖索引)功能:     
- 重组存储结构, 按指定维度聚集.     
- 叶子结点存储include column value, 无需回表(轨迹数据都是append only的, VM bit全部都是clean page, 因此无需回表).     
  

语法:    
    
```    
Command:     CREATE INDEX    
Description: define a new index    
Syntax:    
CREATE [ UNIQUE ] INDEX [ CONCURRENTLY ] [ [ IF NOT EXISTS ] name ] ON [ ONLY ] table_name [ USING method ]    
    ( { column_name | ( expression ) } [ COLLATE collation ] [ opclass [ ( opclass_parameter = value [, ... ] ) ] ] [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] )    
    [ INCLUDE ( column_name [, ...] ) ]    
    [ WITH ( storage_parameter [= value] [, ... ] ) ]    
    [ TABLESPACE tablespace_name ]    
    [ WHERE predicate ]    
    
URL: https://www.postgresql.org/docs/14/sql-createindex.html    
```

## 例子    

共享单车轨迹    
    
```    
create unlogged table tbl_sensor_track    
(    
id serial8 primary key,    
sid int,  -- 单车ID    
pos point,  -- 位置    
traceid int, -- 轨迹ID    
info text,  -- 其他信息    
crt_time timestamp  -- 时间    
);    
    
create index idx_tbl_sensor_track_1 on tbl_sensor_track (sid,traceid,crt_time);    
```

写入压测数据, 1000万条, 1000量单车, 每辆单车约10个轨迹, 每个轨迹约1000条记录.      
    
```    
vi test.sql    
\set sid random(1,1000)    
insert into tbl_sensor_track (sid,pos,traceid,info,crt_time) values    
(:sid, point(random(),random()), random()*10, md5(random()::text), clock_timestamp());    
    
pgbench -M prepared -n -r -P 1 -f ./test.sql -c 10 -j 10 -t 1000000    
    
    
vacuum analyze tbl_sensor_track;    
    
select * from tbl_sensor_track where sid=1 and traceid=1 order by crt_time;    
    
postgres=# explain (analyze,verbose,timing,costs,buffers) select * from tbl_sensor_track where sid=1 and traceid=1 order by crt_time;    
                                                                       QUERY PLAN                                                                           
--------------------------------------------------------------------------------------------------------------------------------------------------------    
 Index Scan using idx_tbl_sensor_track_1 on public.tbl_sensor_track  (cost=0.43..1099.13 rows=980 width=73) (actual time=0.042..1.875 rows=973 loops=1)    
   Output: id, sid, pos, traceid, info, crt_time    
   Index Cond: ((tbl_sensor_track.sid = 1) AND (tbl_sensor_track.traceid = 1))    
   Buffers: shared hit=976    
 Planning Time: 0.144 ms    
 Execution Time: 2.072 ms    
(6 rows)    
    
Time: 3.119 ms    
```

为什么要访问这么多数据块?    
传感器都是活跃的, 大家都在写, 某个sid的多条数据必定会分散到多个PAGE中.     
    
```    
postgres=# select ctid,* from tbl_sensor_track where sid=1 and traceid=1 order by crt_time;    
    ctid     |   id    | sid |                     pos                      | traceid |               info               |          crt_time              
-------------+---------+-----+----------------------------------------------+---------+----------------------------------+----------------------------    
 (289,70)    |   24022 |   1 | (0.3291194292095341,0.7362849779331277)      |       1 | dd989480a73b6ccf0988cd2ce4594c41 | 2021-05-30 12:12:35.221626    
 (316,61)    |   24046 |   1 | (0.30092918298355187,0.7515717517477718)     |       1 | 0f563c5e6fae4ff5335d3c1a6c6d8af7 | 2021-05-30 12:12:35.221854    
 (374,24)    |   27822 |   1 | (0.7082108853392377,0.8023826990838856)      |       1 | cf24c90af1a0cf3211d2be5f6ba09466 | 2021-05-30 12:12:35.26342    
 (387,23)    |   29083 |   1 | (0.6827721329130654,0.6080967981321557)      |       1 | 33a812ceeb690147acaab7b2e3602eaf | 2021-05-30 12:12:35.276581    
 (506,73)    |   38479 |   1 | (0.2819613389873581,0.8749003165249825)      |       1 | e34d53dccf0b5cb9b7e30366a3535f2c | 2021-05-30 12:12:35.376353    
 (1096,75)   |   82442 |   1 | (0.1818822075073001,0.508428318796188)       |       1 | ba3c9669eac963976fcf28f51d33d810 | 2021-05-30 12:12:35.850693    
 (1429,69)   |  107646 |   1 | (0.14452220616886535,0.42080454115547994)    |       1 | 74d3995610885086c7b60744c2feebca | 2021-05-30 12:12:36.129595    
 (1451,54)   |  109320 |   1 | (0.13488553655361812,0.9924723125826809)     |       1 | 3ea411a5dc265fc56887817ca61580c2 | 2021-05-30 12:12:36.146846    
 (1566,59)   |  118072 |   1 | (0.0021032854500404596,0.5327942934971936)   |       1 | bac5b2b038d7e775251df93d65bce9af | 2021-05-30 12:12:36.243577    
 (1677,4)    |  125396 |   1 | (0.11318825440940117,0.46600202700733817)    |       1 | f41da3b991c5ad55a5d9c11993a1f396 | 2021-05-30 12:12:36.322615    
 (1699,1)    |  127190 |   1 | (0.1162984003478087,0.47079561917773916)     |       1 | ba1be1fc77621faacaa0ca4a2fa289f9 | 2021-05-30 12:12:36.341182    
 (2195,1)    |  157052 |   1 | (0.7798969830712075,0.29867198696907593)     |       1 | 346bd94b5be3d30cc749efc939b04c89 | 2021-05-30 12:12:36.658956    
 (2204,39)   |  165303 |   1 | (0.5289847732077746,0.9900722786928533)      |       1 | 3a8eb11558768555f3b04001702c80a7 | 2021-05-30 12:12:36.746148    
 (2284,58)   |  171596 |   1 | (0.7207220490941282,0.7717989719965779)      |       1 | 90f10c713ec430a938b4a6f2d82ad6c2 | 2021-05-30 12:12:36.810753    
 (2464,24)   |  184864 |   1 | (0.7473279005965843,0.036199381685484866)    |       1 | 37705afad1f8be5ab53e89a27c2ddde4 | 2021-05-30 12:12:36.952249    
 (2492,58)   |  186995 |   1 | (0.2527484469419434,0.812076005344526)       |       1 | cdbfb7ff0ec5c85e6eb29cebc1604ce9 | 2021-05-30 12:12:36.975931    
 (2602,64)   |  195112 |   1 | (0.3987292028407765,0.5321831181795922)      |       1 | ccf72d761d93b6834e5586f69ff7ac97 | 2021-05-30 12:12:37.06301    
 (2624,29)   |  196765 |   1 | (0.7868492865049959,0.3834753392663437)      |       1 | 59da61df594812fd801f70361d3b9b0e | 2021-05-30 12:12:37.079775    
 (2662,36)   |  199557 |   1 | (0.4178753519891032,0.6877918734552573)      |       1 | 807c6dcfb2aabae2decbce3f3f502d93 | 2021-05-30 12:12:37.11165    
 (2686,11)   |  201309 |   1 | (0.2643845772676734,0.9410469934455321)      |       1 | 19bdddf814473489670604e658f60cd1 | 2021-05-30 12:12:37.130968    
 (2736,52)   |  205340 |   1 | (0.052081088208076665,0.05945480161325989)   |       1 | 9bdcfc79a023d2e924085bf0d4f46c28 | 2021-05-30 12:12:37.172398    
 (2786,55)   |  209333 |   1 | (0.11154442604792081,0.27523521858215716)    |       1 | bdc9393ed3448f6084dd99638a49a576 | 2021-05-30 12:12:37.21326    
 (2811,67)   |  211280 |   1 | (0.764354185520979,0.46827810976283857)      |       1 | 7b332c186e088fcd44e616ede4f1f33a | 2021-05-30 12:12:37.235043    
 (2821,57)   |  211800 |   1 | (0.9366284195602432,0.14309645108922453)     |       1 | 5268a745de0e8d951fbe6b35b1ae53ed | 2021-05-30 12:12:37.240254    
 (2886,44)   |  216725 |   1 | (0.7654642711557571,0.08884990148832017)     |       1 | a19c9ae847d408b51ea5c6171c1ba5a4 | 2021-05-30 12:12:37.291206    
...     
    
```

覆盖索引:     
无需回表, 访问的数据块从976下降到22    
    
```    
create index idx_tbl_sensor_track_2 on tbl_sensor_track (sid,traceid,crt_time) include (id,pos,info);     
```

```    
postgres=# explain (analyze,verbose,timing,costs,buffers) select * from tbl_sensor_track where sid=1 and traceid=1 order by crt_time;    
                                                                         QUERY PLAN                                                                             
------------------------------------------------------------------------------------------------------------------------------------------------------------    
 Index Only Scan using idx_tbl_sensor_track_2 on public.tbl_sensor_track  (cost=0.56..34.92 rows=1003 width=73) (actual time=0.033..0.542 rows=973 loops=1)    
   Output: id, sid, pos, traceid, info, crt_time    
   Index Cond: ((tbl_sensor_track.sid = 1) AND (tbl_sensor_track.traceid = 1))    
   Heap Fetches: 0    
   Buffers: shared hit=22    
 Planning Time: 0.166 ms    
 Execution Time: 0.742 ms    
(7 rows)    
    
    
postgres=# \dt+    
                                         List of relations    
 Schema |       Name       | Type  |  Owner   | Persistence | Access method |  Size   | Description     
--------+------------------+-------+----------+-------------+---------------+---------+-------------    
 public | tbl_sensor_track | table | postgres | unlogged    | heap          | 1042 MB |     
(1 row)    
    
postgres=# \di+    
                                                     List of relations    
 Schema |          Name          | Type  |  Owner   |      Table       | Persistence | Access method |  Size  | Description     
--------+------------------------+-------+----------+------------------+-------------+---------------+--------+-------------    
 public | idx_tbl_sensor_track_1 | index | postgres | tbl_sensor_track | unlogged    | btree         | 301 MB |     
 public | idx_tbl_sensor_track_2 | index | postgres | tbl_sensor_track | unlogged    | btree         | 994 MB |     
 public | tbl_sensor_track_pkey  | index | postgres | tbl_sensor_track | unlogged    | btree         | 214 MB |     
(3 rows)    
```

并发能力压测      
    
```    
vi test.sql    
select * from tbl_sensor_track where sid=1 and traceid=1 order by crt_time;    
```

全内存命中的情况下, 差异较小, 但是实际生产环境中数据不可能全部在内存中, 此时IO带来的问题就会凸显, 性能差异明显.      
    
非聚集索引    
    
```    
pgbench (PostgreSQL) 14.0    
transaction type: ./test.sql    
scaling factor: 1    
query mode: prepared    
number of clients: 12    
number of threads: 12    
duration: 120 s    
number of transactions actually processed: 366951    
latency average = 3.924 ms    
latency stddev = 10.133 ms    
initial connection time = 12.210 ms    
tps = 3058.035523 (without initial connection time)    
statement latencies in milliseconds:    
         3.925  select * from tbl_sensor_track where sid=1 and traceid=1 order by crt_time;    
```

聚集覆盖索引    
    
```    
pgbench (PostgreSQL) 14.0    
transaction type: ./test.sql    
scaling factor: 1    
query mode: prepared    
number of clients: 12    
number of threads: 12    
duration: 120 s    
number of transactions actually processed: 460368    
latency average = 3.127 ms    
latency stddev = 8.477 ms    
initial connection time = 12.219 ms    
tps = 3836.628407 (without initial connection time)    
statement latencies in milliseconds:    
         3.128  select * from tbl_sensor_track where sid=1 and traceid=1 order by crt_time;    
```


​    
​    
## include index解决了什么问题?    
1、在某个维度上查询需要返回N条记录, N条记录在HEAP PAGE中非常分散, 需要耗费大量IO的问题.    
- 不需要联合索引, 减少索引build的时间和复杂度, 并且索引有<1/3 index page的限制, 复合索引会导致数据超长写入失败.    
  

2、支持按任意维度查询, 每个维度都需要返回N条, N条记录在HEAP PAGE中非常分散, 需要耗费大量IO的问题.    
- index include 可以按任意维度进行聚集存储. 满足不同维度的大范围搜索需求.  比聚集表要厉害: 聚集表只能按1个维度(也就是PK)来进行存储.     
  
## 其他    
除了使用index include, 在业务侧也可以改进来解决离散IO问题, 例如    
- 轨迹合并存储(使用1条记录, 而非存储到N条记录里面.)    
- 按SID分片存储, 每个SID一个表. 这样就不会和其他SID混PAGE了.    
- 轨迹合并可以使用PG的PostGIS或者阿里云ganos的轨迹数据类型(支持轨迹的计算、例如伴随、相似、拟合等分析).    
  
    
##### 202104/20210406_04.md   [《PostgreSQL 14 preview - SP-GiST 索引新增 index 叶子结点 include column value 功能 支持》](../202104/20210406_04.md)      
##### 202011/20201117_01.md   [《使用Postgres，MobilityDB和Citus大规模(百亿级)实时分析GPS轨迹》](../202011/20201117_01.md)      
##### 202004/20200429_01.md   [《PostgreSQL 索引算子下推扩展 - 索引内offset - 索引内过滤 - include index - 随机偏移》](../202004/20200429_01.md)      
##### 201905/20190503_03.md   [《PostgreSQL index include - 类聚簇表与应用(append only, IoT时空轨迹, 离散多行扫描与返回)》](../201905/20190503_03.md)      
##### 201903/20190331_08.md   [《PostgreSQL 12 preview - GiST 索引支持INCLUDE columns - 覆盖索引 - 类聚簇索引》](../201903/20190331_08.md)      
##### 201812/20181209_01.md   [《PostgreSQL IoT，车联网 - 实时轨迹、行程实践 2 - (含index only scan类聚簇表效果)》](../201812/20181209_01.md)      
##### 201812/20181207_01.md   [《PostgreSQL IoT，车联网 - 实时轨迹、行程实践 1》](../201812/20181207_01.md)      
##### 201811/20181101_02.md   [《PostgreSQL pipelinedb 流计算插件 - IoT应用 - 实时轨迹聚合》](../201811/20181101_02.md)      
##### 201806/20180607_02.md   [《Greenplum 轨迹相似(伴随分析)》](../201806/20180607_02.md)      
##### 201712/20171231_01.md   [《PostgreSQL 实时位置跟踪+轨迹分析系统实践 - 单机顶千亿轨迹/天》](../201712/20171231_01.md)      
##### 201712/20171204_01.md   [《GIS术语 - POI、AOI、LOI、路径、轨迹》](../201712/20171204_01.md)      
##### 201708/20170803_01.md   [《菜鸟末端轨迹 - 电子围栏(解密支撑每天251亿个包裹的数据库) - 阿里云RDS PostgreSQL最佳实践》](../201708/20170803_01.md)      
##### 201707/20170722_02.md   [《车联网案例，轨迹清洗 - 阿里云RDS PostgreSQL最佳实践 - 窗口函数》](../201707/20170722_02.md)      
##### 201704/20170418_01.md   [《PostgreSQL 物流轨迹系统数据库需求分析与设计 - 包裹侠实时跟踪与召回》](../201704/20170418_01.md)      
##### 201703/20170312_23.md   [《PostgreSQL 10.0 preview 功能增强 - 唯一约束+附加字段组合功能索引 - 覆盖索引 - covering index》](../201703/20170312_23.md)      
##### 201702/20170219_01.md   [《PostgreSQL 聚集存储 与 BRIN索引 - 高并发行为、轨迹类大吞吐数据查询场景解说》](../201702/20170219_01.md)      
##### 201606/20160611_02.md   [《PostgreSQL 如何轻松搞定行驶、运动轨迹合并和切分》](../201606/20160611_02.md)      
