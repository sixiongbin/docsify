## 重新发现PostgreSQL之美 - 9 面向多值列的倒排索引GIN|RUM    
          
### 作者          
digoal          
          
### 日期          
2021-05-31          
          
### 标签          
PostgreSQL , 倒排 , rum , gin , posting tree , rank         
          
----          
          
## 背景   
视频回放: https://www.bilibili.com/video/BV1CA411g7oK/       
   
场景:    
通用业务, 分词查询诉求.    
    
挑战:    
传统数据库没有分词、实时全文检索索引功能, 需要将数据同步到搜索引擎, 这种解决方案的弊端:     
- 研发成本增加、    
- 软硬件成本增加、    
- 系统问题增多(同步延迟问题、同步异常问题、同步一致性问题)、    
- 开发灵活性下降(无法同时过滤分词条件与表的其他条件, 需要业务层交换数据)    
- 同时过滤分词条件与表的其他条件后, 无法有效的按RANK排序分词相似性    
    
PG 解决方案:    
- 1、倒排索引GIN:     
    - 支持多值类型的按元素检索: tsvector, array, json, xml, hstore, 任意字段组合搜索       
    - 一对多的数据模型      
- 2、增强倒排索引RUM, RANK 加速方案:     
    - RUM索引在posting list里面, 每个行号后面附加addon内容(文本向量的对应位置信息), 同时支持自定义addon信息.     
    - addon的内容优势: 不需要回表搜索tuple内容. 降低IO, 提高性能.     
    
## 文档    
https://github.com/postgrespro/rum/blob/master/rum--1.3.sql    
    
https://www.postgresql.org/docs/14/textsearch-controls.html#TEXTSEARCH-RANKING    
    
    
如何计算rank :     
- 0 (the default) ignores the document length    
- 1 divides the rank by 1 + the logarithm of the document length    
- 2 divides the rank by the document length    
- 4 divides the rank by the mean harmonic distance between extents (this is implemented only by ts_rank_cd)    
- 8 divides the rank by the number of unique words in document    
- 16 divides the rank by 1 + the logarithm of the number of unique words in document    
- 32 divides the rank by itself + 1    
    
      
## 例子 GIN    
分词搜索, 同时按rank排序返回    
    
    
```    
SELECT title, ts_rank_cd(textsearch, query) AS rank    
FROM apod, to_tsquery('neutrino|(dark & matter)') query    
WHERE query @@ textsearch    
ORDER BY rank DESC    
LIMIT 10;    
    
    
                     title                     |   rank    
-----------------------------------------------+----------    
 Neutrinos in the Sun                          |      3.1    
 The Sudbury Neutrino Detector                 |      2.4    
 A MACHO View of Galactic Dark Matter          |  2.01317    
 Hot Gas and Dark Matter                       |  1.91171    
 The Virgo Cluster: Hot Plasma and Dark Matter |  1.90953    
 Rafting for Solar Neutrinos                   |      1.9    
 NGC 4650A: Strange Galaxy and Dark Matter     |  1.85774    
 Hot Gas and Dark Matter                       |   1.6123    
 Ice Fishing for Cosmic Neutrinos              |      1.6    
 Weak Lensing Distorts the Universe            | 0.818218    
```    
    
    
GIN索引: 分词检索在索引内完成, 但是rank排序需要回表, 将所有tuple找到后计算rank, 然后排序    
    
GIN 数据和索引结构演示:     
    
数据:     
```    
行号1: hello world    
行号2: digoal hello    
行号3: apple alibaba     
行号4: aliyun apple    
行号5: hello aliyun    
```    
    
GIN 倒排索引结构:     
token tree:     
```    
hello,world,digoal,apple,aliyun,alibaba     
```    
    
posting list|tree:     
```    
hello: 行号1, 行号2, 行号5    
world: 行号1    
digoal: 行号2    
apple: 行号3, 行号4    
aliyun: 行号4, 行号5    
alibaba: 行号3     
```    
    
## 例子 RUM    
    
如何解决RANK需要回表的性能问题:     
    
数据和索引结构演示:     
    
数据:     
```    
行号1: hello world    
行号2: digoal hello    
行号3: apple alibaba     
行号4: aliyun apple    
行号5: hello aliyun    
```    
    
RUM 倒排索引结构:     
token tree:     
```    
hello,world,digoal,apple,aliyun,alibaba     
```    
    
posting list|tree:     
```    
hello: 行号1 hello出现的位置, 行号2 hello出现的位置, 行号5 hello出现的位置    
world: 行号1 world出现的位置    
digoal: 行号2 digoal出现的位置    
apple: 行号3 apple出现的位置, 行号4 apple出现的位置    
aliyun: 行号4 aliyun出现的位置, 行号5 aliyun出现的位置    
alibaba: 行号3 alibaba出现的位置    
```    
    
## rum和gin对比例子     
```    
SELECT to_tsvector('english', 'a fat  cat sat on a mat - it ate a fat rats');    
                  to_tsvector    
-----------------------------------------------------    
 'ate':9 'cat':3 'fat':2,11 'mat':7 'rat':12 'sat':4    
```    
    
生成随机文本的例子      
    
```    
postgres=# select to_tsvector( string_agg(chr((array[32,97,98,99,100,101])[ceil(random()*6)]),'')) from generate_series(1,100);    
                                                                    to_tsvector                                                                         
----------------------------------------------------------------------------------------------------------------------------------------------------    
 'abbcb':9 'bdc':4 'beaeeddddbdbdb':11 'beeaba':12 'dbdbb':5 'dccbdbbebd':1 'dcccbbbecac':3 'dccec':10 'decaabc':6 'ebacaececd':2 'ecbcb':7 'edd':8    
(1 row)    
    
    
create or replace function fu() returns tsvector as $$    
   select to_tsvector( string_agg(chr((array[32,97,98,99,100,101])[ceil(random()*6)]),'')) from generate_series(1,100);    
 $$ language sql strict volatile;    
CREATE FUNCTION    
```    
    
写入111万随机文本数据    
    
```    
postgres=# create unlogged table tsv (id serial8 primary key, tsv tsvector);    
CREATE TABLE    
    
    
    
postgres=# insert into tsv (tsv) select fu() from generate_series(1,10000);    
INSERT 0 10000    
Time: 382.318 ms    
postgres=# insert into tsv (tsv) select fu() from generate_series(1,100000);    
INSERT 0 100000    
Time: 3804.033 ms (00:03.804)    
postgres=# insert into tsv (tsv) select fu() from generate_series(1,1000000);    
INSERT 0 1000000    
Time: 37887.880 ms (00:37.888)    
    
postgres=# select * from tsv limit 10;    
 id |                                                                                      tsv                                                                                           
----+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------    
  1 | 'adb':1 'bbebebdcd':5 'c':4 'ca':6 'cabddbecbcdabcdcbabbbdb':9 'cabeaaea':8 'cbbadaebdebedabdd':10 'd':2,12 'dac':11 'ddbaadebeb':7 'eceaabddb':3    
  2 | 'acdaaaa':10 'ad':6 'adcdcebeaeecc':4 'bcbdac':8 'bed':12 'bedaacabdbeceecaa':2 'cbabcddcda':7 'cecdec':1 'd':11 'dbbbcbacbd':9 'edddebcacea':5    
  3 | 'acececccbabadbdeeec':11 'aeadacbecad':8 'bdbadbadceedcadaeb':9 'bddab':4 'cebedbedb':3 'd':1,7 'da':2 'dda':12 'ddaacb':13 'ec':10 'eeadbeca':6    
  4 | 'aadccceb':18 'ae':5 'bcbbceda':3 'bdcd':13 'cabeac':12 'cb':2 'cbadccb':15 'cbeaba':8 'cbeb':6 'ccbbdbcaa':14 'cecbaeecb':7 'db':16 'dead':4 'e':1,11    
  5 | 'ab':2 'bc':12 'cadbaebeeabeec':7 'cce':6 'dab':9 'dadbccabbd':4 'dadddb':3 'db':1 'eb':10 'ecaadac':11 'eddbebcbabdccacbccde':8 'edddbcbde':5    
  6 | 'ae':11 'ba':9 'bb':12 'bcdd':7 'bd':15 'bdec':4 'beada':10 'caa':1 'caacc':14 'cabcbedadadceadbdbecec':8 'ddcdbaeeecad':13 'de':2 'dea':6 'deeaabba':3 'eeaad':5    
  7 | 'abaddcb':13 'acbedbbdceadcdda':10 'bcece':11 'cacbedc':3 'ccdb':2 'cd':7 'db':6 'dbebdd':12 'deb':8 'eaecba':4 'ecabeb':1 'ededdbbdacad':9 'eecd':5    
  8 | 'aaccccc':5 'aada':13 'abbddcbbcb':3 'b':7 'bae':1 'bb':4 'bcabceeabc':16 'bdbac':2 'bddaceeb':11 'cdcdceec':15 'cddea':9 'cecaa':8 'dbee':10 'e':14 'ee':6 'eedeba':12    
  9 | 'abcbcbd':1 'abdaa':12 'adaecaa':7 'caaabac':5 'caeeeebdc':6 'ccdd':9 'ddeab':10 'eaeebbbcadcbaac':4 'ebcddadeacb':11 'ebeeb':3 'ecbbcecaa':8 'ee':2    
 10 | 'aabadcab':14 'abaceeebdbeaaadbedccc':15 'b':16 'bcccdd':13 'bceedb':11 'c':2 'cc':12 'ccc':9 'cedac':4 'dadbba':7 'daee':8 'dd':10 'dddde':17 'e':6 'ecbea':5 'ed':1 'eeec':3    
(10 rows)    
```    
    
搜索用例    
    
```    
postgres=# explain (analyze,verbose,timing,costs,buffers)     
select *, ts_rank(tsv , to_tsquery('abc & c')) from tsv     
where tsv @@ to_tsquery('abc & c')     
order by ts_rank(tsv, to_tsquery('abc & c')) desc     
limit 10;    
```    
    
1、GIN 索引, 需要回表计算rank:      
    
```    
                                                                   QUERY PLAN                                                                       
------------------------------------------------------------------------------------------------------------------------------------------------    
 Limit  (cost=9763.40..9763.42 rows=10 width=215) (actual time=39.030..39.033 rows=10 loops=1)    
   Output: id, tsv, (ts_rank(tsv, to_tsquery('abc & c'::text)))    
   Buffers: shared hit=7336    
   ->  Sort  (cost=9763.40..9782.45 rows=7622 width=215) (actual time=39.029..39.031 rows=10 loops=1)    
         Output: id, tsv, (ts_rank(tsv, to_tsquery('abc & c'::text)))    
         Sort Key: (ts_rank(tsv.tsv, to_tsquery('abc & c'::text))) DESC    
         Sort Method: top-N heapsort  Memory: 30kB    
         Buffers: shared hit=7336    
         ->  Index Scan using idx_tsv_1 on public.tsv  (cost=11.25..9598.69 rows=7622 width=215) (actual time=20.434..37.754 rows=7531 loops=1)    
               Output: id, tsv, ts_rank(tsv, to_tsquery('abc & c'::text))    
               Index Cond: (tsv.tsv @@ to_tsquery('abc & c'::text))    
               Buffers: shared hit=7336    
 Planning:    
   Buffers: shared hit=2    
 Planning Time: 0.199 ms    
 Execution Time: 39.427 ms    
(16 rows)    
    
Time: 40.055 ms    
    
    
    
    
    
postgres=#  select *, ts_rank_cd(tsv , to_tsquery('abc & c'),32) from tsv where tsv @@ to_tsquery('abc & c') order by ts_rank_cd(tsv, to_tsquery('abc & c'),32) desc limit 10;     
   id   |                                                                                        tsv                                                                                        | ts_rank_cd     
--------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------    
 386419 | 'abc':9,12 'abceadba':3 'b':8,16 'badeebdebbb':4 'bdd':20 'bea':2 'c':10,14 'cad':7 'cbedbbdeb':5 'ccdadcda':6 'd':1,15 'ddeceb':13 'deabdd':11 'e':18                            | 0.16666667    
 568798 | 'abc':12 'addb':14 'aecdcecdbcebdbceeebaebca':4 'c':11,13 'cceddac':10 'd':1,3 'dab':6 'dcaddeeb':7 'de':8 'ea':5 'eadbadeacb':15 'edbb':2 'eecddada':9                           | 0.16666667    
 246969 | 'aa':3 'aabbdaaea':15 'abc':6 'adaecaceec':8 'aea':9 'aebb':20 'beed':14 'c':4,5,7,17 'dbaa':18 'dbe':12 'dcdda':13 'dd':16 'ebadeec':10 'ebcce':1 'ebdbedc':19                   | 0.16666667    
 276756 | 'aba':13 'abc':8 'aeecdceedaacdec':16 'b':5,6 'babba':12 'beadbcdeeeecbbbde':15 'c':7,9 'cabaacadb':11 'cb':10 'dcaebdbabddd':1 'dedaadd':14 'e':2 'ebdb':4                       | 0.16666667    
 534425 | 'aa':7 'abc':19 'b':2 'baebdacbc':13 'c':9,18,20 'cabcdaaada':5 'cc':17 'cd':4 'ce':21 'd':12 'dcadccbcddc':15 'dcbcacec':11 'dea':1 'e':14 'ea':6 'eceeeadc':16 'edd':10 'ede':3 | 0.16666667    
 545684 | 'abc':11 'c':10,12 'caebdacdeabbccadec':3 'cbecc':2 'ccace':15 'cdb':13 'cddaccbacbcb':8 'cecadbbecda':5 'ddaab':6 'e':4,14 'eaeccedbdbc':1 'ebdeb':7                             | 0.16666667    
 137518 | 'abc':13 'accebbedeabbdaa':2 'c':4,9,12,14 'cba':7 'cc':16 'ce':10 'dbbeebbdbd':18 'dce':8 'ddaeab':17 'eabadc':3 'eac':1 'eacc':11 'ecabdcddbecaebb':15 'ed':5                   | 0.16666667    
 227131 | 'abc':2 'abeaacaeceaed':7 'b':12 'baae':13 'bddccbabcbbcaa':9 'c':1,3,11 'caeabd':8 'cbadea':5 'ccaacdeeabccabc':10 'dcac':6 'eadaebbcccac':4 'ed':14                             | 0.16666667    
 264722 | 'abc':4 'acaabbeacacdbcaec':10 'ad':1 'ada':2 'c':3,5,6 'cbbcbcadee':13 'd':14 'dbdc':8 'decaceaaa':7 'deec':11 'ebcdaecea':12 'ecc':16 'ecdbebbbca':9                            | 0.16666667    
 634783 | 'aaaba':11 'aba':10 'abbc':5 'abc':14 'ad':2 'adca':16 'bdbbb':6 'c':9,13,15 'cbacccadacbbdeda':1 'cbb':17 'd':4 'dcbd':3 'debdadd':12 'dedadeaeecacaceceebdd':8 'ea':7           | 0.16666667    
(10 rows)    
    
Time: 63.782 ms    
```    
    
    
2、RUM 索引, 不需要回表计算rank:      
    
    
```    
postgres=# explain (analyze,verbose,timing,costs,buffers) select *,tsv <=> to_tsquery('abc & c') from tsv where tsv @@ to_tsquery('abc & c') order by tsv <=> to_tsquery('abc & c') limit 10;    
                                                               QUERY PLAN                                                                   
----------------------------------------------------------------------------------------------------------------------------------------    
 Limit  (cost=11.25..23.91 rows=10 width=215) (actual time=35.455..35.547 rows=10 loops=1)    
   Output: id, tsv, ((tsv <=> to_tsquery('abc & c'::text)))    
   Buffers: shared hit=1086    
   ->  Index Scan using idx_tsv_1 on public.tsv  (cost=11.25..8899.62 rows=7021 width=215) (actual time=35.453..35.526 rows=10 loops=1)    
         Output: id, tsv, (tsv <=> to_tsquery('abc & c'::text))    
         Index Cond: (tsv.tsv @@ to_tsquery('abc & c'::text))    
         Order By: (tsv.tsv <=> to_tsquery('abc & c'::text))    
         Buffers: shared hit=1086    
 Planning:    
   Buffers: shared hit=1    
 Planning Time: 0.187 ms    
 Execution Time: 36.395 ms    
(12 rows)    
    
Time: 37.025 ms    
    
    
    
    
    
postgres=#  select *,tsv <=> to_tsquery('abc & c') from tsv where tsv @@ to_tsquery('abc & c') order by tsv <=> to_tsquery('abc & c') limit 10;    
   id    |                                                                                        tsv                                                                                        | ?column?     
---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------    
  534425 | 'aa':7 'abc':19 'b':2 'baebdacbc':13 'c':9,18,20 'cabcdaaada':5 'cc':17 'cd':4 'ce':21 'd':12 'dcadccbcddc':15 'dcbcacec':11 'dea':1 'e':14 'ea':6 'eceeeadc':16 'edd':10 'ede':3 | 4.112335    
 1082907 | 'abc':9 'aeac':12 'bac':5 'baddaecd':6 'bc':7 'c':4,8,10 'd':3 'dadebcd':2 'dbebecabcaaeedaacdaaceebbbadeeeaad':11 'eaab':1 'ebaaebdaaaedc':13 'ed':14                            | 4.112335    
  264722 | 'abc':4 'acaabbeacacdbcaec':10 'ad':1 'ada':2 'c':3,5,6 'cbbcbcadee':13 'd':14 'dbdc':8 'decaceaaa':7 'deec':11 'ebcdaecea':12 'ecc':16 'ecdbebbbca':9                            | 4.112335    
  568798 | 'abc':12 'addb':14 'aecdcecdbcebdbceeebaebca':4 'c':11,13 'cceddac':10 'd':1,3 'dab':6 'dcaddeeb':7 'de':8 'ea':5 'eadbadeacb':15 'edbb':2 'eecddada':9                           | 4.112335    
  545684 | 'abc':11 'c':10,12 'caebdacdeabbccadec':3 'cbecc':2 'ccace':15 'cdb':13 'cddaccbacbcb':8 'cecadbbecda':5 'ddaab':6 'e':4,14 'eaeccedbdbc':1 'ebdeb':7                             | 4.112335    
  634783 | 'aaaba':11 'aba':10 'abbc':5 'abc':14 'ad':2 'adca':16 'bdbbb':6 'c':9,13,15 'cbacccadacbbdeda':1 'cbb':17 'd':4 'dcbd':3 'debdadd':12 'dedadeaeecacaceceebdd':8 'ea':7           | 4.112335    
  743324 | 'abc':14 'aeaddecbbd':12 'c':13,15,21 'ca':19 'ccca':1 'cdbbebdb':17 'cdc':2 'cdeeea':11 'd':4 'dadb':7 'db':8 'dbbcb':10 'dcdbeab':5 'ece':3 'ed':18 'eedcddddb':9               | 4.112335    
  905287 | 'abc':15 'ad':12 'b':10 'babebaca':17 'bbeebbedaacc':18 'bccdbc':1 'bcebb':9 'c':14,16 'cebaaaea':2 'd':13 'dc':7,11 'dca':4 'ddaebdddbc':8 'ddddebdbd':5 'e':6 'ebddb':3         | 4.112335    
  826801 | 'abc':4 'accbbe':11 'aebeedcebdcb':7 'b':1,9 'ba':13 'bbae':14 'bc':16 'bcecdeceeebad':17 'c':3,5 'cbcbbdeb':6 'd':15 'dcadbebce':12 'ebeceaabad':8 'ed':10 'ee':2                | 4.112335    
  246969 | 'aa':3 'aabbdaaea':15 'abc':6 'adaecaceec':8 'aea':9 'aebb':20 'beed':14 'c':4,5,7,17 'dbaa':18 'dbe':12 'dcdda':13 'dd':16 'ebadeec':10 'ebcce':1 'ebdbedc':19                   | 4.112335    
(10 rows)    
    
Time: 38.362 ms    
```    
    
    
    
    
    
    
## 例子2: RUM addon 其他字段信息到posting tree     
    
### rum_TYPE_ops    
    
For types: int2, int4, int8, float4, float8, money, oid, time, timetz, date,    
interval, macaddr, inet, cidr, text, varchar, char, bytea, bit, varbit,    
numeric, timestamp, timestamptz    
    
Supported operations: `<`, `<=`, `=`, `>=`, `>` for all types and    
`<=>`, `<=|` and `|=>` for int2, int4, int8, float4, float8, money, oid,    
timestamp and timestamptz types.    
    
Supports ordering by `<=>`, `<=|` and `|=>` operators. Can be used with    
`rum_tsvector_addon_ops`, `rum_tsvector_hash_addon_ops' and `rum_anyarray_addon_ops` operator classes.    
    
### rum_tsvector_addon_ops    
    
For type: `tsvector`    
    
This operator class stores `tsvector` lexems with any supported by module    
field. There is the example.    
    
Let us assume we have the table:    
```sql    
CREATE TABLE tsts (id int, t tsvector, d timestamp);    
    
\copy tsts from 'rum/data/tsts.data'    
    
CREATE INDEX tsts_idx ON tsts USING rum (t rum_tsvector_addon_ops, d)    
    WITH (attach = 'd', to = 't');    
```    
    
Now we can execute the following queries:    
```sql    
EXPLAIN (costs off)    
    SELECT id, d, d <=> '2016-05-16 14:21:25' FROM tsts WHERE t @@ 'wr&qh' ORDER BY d <=> '2016-05-16 14:21:25' LIMIT 5;    
                                    QUERY PLAN    
-----------------------------------------------------------------------------------    
 Limit    
   ->  Index Scan using tsts_idx on tsts    
         Index Cond: (t @@ '''wr'' & ''qh'''::tsquery)    
         Order By: (d <=> 'Mon May 16 14:21:25 2016'::timestamp without time zone)    
(4 rows)    
    
SELECT id, d, d <=> '2016-05-16 14:21:25' FROM tsts WHERE t @@ 'wr&qh' ORDER BY d <=> '2016-05-16 14:21:25' LIMIT 5;    
 id  |                d                |   ?column?    
-----+---------------------------------+---------------    
 355 | Mon May 16 14:21:22.326724 2016 |      2.673276    
 354 | Mon May 16 13:21:22.326724 2016 |   3602.673276    
 371 | Tue May 17 06:21:22.326724 2016 |  57597.326724    
 406 | Wed May 18 17:21:22.326724 2016 | 183597.326724    
 415 | Thu May 19 02:21:22.326724 2016 | 215997.326724    
(5 rows)    
```    
    
> **Warning:** Currently RUM has bogus behaviour when one creates an index using ordering over pass-by-reference additional information. This is due to the fact that posting trees have fixed length right bound and fixed length non-leaf posting items. It isn't allowed to create such indexes.    
    
### rum_tsvector_hash_addon_ops    
    
For type: `tsvector`    
    
This operator class stores hash of `tsvector` lexems with any supported by module    
field.    
    
**Doesn't** support prefix search.    
    
    
    
### rum_anyarray_addon_ops    
    
For type: `anyarray`    
    
This operator class stores `anyarrray` elements with any supported by module field.    
    
    
    
    
    
    
### rum_anyarray_addon_ops 的例子    
    
```    
postgres=# create extension rum;    
CREATE EXTENSION    
    
create table tbl (    
id serial8 primary key,    
a int[],    
n int,    
crt_time timestamp    
);    
    
    
create index idx_tbl_1 on tbl using rum (a rum_anyarray_addon_ops, n) with (attach='n', to='a');    
    
    
postgres=# explain (analyze,verbose,timing,costs,buffers) select * from tbl where a @> array[1,2,3] and n <= 1;     
    
                                                          QUERY PLAN                                                               
-------------------------------------------------------------------------------------------------------------------------------    
 Index Scan using idx_tbl_1 on public.tbl  (cost=15.40..38.90 rows=20 width=53) (actual time=130.694..130.711 rows=20 loops=1)    
   Output: id, a, n, crt_time    
   Index Cond: ((tbl.a @> '{1,2,3}'::integer[]) AND (tbl.n <= 1))    
   Buffers: shared hit=1768    
 Planning:    
   Buffers: shared hit=1    
 Planning Time: 0.092 ms    
 Execution Time: 130.735 ms    
(8 rows)    
```    
    
    
    
    
    
  
#### [PostgreSQL 许愿链接](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216")
您的愿望将传达给PG kernel hacker、数据库厂商等, 帮助提高数据库产品质量和功能, 说不定下一个PG版本就有您提出的功能点. 针对非常好的提议，奖励限量版PG文化衫、纪念品、贴纸、PG热门书籍等，奖品丰富，快来许愿。[开不开森](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216").  
  
  
#### [9.9元购买3个月阿里云RDS PostgreSQL实例](https://www.aliyun.com/database/postgresqlactivity "57258f76c37864c6e6d23383d05714ea")
  
  
#### [PostgreSQL 解决方案集合](https://yq.aliyun.com/topic/118 "40cff096e9ed7122c512b35d8561d9c8")
  
  
#### [德哥 / digoal's github - 公益是一辈子的事.](https://github.com/digoal/blog/blob/master/README.md "22709685feb7cab07d30f30387f0a9ae")
  
  
![digoal's wechat](../pic/digoal_weixin.jpg "f7ad92eeba24523fd47a6e1a0e691b59")
  
