## PostgreSQL 应用开发解决方案最佳实践系列课程 - 8. 树状图谱关系系统(营销、分销、流量变现、分佣、引爆流行、裂变式传播、家谱、选课、社交、人才库、刑侦、农产品溯源、药品溯源)            
                  
### 作者                  
digoal                  
                  
### 日期                  
2021-05-10                   
                  
### 标签                  
PostgreSQL , 解决方案 , 最佳实践 , 课程                   
                  
----                  
                  
## 背景                  
本文信息量很大.     
      
PostgreSQL 应用开发解决方案最佳实践系列课程 - 8. 树状图谱关系系统(营销、分销、流量变现、分佣、引爆流行、裂变式传播、家谱、选课、社交、人才库、刑侦、农产品溯源、药品溯源)   
                  
[视频回放](xx)                
                
## 课程对象                  
数据库架构师, 应用开发者, DBA.                   
                  
## 1、应用场景                  
     
- 营销、分销业务系统:  
    - 多层分佣, 递归关系  
    - 短视频工会, 联盟等分级体系.
    - 喜马拉雅这类音频的工会、联盟等分级体系.
    - 网约车运营公司、联盟等分级体系.  
- 流量变现系统   
    - 裂变传播, 按裂变递归分组统计分析裂变效果   
- 分佣例如类保险行业    
    - 多层分佣, 递归关系  
- 引爆流行系统   
    - 类似裂变传播, 按裂变层级设置积分规则, 刺激裂变  
- 家谱    
    - 树状关系数据  
- 选课    
    - 选了A课的同学还选了其他哪些课(并按热度TOP返回). 用于图谱推荐.   
- 社交关系   
    - like了A的人还like哪些人. 用于图谱推荐.  
    - 一度关系, N度关系  
    - 两个用户之间的最短关系路径  
- 人才库人脉系统    
    - 一度关系, N度关系  
    - 两个用户之间的最短关系路径  
- 刑侦系统  
    - 一度关系, N度关系   
    - 两个相关人之间的最短关系路径   
- 金融风控   
    - 一度关系, N度关系   
- 农产品溯源   
    - 递归追溯源头   
- 药品溯源   
    - 递归追溯源头   
  
                  
## 2、业界解决方案                  
          
应用连数据库      
数据库增量订阅到MQ      
MQ同步到图数据库     
应用连图数据库搜索      
应用既要访问关系数据库又要访问图数据库, 并在应用中合并结果   
             
## 3、业界解决方案的挑战或痛点          
      
开发成本增加      
维护成本增加      
软硬件成本增加      
同步延时增加, 图谱数据更新慢, 无法实时写入实时查询      
同步容易出现错误, 导致查询结果不一致, 需定期刷全量或定期检查和修复差异      
跨库查询代价高, 例如业务查询需要同时访问图数据库和关系数据库, 数据的JOIN需要在应用层实现, 如果JOIN数据量大效率将极差.      
图数据库不支持GIS, 无法实现既有图又有GIS的多重搜索需求    
  
                  
## 4、PG的解决方案        
    
PostgreSQL 内置图式搜索功能, 性能高   
一份数据, 不需要同步数据, 无同步延迟.     
PG还支持GIS, 支持按地理位置、中文分词、模糊搜索、数组包含等的综合搜索  
开发成本低    
维护成本低    
软硬件成本低    
无需跨库查询   
  
   
1、cte recursive query  
2、ltree 树类型    
  
  
## 5、PG的解决方案原理       
  
1、cte recursive query  
2、ltree 树类型             
          
                  
## 6、PG的解决方案 VS 业界解决方案                  
         
PostgreSQL 内置图式搜索功能, 性能高   
一份数据, 不需要同步数据, 无同步延迟.     
PG还支持GIS, 支持按地理位置、中文分词、模糊搜索、数组包含等的综合搜索  
开发成本低    
维护成本低    
软硬件成本低    
无需跨库查询   
            
## 7、DEMO                  
                  
准备工作                  
ECS , Linux + PostgreSQL 客户端软件                  
阿里云 RDS PostgreSQL 13           
            
                  
### 7.1、 公安刑侦  
[《金融风控、公安刑侦、社会关系、人脉分析等需求分析与数据库实现 - PostgreSQL图数据库场景应用》](../201612/20161213_01.md)    
  
  
### 7.2、 百亿级图谱关系实时搜索  
[《PostgreSQL 图式搜索(graph search)实践 - 百亿级图谱，毫秒响应》](../201801/20180102_04.md)      
  
### 7.3、 家谱  
[《PostgreSQL 家族图谱、社交图谱、树状关系、藤状分佣、溯源、等场景实践 - 递归,with recursive query (有向无环 , 有向有环)》](../202003/20200329_01.md)    
[《PostgreSQL 家谱、族谱类应用实践 - 图式关系存储与搜索》](../201804/20180408_03.md)    
  
  
### 7.4、 溯源  
[《PostgreSQL 硬通胀产品(茅台、藏品等) 打假、防伪、溯源 区块链应用 结构设计和demo》](../202009/20200904_01.md)     
  
### 7.5、 裂变式传播  
[《PostgreSQL 递归应用实践 - 非“传销”的高并发实时藤、树状佣金分配体系》](../201808/20180808_02.md)    
  
### 7.6、 共享充电宝实时经营分析系统  
[《经营、销售分析系统DB设计之PostgreSQL, Greenplum - 共享充电宝 案例实践》](../201709/20170923_01.md)    
    
## 8、知识点回顾   
  
1、cte recursive query  
https://www.postgresql.org/docs/devel/queries-with.html#QUERIES-WITH-RECURSIVE  
  
2、ltree 树类型     
https://www.postgresql.org/docs/devel/ltree.html  
                  
## 9、参考文档                  
[《PostgreSQL 15大惊奇应用场景实践 - 直播预告》](../202009/20200903_02.md)     
  
https://help.aliyun.com/document_detail/142340.html  
  
[《PostgreSQL 家谱、族谱类应用实践 - 图式关系存储与搜索》](../201804/20180408_03.md)    
  
[《PostgreSQL 递归应用实践 - 非“传销”的高并发实时藤、树状佣金分配体系》](../201808/20180808_02.md)    
  
[《PostgreSQL 硬通胀产品(茅台、藏品等) 打假、防伪、溯源 区块链应用 结构设计和demo》](../202009/20200904_01.md)     
  
[《PostgreSQL 家族图谱、社交图谱、树状关系、藤状分佣、溯源、等场景实践 - 递归,with recursive query (有向无环 , 有向有环)》](../202003/20200329_01.md)    
      
[《PostgreSQL 大学选课相关性应用实践》](../201801/20180105_02.md)    
  
[《PostgreSQL 图式搜索(graph search)实践 - 百亿级图谱，毫秒响应》](../201801/20180102_04.md)      
  
[《金融风控、公安刑侦、社会关系、人脉分析等需求分析与数据库实现 - PostgreSQL图数据库场景应用》](../201612/20161213_01.md)    
  
[《PostgreSQL 递归查询在分组合并中的用法》](../202011/20201125_01.md)    
  
[《递归+排序字段加权 skip scan 解决 窗口查询多列分组去重的性能问题》](../202006/20200615_01.md)    
  
[《PostgreSQL 排序去重limit查询优化 - 递归 vs group分组 (loop降到极限, block scan降到极限)》](../202005/20200515_01.md)    
  
[《累加链条件过滤 - 递归、窗口、UDF、游标、模拟递归、scan 剪切》](../202002/20200228_01.md)    
  
[《PostgreSQL 递归妙用案例 - 分组数据去重与打散》](../201804/20180406_01.md)    
  
[《PostgreSQL Oracle 兼容性之 - INDEX SKIP SCAN (递归查询变态优化) 非驱动列索引扫描优化》](../201803/20180323_03.md)    
  
[《PostgrSQL 递归SQL的几个应用 - 极客与正常人的思维》](../201705/20170519_01.md)    
  
[《PostgreSQL 递归查询CASE - 树型路径分组输出》](../201703/20170324_01.md)    
  
[《用PostgreSQL找回618秒逝去的青春 - 递归收敛优化》](../201612/20161201_01.md)    
  
[《distinct xx和count(distinct xx)的变态递归优化方法 - 索引收敛(skip scan)扫描》](../201611/20161128_02.md)    
  
[《时序数据合并场景加速分析和实现 - 复合索引，窗口分组查询加速，变态递归加速》](../201611/20161128_01.md)    
  
[《PostgreSQL 使用递归SQL 找出数据库对象之间的依赖关系 - 例如视图依赖》](../201607/20160725_01.md)    
  
[《PostgreSQL 递归死循环案例及解法》](../201607/20160723_01.md)    
  
[《PostgreSQL 递归查询一例 - 资金累加链》](../201604/20160405_01.md)    
  
[《PostgreSQL Oracle 兼容性之 - WITH 递归 ( connect by )》](../201512/20151221_02.md)    
  
[《递归优化CASE - group by & distinct tuning case : use WITH RECURSIVE and min() function》](../201210/20121009_01.md)    
  
[《递归优化CASE - performance tuning case :use cursor\trigger\recursive replace (group by and order by) REDUCE needed blockes scan》](../201209/20120914_01.md)    
  
[《PostgreSQL 树状数据存储与查询(非递归) - Use ltree extension deal tree-like data type》](../201105/20110527_01.md)    
  
[《经营、销售分析系统DB设计之PostgreSQL, Greenplum - 共享充电宝 案例实践》](../201709/20170923_01.md)    
  
  
#### [PostgreSQL 许愿链接](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216")
您的愿望将传达给PG kernel hacker、数据库厂商等, 帮助提高数据库产品质量和功能, 说不定下一个PG版本就有您提出的功能点. 针对非常好的提议，奖励限量版PG文化衫、纪念品、贴纸、PG热门书籍等，奖品丰富，快来许愿。[开不开森](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216").  
  
  
#### [9.9元购买3个月阿里云RDS PostgreSQL实例](https://www.aliyun.com/database/postgresqlactivity "57258f76c37864c6e6d23383d05714ea")
  
  
#### [PostgreSQL 解决方案集合](https://yq.aliyun.com/topic/118 "40cff096e9ed7122c512b35d8561d9c8")
  
  
#### [德哥 / digoal's github - 公益是一辈子的事.](https://github.com/digoal/blog/blob/master/README.md "22709685feb7cab07d30f30387f0a9ae")
  
  
![digoal's wechat](../pic/digoal_weixin.jpg "f7ad92eeba24523fd47a6e1a0e691b59")
  
