## 重新发现PostgreSQL之美 - 6 index链表跳跳糖 (CTE recursive 递归的详细用例)  

### 作者  
digoal  

### 日期  
2021-05-29   

### 标签  
PostgreSQL , cte , 递归   

----

## 背景  
CTE 递归语法是PG 8.4引入的功能, 至今已经10多年, 非常文档.  

CTE 递归可以解决很多问题: 时序场景取所有传感器最新的value, 图式数据的搜索(一度人脉,N度人脉,最近的路径关系), 树状数据的累加分析, 知识图谱, 去稀疏数据的唯一值等.  

使用CTE递归比通用的方法通常有数百倍的性能提升.  

## 用例  
假设传感器有1万个, 每个传感器每秒上传一条记录.   
取出今天处于活跃状态(有数据)的传感器的最后一个值.   


1、创建测试表  

```  plsql
create unlogged table tbl_sensor_log (  
  id serial8 ,     
  sid int, -- 传感器ID (例如 网约车、警车、巡逻车、共享单车、物联网传感器等设备)  
  val jsonb, -- 传感器上传的数据  
  crt_time timestamp  -- 上传时间  
)  
partition by range (crt_time)  
;   
```

2、创建分区  

```  plsql
do language plpgsql $$  
declare  
begin  
  for i in 0..365 loop  
    execute format($_$  
        create unlogged table tbl_sensor_log_%s PARTITION of tbl_sensor_log   
        for values from (%L) to (%L)  
    $_$, to_char(current_date+i, 'yyyymmdd'), current_date+i, current_date+i+1);  
  end loop;  
end;  
$$;  
```

3、创建索引  

```  plsql
create index idx_tbl_sensor_log_1 on tbl_sensor_log (sid,crt_time desc);  
```

4、写入5000万条记录, 均匀分布在最近3天的分区内  

```  plsql
insert into tbl_sensor_log (sid, val, crt_time)  
  select random()*10000 , row_to_json(row(random(),random(),clock_timestamp())), current_date+(round(random()::numeric*72::numeric,2)||' hour')::interval  
from generate_series(1,50000000);  
```

取出今天处于活跃状态(有数据)的传感器的最后一个值.   

方法1: 使用传统的窗口查询  

```  plsql
select id,sid,val,crt_time from   
(  
select *, row_number() over w as RN  
  from tbl_sensor_log   
  where crt_time >= current_date and crt_time < current_date+1   
  window w as (partition by sid order by crt_time desc)  
) t  
where rn=1;  
  
  
  
  
  
                                                                                    QUERY PLAN                                                                                      
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
 Subquery Scan on t  (cost=5057407.30..5598899.49 rows=83306 width=119) (actual time=40763.507..52716.622 rows=10001 loops=1)  
   Filter: (t.rn = 1)  
   Rows Removed by Filter: 16653784  
   ->  WindowAgg  (cost=5057407.30..5390633.26 rows=16661298 width=127) (actual time=40763.503..51533.043 rows=16663785 loops=1)  
         ->  Sort  (cost=5057407.30..5099060.55 rows=16661298 width=119) (actual time=40763.483..44945.556 rows=16663785 loops=1)  
               Sort Key: tbl_sensor_log.sid, tbl_sensor_log.crt_time DESC  
               Sort Method: external merge  Disk: 2177080kB  
               ->  Append  (cost=0.00..1257703.57 rows=16661298 width=119) (actual time=0.065..5597.541 rows=16663785 loops=1)  
                     Subplans Removed: 365  
                     ->  Seq Scan on tbl_sensor_log_20210529 tbl_sensor_log_1  (cost=0.00..683635.38 rows=16660933 width=119) (actual time=0.064..4066.655 rows=16663785 loops=1)  
                           Filter: ((crt_time >= CURRENT_DATE) AND (crt_time < (CURRENT_DATE + 1)))  
 Planning Time: 219.559 ms  
 Execution Time: 53133.463 ms  
(13 rows)  
  
  
                                                                                                          QUERY PLAN                                                                                                             
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
 Subquery Scan on t  (cost=65.16..10962058.77 rows=83306 width=119) (actual time=0.041..104082.386 rows=10001 loops=1)  
   Filter: (t.rn = 1)  
   Rows Removed by Filter: 16653784  
   ->  WindowAgg  (cost=65.16..10753792.55 rows=16661298 width=127) (actual time=0.040..102532.935 rows=16663785 loops=1)  
         ->  Merge Append  (cost=65.16..10462219.83 rows=16661298 width=119) (actual time=0.029..92266.387 rows=16663785 loops=1)  
               Sort Key: tbl_sensor_log.sid, tbl_sensor_log.crt_time DESC  
               Subplans Removed: 365  
               ->  Index Scan using tbl_sensor_log_20210529_sid_crt_time_idx on tbl_sensor_log_20210529 tbl_sensor_log_1  (cost=0.44..9177871.39 rows=16660933 width=119) (actual time=0.029..89908.207 rows=16663785 loops=1)  
                     Index Cond: ((crt_time >= CURRENT_DATE) AND (crt_time < (CURRENT_DATE + 1)))  
 Planning Time: 39.511 ms  
 Execution Time: 104088.128 ms  
(11 rows)  
```

方法2: 使用索引链表跳跳糖  
递归, 每次扫描定位到1个目标SID, 然后跳到第二个SID, 而不是通过索引链表顺序扫描.   
链表顺序扫描的缺点: 整张索引的时间范围内的所有叶子结点的每个page都要扫描到, 性能烂到家.   

```  plsql
with recursive tmp as (  
(select t from tbl_sensor_log as t where crt_time >= current_date and crt_time < current_date+1 order by sid, crt_time desc limit 1)  
union all   
select (select tbl_sensor_log from tbl_sensor_log where sid>(tmp.t).sid   
        and crt_time >= current_date and crt_time < current_date+1 order by sid, crt_time desc limit 1) as t  
from tmp where tmp.* is not null  
)  
select (tmp.t).* from tmp   
where tmp.* is not null;  
  
  
  
                                                                                QUERY PLAN                                                                                  
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
 CTE Scan on tmp  (cost=6650.50..6652.52 rows=100 width=52)  
   Filter: (tmp.* IS NOT NULL)  
   CTE tmp  
     ->  Recursive Union  (cost=65.16..6650.50 rows=101 width=32)  
           ->  Subquery Scan on "*SELECT* 1"  (cost=65.16..65.80 rows=1 width=32)  
                 ->  Limit  (cost=65.16..65.79 rows=1 width=44)  
                       ->  Merge Append  (cost=65.16..10462219.83 rows=16661298 width=44)  
                             Sort Key: t.sid, t.crt_time DESC  
                             Subplans Removed: 365  
                             ->  Index Scan using tbl_sensor_log_20210529_sid_crt_time_idx on tbl_sensor_log_20210529 t_1  (cost=0.44..9177871.39 rows=16660933 width=44)  
                                   Index Cond: ((crt_time >= CURRENT_DATE) AND (crt_time < (CURRENT_DATE + 1)))  
           ->  WorkTable Scan on tmp tmp_1  (cost=0.00..658.27 rows=10 width=32)  
                 Filter: (tmp_1.* IS NOT NULL)  
(13 rows)  
  
  
  
                                                                                                              QUERY PLAN                                                                                                                
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
 CTE Scan on tmp  (cost=6650.50..6652.52 rows=100 width=52) (actual time=0.036..124.680 rows=10001 loops=1)  
   Filter: (tmp.* IS NOT NULL)  
   Rows Removed by Filter: 1  
   CTE tmp  
     ->  Recursive Union  (cost=65.16..6650.50 rows=101 width=32) (actual time=0.032..119.486 rows=10002 loops=1)  
           ->  Subquery Scan on "*SELECT* 1"  (cost=65.16..65.80 rows=1 width=32) (actual time=0.031..0.033 rows=1 loops=1)  
                 ->  Limit  (cost=65.16..65.79 rows=1 width=44) (actual time=0.031..0.032 rows=1 loops=1)  
                       ->  Merge Append  (cost=65.16..10462219.83 rows=16661298 width=44) (actual time=0.030..0.030 rows=1 loops=1)  
                             Sort Key: t.sid, t.crt_time DESC  
                             Subplans Removed: 365  
                             ->  Index Scan using tbl_sensor_log_20210529_sid_crt_time_idx on tbl_sensor_log_20210529 t_1  (cost=0.44..9177871.39 rows=16660933 width=44) (actual time=0.029..0.030 rows=1 loops=1)  
                                   Index Cond: ((crt_time >= CURRENT_DATE) AND (crt_time < (CURRENT_DATE + 1)))  
           ->  WorkTable Scan on tmp tmp_1  (cost=0.00..658.27 rows=10 width=32) (actual time=0.011..0.011 rows=1 loops=10002)  
                 Filter: (tmp_1.* IS NOT NULL)  
                 Rows Removed by Filter: 0  
                 SubPlan 1  
                   ->  Limit  (cost=65.16..65.81 rows=1 width=44) (actual time=0.011..0.011 rows=1 loops=10001)  
                         ->  Merge Append  (cost=65.16..3572009.02 rows=5554009 width=44) (actual time=0.011..0.011 rows=1 loops=10001)  
                               Sort Key: tbl_sensor_log.sid, tbl_sensor_log.crt_time DESC  
                               Subplans Removed: 365  
                               ->  Index Scan using tbl_sensor_log_20210529_sid_crt_time_idx on tbl_sensor_log_20210529 tbl_sensor_log_1  (cost=0.44..3115515.85 rows=5553644 width=44) (actual time=0.010..0.010 rows=1 loops=10001)  
                                     Index Cond: ((sid > (tmp_1.t).sid) AND (crt_time >= CURRENT_DATE) AND (crt_time < (CURRENT_DATE + 1)))  
 Planning Time: 69.016 ms  
 Execution Time: 125.866 ms  
(24 rows)  
```


方法3: 使用subquery  
但是, 需要维护一张SID表, 实际业务逻辑可能比这复杂, SID表可能没有这么好维护.   
而且还有1个问题: 今天没有活跃的SID也会被查出来. 如果选择过滤今天不活跃的记录, 需要多次评估, 性能就会下降.    


```  plsql
create table tbl_sensor (sid int primary key);  
insert into tbl_sensor select generate_series(0,10010);  
  
  
  
select (select tbl_sensor_log from tbl_sensor_log where sid=t.sid and crt_time >= current_date and crt_time < current_date+1 order by crt_time desc limit 1) as val  
from tbl_sensor as t;  
  
  
                                                                             QUERY PLAN                                                                               
--------------------------------------------------------------------------------------------------------------------------------------------------------------------  
 Index Only Scan using tbl_sensor_pkey on tbl_sensor t  (cost=0.29..509812.03 rows=10011 width=32)  
   SubPlan 1  
     ->  Limit  (cost=49.58..50.91 rows=1 width=40)  
           ->  Append  (cost=49.58..2751.07 rows=2036 width=40)  
                 Subplans Removed: 365  
                 ->  Index Scan using tbl_sensor_log_20210529_sid_crt_time_idx on tbl_sensor_log_20210529 tbl_sensor_log_1  (cost=0.44..1880.54 rows=1671 width=40)  
                       Index Cond: ((sid = t.sid) AND (crt_time >= CURRENT_DATE) AND (crt_time < (CURRENT_DATE + 1)))  
(7 rows)  
  
  
  
                                                                                                    QUERY PLAN                                                                                                      
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
 Index Only Scan using tbl_sensor_pkey on tbl_sensor t  (cost=0.29..509812.03 rows=10011 width=32) (actual time=0.039..116.315 rows=10011 loops=1)  
   Heap Fetches: 0  
   SubPlan 1  
     ->  Limit  (cost=49.58..50.91 rows=1 width=40) (actual time=0.011..0.011 rows=1 loops=10011)  
           ->  Append  (cost=49.58..2751.07 rows=2036 width=40) (actual time=0.011..0.011 rows=1 loops=10011)  
                 Subplans Removed: 365  
                 ->  Index Scan using tbl_sensor_log_20210529_sid_crt_time_idx on tbl_sensor_log_20210529 tbl_sensor_log_1  (cost=0.44..1880.54 rows=1671 width=40) (actual time=0.010..0.010 rows=1 loops=10011)  
                       Index Cond: ((sid = t.sid) AND (crt_time >= CURRENT_DATE) AND (crt_time < (CURRENT_DATE + 1)))  
 Planning Time: 40.798 ms  
 Execution Time: 117.059 ms  
(10 rows)  
```

过滤不活跃的SID将增加计算量, 性能有下降.   

```  plsql
select * from   
(  
select (select tbl_sensor_log from tbl_sensor_log where sid=t.sid and crt_time >= current_date and crt_time < current_date+1 order by crt_time desc limit 1) as val  
from tbl_sensor as t  
) t1  
where t1.val is not null;   
  
  
                                                                                                    QUERY PLAN                                                                                                      
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  
 Index Only Scan using tbl_sensor_pkey on tbl_sensor t  (cost=0.29..1016895.27 rows=9961 width=32) (actual time=0.044..173.412 rows=10001 loops=1)  
   Filter: ((SubPlan 2) IS NOT NULL)  
   Rows Removed by Filter: 10  
   Heap Fetches: 0  
   SubPlan 1  
     ->  Limit  (cost=49.58..50.91 rows=1 width=40) (actual time=0.007..0.007 rows=1 loops=10001)  
           ->  Append  (cost=49.58..2751.07 rows=2036 width=40) (actual time=0.007..0.007 rows=1 loops=10001)  
                 Subplans Removed: 365  
                 ->  Index Scan using tbl_sensor_log_20210529_sid_crt_time_idx on tbl_sensor_log_20210529 tbl_sensor_log_1  (cost=0.44..1880.54 rows=1671 width=40) (actual time=0.006..0.006 rows=1 loops=10001)  
                       Index Cond: ((sid = t.sid) AND (crt_time >= CURRENT_DATE) AND (crt_time < (CURRENT_DATE + 1)))  
   SubPlan 2  
     ->  Limit  (cost=49.58..50.91 rows=1 width=40) (actual time=0.010..0.010 rows=1 loops=10011)  
           ->  Append  (cost=49.58..2751.07 rows=2036 width=40) (actual time=0.010..0.010 rows=1 loops=10011)  
                 Subplans Removed: 365  
                 ->  Index Scan using tbl_sensor_log_20210529_sid_crt_time_idx on tbl_sensor_log_20210529 tbl_sensor_log_3  (cost=0.44..1880.54 rows=1671 width=40) (actual time=0.009..0.009 rows=1 loops=10011)  
                       Index Cond: ((sid = t.sid) AND (crt_time >= CURRENT_DATE) AND (crt_time < (CURRENT_DATE + 1)))  
 Planning Time: 68.114 ms  
 Execution Time: 174.239 ms  
(18 rows)  
```

## 更多CTE的应用场景  

##### 202103/20210320_02.md   [《PostgreSQL 14 preview - recovery 性能增强 - recovery_init_sync_method=syncfs - 解决表很多时, crash recovery 递归open所有file的性能问题 - 需Linux新内核支持》](../202103/20210320_02.md)    
##### 202102/20210201_03.md   [《PostgreSQL 14 preview - SQL标准增强, 递归(CTE)图式搜索增加广度优先、深度优先语法, 循环语法 - breadth- or depth-first search orders and detect cycles》](../202102/20210201_03.md)    
##### 202011/20201125_01.md   [《PostgreSQL 递归查询在分组合并中的用法》](../202011/20201125_01.md)    
##### 202006/20200615_01.md   [《递归+排序字段加权 skip scan 解决 窗口查询多列分组去重的性能问题》](../202006/20200615_01.md)    
##### 202005/20200515_01.md   [《PostgreSQL 排序去重limit查询优化 - 递归 vs group分组 (loop降到极限, block scan降到极限)》](../202005/20200515_01.md)    
##### 202003/20200329_01.md   [《PostgreSQL 家族图谱、社交图谱、树状关系、藤状分佣、溯源、等场景实践 - 递归,with recursive query (有向无环 , 有向有环)》](../202003/20200329_01.md)    
##### 202002/20200228_01.md   [《累加链条件过滤 - 递归、窗口、UDF、游标、模拟递归、scan 剪切》](../202002/20200228_01.md)    
##### 201903/20190318_04.md   [《PostgreSQL 并行计算解说 之29 - parallel 递归查询, 树状查询, 异构查询, CTE, recursive CTE, connect by》](../201903/20190318_04.md)    
##### 201808/20180808_02.md   [《PostgreSQL 递归应用实践 - 非“传销”的高并发实时藤、树状佣金分配体系》](../201808/20180808_02.md)    
##### 201804/20180406_01.md   [《PostgreSQL 递归妙用案例 - 分组数据去重与打散》](../201804/20180406_01.md)    
##### 201803/20180323_03.md   [《PostgreSQL Oracle 兼容性之 - INDEX SKIP SCAN (递归查询变态优化) 非驱动列索引扫描优化》](../201803/20180323_03.md)    
##### 201705/20170519_01.md   [《PostgrSQL 递归SQL的几个应用 - 极客与正常人的思维》](../201705/20170519_01.md)    
##### 201703/20170324_01.md   [《PostgreSQL 递归查询CASE - 树型路径分组输出》](../201703/20170324_01.md)    
##### 201612/20161201_01.md   [《用PostgreSQL找回618秒逝去的青春 - 递归收敛优化》](../201612/20161201_01.md)    
##### 201611/20161128_02.md   [《distinct xx和count(distinct xx)的变态递归优化方法 - 索引收敛(skip scan)扫描》](../201611/20161128_02.md)    
##### 201611/20161128_01.md   [《时序数据合并场景加速分析和实现 - 复合索引，窗口分组查询加速，变态递归加速》](../201611/20161128_01.md)    
##### 201608/20160815_04.md   [《PostgreSQL雕虫小技cte 递归查询，分组TOP性能提升44倍》](../201608/20160815_04.md)    
##### 201607/20160725_01.md   [《PostgreSQL 使用递归SQL 找出数据库对象之间的依赖关系 - 例如视图依赖》](../201607/20160725_01.md)    
##### 201607/20160723_01.md   [《PostgreSQL 递归死循环案例及解法》](../201607/20160723_01.md)    
##### 201604/20160405_01.md   [《PostgreSQL 递归查询一例 - 资金累加链》](../201604/20160405_01.md)    
##### 201512/20151221_02.md   [《PostgreSQL Oracle 兼容性之 - WITH 递归 ( connect by )》](../201512/20151221_02.md)    
##### 201210/20121009_01.md   [《递归优化CASE - group by & distinct tuning case : use WITH RECURSIVE and min() function》](../201210/20121009_01.md)    
##### 201209/20120914_01.md   [《递归优化CASE - performance tuning case :use cursor\trigger\recursive replace (group by and order by) REDUCE needed blockes scan》](../201209/20120914_01.md)    
##### 201105/20110527_01.md   [《PostgreSQL 树状数据存储与查询(非递归) - Use ltree extension deal tree-like data type》](../201105/20110527_01.md)    
