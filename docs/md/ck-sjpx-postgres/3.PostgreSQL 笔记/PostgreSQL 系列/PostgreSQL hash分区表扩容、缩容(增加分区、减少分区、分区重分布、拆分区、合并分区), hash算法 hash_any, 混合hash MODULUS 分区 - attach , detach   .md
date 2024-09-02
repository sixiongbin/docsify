## PostgreSQL hash分区表扩容、缩容(增加分区、减少分区、分区重分布、拆分区、合并分区), hash算法 hash_any, 混合hash MODULUS 分区 - attach , detach   
  
### 作者  
digoal  
  
### 日期  
2021-04-22  
  
### 标签  
PostgreSQL , partition , 分区表  
  
----  
  
## 背景  
1、hash 分区表如何扩容? 例如1024个分区, 扩到2048个分区  
  
2、hash 分区表如何缩容? 例如512个分区, 缩小到64个分区  
  
3、hash 分区表支持不同的分区模数吗? 例如一个分区表既有按2取模的子分区, 也有按4取模的自分区  
  
4、hash 分区表的hash算法是什么?  
  
## 例子  
1、创建2个分区表, 分别1024, 4个分区.  
  
```  
create table a (id int, info text) partition by hash (id);  
do language plpgsql $$  
declare  
begin  
  for i in 0..1023 loop  
    execute format('create table a%s partition of a for values with (MODULUS %s, REMAINDER %s)', i, 1024, i);  
  end loop;  
end;  
$$;  
```  
  
```  
create table b (id int, info text) partition by hash (id);  
do language plpgsql $$  
declare  
begin  
  for i in 0..3 loop  
    execute format('create table b%s partition of b for values with (MODULUS %s, REMAINDER %s)', i, 4, i);  
  end loop;  
end;  
$$;  
```  
  
2、插入数据到4个分区的分区表  
  
```  
postgres=# insert into b select generate_series(1,1000000), random()::text;  
INSERT 0 1000000  
postgres=# \dt+ b*  
                                 List of relations  
 Schema | Name |       Type        |  Owner   | Persistence |  Size   | Description   
--------+------+-------------------+----------+-------------+---------+-------------  
 public | b    | partitioned table | postgres | permanent   | 0 bytes |   
 public | b0   | table             | postgres | permanent   | 13 MB   |   
 public | b1   | table             | postgres | permanent   | 13 MB   |   
 public | b2   | table             | postgres | permanent   | 13 MB   |   
 public | b3   | table             | postgres | permanent   | 13 MB   |   
(5 rows)  
```  
  
3、查询b0子分区的数据, 你会发现ID的value和直接hash取模的结果没有直接关系.   
  
```  
postgres=# select * from b0 limit 10;  
 id |         info           
----+----------------------  
  1 | 0.5017019882606526  
 12 | 0.4488983802956099  
 14 | 0.036601106391117355  
 16 | 0.8031411032108977  
 17 | 0.2561659714681248  
 26 | 0.8746840737824613  
 28 | 0.9703716579314445  
 30 | 0.597006160271178  
 32 | 0.4628211184001785  
 34 | 0.9344181322067726  
(10 rows)  
```  
  
使用hashint4的到的hashvalue, 取余后结果和子分区的REMAINDER也没有关系, 那么hash分区的算法到底是什么?  
  
```  
postgres=#  select hashint4(id),* from b0 limit 10;  
  hashint4   | id |         info           
-------------+----+----------------------  
 -1905060026 |  1 | 0.5017019882606526  
  1938269380 | 12 | 0.4488983802956099  
 -1018458207 | 14 | 0.036601106391117355  
  -771555569 | 16 | 0.8031411032108977  
  -917053276 | 17 | 0.2561659714681248  
 -1668038584 | 26 | 0.8746840737824613  
   496699940 | 28 | 0.9703716579314445  
  1660582443 | 30 | 0.597006160271178  
  1125460419 | 32 | 0.4628211184001785  
   800709825 | 34 | 0.9344181322067726  
(10 rows)  
  
postgres=#  select mod(hashint4(id),4),* from b0 limit 10;  
 mod | id |         info           
-----+----+----------------------  
  -2 |  1 | 0.5017019882606526  
   0 | 12 | 0.4488983802956099  
  -3 | 14 | 0.036601106391117355  
  -1 | 16 | 0.8031411032108977  
   0 | 17 | 0.2561659714681248  
   0 | 26 | 0.8746840737824613  
   0 | 28 | 0.9703716579314445  
   3 | 30 | 0.597006160271178  
   3 | 32 | 0.4628211184001785  
   1 | 34 | 0.9344181322067726  
(10 rows)  
```  
  
hash分区使用hash_any(), 无对应SQL函数接口.  
  
https://github.com/postgres/postgres/blob/master/src/backend/access/hash/hashfunc.c  
  
src/include/common/hashfn.h  
  
hash_any  
  
```  
static inline Datum  
hash_any(const unsigned char *k, int keylen)  
{  
        return UInt32GetDatum(hash_bytes(k, keylen));  
}  
```  
  
4、将b0的数据写入a, 观察写入了哪些子分区.  
  
写入的子分区按4取模后=0, 符合预期, 这个用于拆分区、合分区就比较方便了.   
  
```  
postgres=# insert into a select * from b0;  
  
postgres=# select distinct mod(substring(x,'a(\d*)')::int, 4) from ( select distinct tableoid::regclass::text x from a order by 1) t;  
 mod   
-----  
   0  
(1 row)  
```  
  
扩容例子, 为了避免数据全部重新计算, 可以对齐 MODULUS , REMAINDER , 只要是倍数即可.   
例如 1024 个分区 合并 到 512个分区  
  
```  
REMAINDER 0,512 合并到 0  
REMAINDER 1,513 合并到 1  
...  
REMAINDER 511,1023 合并到 511   
```  
  
甚至一个分区表可以有不同的MODULUS子分区.     
  
```  
create table c (id int, info text) partition by hash (id);  
create table c2_0 partition of c for values with (MODULUS 2, REMAINDER 0);  
create table c4_1 partition of c for values with (MODULUS 4, REMAINDER 1);  
create table c4_3 partition of c for values with (MODULUS 4, REMAINDER 3);  
  
postgres=# \d+ c  
                               Partitioned table "public.c"  
 Column |  Type   | Collation | Nullable | Default | Storage  | Stats target | Description   
--------+---------+-----------+----------+---------+----------+--------------+-------------  
 id     | integer |           |          |         | plain    |              |   
 info   | text    |           |          |         | extended |              |   
Partition key: HASH (id)  
Partitions: c2_0 FOR VALUES WITH (modulus 2, remainder 0),  
            c4_1 FOR VALUES WITH (modulus 4, remainder 1),  
            c4_3 FOR VALUES WITH (modulus 4, remainder 3)  
```  
  
```  
postgres=# create table c4_0 partition of c for values with (MODULUS 4, REMAINDER 0);  
ERROR:  42P17: partition "c4_0" would overlap partition "c2_0"  
LOCATION:  check_new_partition_bound, partbounds.c:3077  
```  
  
https://www.postgresql.org/docs/current/sql-createtable.html  
  
拆分区、扩分区例子:  
  
```  
set lock_timeout='1s';
begin;  
alter table c detach partition c2_0;  
create table c4_0 partition of c for values with (MODULUS 4, REMAINDER 0);  
create table c4_2 partition of c for values with (MODULUS 4, REMAINDER 2);  
insert into c select * from c2_0;  
end;  
```  
  
合并分区、缩分区例子:  
  
```  
set lock_timeout='1s';
begin;  
alter table c detach partition c4_0;  
alter table c detach partition c4_2;  
create table c2_0 partition of c for values with (MODULUS 2, REMAINDER 0);  
insert into c select * from c4_0;  
insert into c select * from c4_2;  
end;  
```  
    
  
#### [PostgreSQL 许愿链接](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216")
您的愿望将传达给PG kernel hacker、数据库厂商等, 帮助提高数据库产品质量和功能, 说不定下一个PG版本就有您提出的功能点. 针对非常好的提议，奖励限量版PG文化衫、纪念品、贴纸、PG热门书籍等，奖品丰富，快来许愿。[开不开森](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216").  
  
  
#### [9.9元购买3个月阿里云RDS PostgreSQL实例](https://www.aliyun.com/database/postgresqlactivity "57258f76c37864c6e6d23383d05714ea")
  
  
#### [PostgreSQL 解决方案集合](https://yq.aliyun.com/topic/118 "40cff096e9ed7122c512b35d8561d9c8")
  
  
#### [德哥 / digoal's github - 公益是一辈子的事.](https://github.com/digoal/blog/blob/master/README.md "22709685feb7cab07d30f30387f0a9ae")
  
  
![digoal's wechat](../pic/digoal_weixin.jpg "f7ad92eeba24523fd47a6e1a0e691b59")
  
