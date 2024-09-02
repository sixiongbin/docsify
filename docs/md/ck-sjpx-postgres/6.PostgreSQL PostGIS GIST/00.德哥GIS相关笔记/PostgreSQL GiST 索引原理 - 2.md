## PostgreSQL GiST 索引原理 - 2           

### 作者            
digoal            
            
### 日期            
2020-10-04            
            
### 标签            
PostgreSQL , GiST , 索引原理             
            
----

## 背景           
interval类型的GiST索引, 这个索引支持的操作符.    
    
excluding约束, 会议室预定中的应用.    
    
GiST+Btree (sclar)类型的联合索引.      
    
## R-tree for intervals    
interval类型, 如tsrange, int4range等存储时间、数值范围的类型.    
    
可以采用GiST索引, 按范围划分node, 在index page中存储不同范围区间的node, 形成高度平衡树.    
    
Another example of using GiST access method is indexing of intervals, for example, time intervals ("tsrange" type). All the difference is that internal nodes will contain bounding intervals instead of bounding rectangles.    
    
Let's consider a simple example. We will be renting out a cottage and storing reservation intervals in a table:    
    
```    
postgres=# create table reservations(during tsrange);    
    
postgres=# insert into reservations(during) values    
('[2016-12-30, 2017-01-09)'),    
('[2017-02-23, 2017-02-27)'),    
('[2017-04-29, 2017-05-02)');    
    
postgres=# create index on reservations using gist(during);    
```

The index can be used to speed up the following query, for example:    
    
```    
postgres=# select * from reservations where during && '[2017-01-01, 2017-04-01)';    
    
                    during                        
-----------------------------------------------    
 ["2016-12-30 00:00:00","2017-01-08 00:00:00")    
 ["2017-02-23 00:00:00","2017-02-26 00:00:00")    
(2 rows)    
    
postgres=# explain (costs off) select * from reservations where during && '[2017-01-01, 2017-04-01)';    
    
                                     QUERY PLAN                                        
------------------------------------------------------------------------------------    
 Index Only Scan using reservations_during_idx on reservations    
   Index Cond: (during && '["2017-01-01 00:00:00","2017-04-01 00:00:00")'::tsrange)    
(2 rows)    
```

```&&``` operator for intervals denotes intersection; therefore, the query must return all intervals intersecting with the given one. For such an operator, the consistency function determines whether the given interval intersects with a value in an internal or leaf row.    
    
Note that this is either not about getting intervals in a certain order, although comparison operators are defined for intervals. We can use "btree" index for intervals, but in this case, we will have to do without support of operations like these:    
    
```    
postgres=# select amop.amopopr::regoperator, amop.amoppurpose, amop.amopstrategy    
from pg_opclass opc, pg_opfamily opf, pg_am am, pg_amop amop    
where opc.opcname = 'range_ops'    
and opf.oid = opc.opcfamily    
and am.oid = opf.opfmethod    
and amop.amopfamily = opc.opcfamily    
and am.amname = 'gist'    
and amop.amoplefttype = opc.opcintype;    
    
         amopopr         | amoppurpose | amopstrategy    
-------------------------+-------------+--------------     
 @>(anyrange,anyelement) | s           |           16  contains element    
 <<(anyrange,anyrange)   | s           |            1  strictly left    
 &<(anyrange,anyrange)   | s           |            2  not beyond right boundary    
 &&(anyrange,anyrange)   | s           |            3  intersects    
 &>(anyrange,anyrange)   | s           |            4  not beyond left boundary    
 >>(anyrange,anyrange)   | s           |            5  strictly right    
 -|-(anyrange,anyrange)  | s           |            6  adjacent    
 @>(anyrange,anyrange)   | s           |            7  contains interval    
 <@(anyrange,anyrange)   | s           |            8  contained in interval    
 =(anyrange,anyrange)    | s           |           18  equals    
(10 rows)    
```

(Except the equality, which is contained in the operator class for "btree" access method.)    
    
### Internals    
We can look inside using the same "gevel" extension. We only need to remember to change the data type in the call to gist_print:    
    
```    
postgres=# select level, a from gist_print('reservations_during_idx')    
as t(level int, valid bool, a tsrange);    
    
 level |                       a                          
-------+-----------------------------------------------    
     1 | ["2016-12-30 00:00:00","2017-01-09 00:00:00")    
     1 | ["2017-02-23 00:00:00","2017-02-27 00:00:00")    
     1 | ["2017-04-29 00:00:00","2017-05-02 00:00:00")    
(3 rows)    
```

## Exclusion constraint    
GiST index can be used to support exclusion constraints (EXCLUDE).    
    
The exclusion constraint ensures given fields of any two table rows not to "correspond" to each other in terms of some operators. If "equals" operator is chosen, we exactly get the unique constraint: given fields of any two rows do not equal each other.    
    
The exclusion constraint is supported by the index, as well as the unique constraint. We can choose any operator so that:    
- It is supported by the index method — "can_exclude" property (for example, "btree", GiST, or SP-GiST, but not GIN).    
- It is commutative, that is, the condition is met: a operator b = b operator a.    
  

This is a list of suitable strategies and examples of operators (operators, as we remember, can have different names and be available not for all data types):    
- For "btree":    
    - "equals" ```=```    
- For GiST and SP-GiST:    
    - "intersection" ```&&```    
    - "coincidence" ```~=```    
    - "adjacency" ```-|-```    
    

Note that we can use the equality operator in an exclusion constraint, but it is impractical: a unique constraint will be more efficient. That is exactly why we did not touch upon exclusion constraints when we discussed B-trees.    
    
Let's provide an example of using an exclusion constraint. It is reasonable not to permit reservations for intersecting intervals.    
    
```    
postgres=# alter table reservations add exclude using gist(during with &&);    
```

Once we created the exclusion constraint, we can add rows:    
    
```    
postgres=# insert into reservations(during) values ('[2017-06-10, 2017-06-13)');    
```

But an attempt to insert an intersecting interval into the table will result in an error:    
    
```    
postgres=# insert into reservations(during) values ('[2017-05-15, 2017-06-15)');    
    
ERROR: conflicting key value violates exclusion constraint "reservations_during_excl"    
DETAIL: Key (during)=(["2017-05-15 00:00:00","2017-06-15 00:00:00")) conflicts with existing key (during)=(["2017-06-10 00:00:00","2017-06-13 00:00:00")).    
```

## "btree_gist" extension    
例子:     
[《会议室预定系统实践(解放开发) - PostgreSQL tsrange(时间范围类型) + 排他约束》](../201712/20171223_02.md)      
    
    
Let's complicate the problem. We expand our humble business, and we are going to rent out several cottages:    
    
```    
postgres=# alter table reservations add house_no integer default 1;    
```

We need to change the exclusion constraint so that house numbers are taken into account. GiST, however, does not support the equality operation for integer numbers:    
    
```    
postgres=# alter table reservations drop constraint reservations_during_excl;    
    
postgres=# alter table reservations add exclude using gist(during with &&, house_no with =);    
    
ERROR: data type integer has no default operator class for access method "gist"    
HINT: You must specify an operator class for the index or define a default operator class for the data type.    
```

In this case, "btree_gist" extension will help, which adds GiST support of operations inherent to B-trees. GiST, eventually, can support any operators, so why should not we teach it to support "greater", "less", and "equal" operators?    
    
    
    
```    
postgres=# create extension btree_gist;    
    
postgres=# alter table reservations add exclude using gist(during with &&, house_no with =);    
```

Now we still cannot reserve the first cottage for the same dates:    
    
```    
postgres=# insert into reservations(during, house_no) values ('[2017-05-15, 2017-06-15)', 1);    
    
ERROR: conflicting key value violates exclusion constraint "reservations_during_house_no_excl"    
```


However, we can reserve the second one:    
    
```    
postgres=# insert into reservations(during, house_no) values ('[2017-05-15, 2017-06-15)', 2);    
```

But be aware that although GiST can somehow support "greater", "less", and "equal" operators, B-tree still does this better. So it is worth using this technique only if GiST index is essentially needed, like in our example.    
    
## 参考    

[《会议室预定系统实践(解放开发) - PostgreSQL tsrange(时间范围类型) + 排他约束》](../201712/20171223_02.md)      
    
https://postgrespro.com/blog/pgsql/4175817    
    
https://www.postgresql.org/docs/13/btree-gist.html    

