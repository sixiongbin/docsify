## PostgreSQL 应用开发解决方案最佳实践系列课程 - 2. 短视频业务实时推荐    
      
### 作者      
digoal      
      
### 日期      
2021-05-03       
      
### 标签      
PostgreSQL , 解决方案 , 最佳实践 , 课程       
      
----      
      
## 背景      
    
短视频实时推荐算法     
随机采样     
HLL     
roaringbitmap     
短视频业务实时推荐 demo     
    
PostgreSQL 应用开发解决方案最佳实践系列课程 - 2. 短视频业务实时推荐     
      
[视频回放](xx)    
    
## 课程对象      
数据库架构师, 应用开发者, DBA.       
      
## 1、应用场景      
      
1、短视频      
实时推荐      
      
## 2、业界解决方案      
1、随机     
      
## 3、业界解决方案的挑战或痛点      
    
成本     
实时性差     
随机无喜好关系     
重复播放现象     
过滤已读需要存储大量数据    
过滤扫描代价极大 (not in 百万级)     
视频标签权重实时变化     
池(大、地理位置、热搜)     
交互次数    
      
## 4、PG的解决方案      
      
HLL或roaringbitmap 少量空间存储已读    
partial index解决大量过滤已读的无用消耗    
sub 单次交互    
异步刷新权重    
      
## 5、PG的解决方案原理      
Hyperloglog    
roaring bitmap    
partial index    
      
## 6、PG的解决方案 VS 业界解决方案       
      
## 7、DEMO      
      
准备工作      
ECS , Linux + PostgreSQL 客户端软件      
阿里云 RDS PostgreSQL 12 (由于RDS PG 13暂时只支持hll, 不支持pg_roaringbitmap)      
      
### 7.1、实时随机采样推荐    
建表    
生成数据    
随机采样查询    
性能    
    
### 7.2、实时精准喜好推荐    
    
建表    
生成数据    
创建索引    
精准喜好推荐查询    
    
    
### 7.3、性能      
大量已读过滤查询    
优化大量已读过滤查询    
    
### 7.4、扩展性      
垂直扩展      
水平扩展       
只读实例      
partial index     
      
## 8、知识点回顾      
      
随机采样    
sub    
HLL    
roaringbitmap    
partial index    
      
## 9、参考文档      
[《PostgreSQL 15大惊奇应用场景实践 - 直播预告》](../202009/20200903_02.md)      
  
https://help.aliyun.com/document_detail/164024.html  
  
https://help.aliyun.com/document_detail/154079.htm  
  
https://help.aliyun.com/document_detail/183124.html  
    
[《PostgreSQL 推荐系统优化总计 - 空间、时间、标量等混合多模查询场景, 大量已读过滤导致CPU IO剧增(类挖矿概率下降优化)》](../202006/20200612_01.md)    
  
[《推荐系统, 已阅读过滤, 大量CPU和IO浪费的优化思路2 - partial index - hash 分片， 降低过滤量》](../202006/20200610_02.md)    
  
[《PostgreSQL 大量IO扫描、计算浪费的优化 - 推荐模块, 过滤已推荐. (热点用户、已推荐列表超大)》](../202006/20200601_01.md)    
  
[《PostgreSQL 随机采样应用 - table sample, tsm_system_rows, tsm_system_time》](../202005/20200509_01.md)    
  
[《社交、电商、游戏等 推荐系统 (相似推荐) - 阿里云pase smlar索引方案对比》](../202004/20200421_01.md)    
  
[《PostgreSQL 向量相似推荐设计 - pase》](../202004/20200424_01.md)    
  
[《用户喜好推荐系统 - PostgreSQL 近似计算应用》](../202002/20200228_02.md)    
  
[《基于GIS位置、群众热点、人为热点的内容推荐服务 - 挑战和策略》](../202002/20200226_01.md)    
  
[《[视频直播]亿级用户量的实时推荐数据库到底要几毛钱?》](../202009/20200910_02.md)    
    
  
      
  
#### [PostgreSQL 许愿链接](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216")
您的愿望将传达给PG kernel hacker、数据库厂商等, 帮助提高数据库产品质量和功能, 说不定下一个PG版本就有您提出的功能点. 针对非常好的提议，奖励限量版PG文化衫、纪念品、贴纸、PG热门书籍等，奖品丰富，快来许愿。[开不开森](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216").  
  
  
#### [9.9元购买3个月阿里云RDS PostgreSQL实例](https://www.aliyun.com/database/postgresqlactivity "57258f76c37864c6e6d23383d05714ea")
  
  
#### [PostgreSQL 解决方案集合](https://yq.aliyun.com/topic/118 "40cff096e9ed7122c512b35d8561d9c8")
  
  
#### [德哥 / digoal's github - 公益是一辈子的事.](https://github.com/digoal/blog/blob/master/README.md "22709685feb7cab07d30f30387f0a9ae")
  
  
![digoal's wechat](../pic/digoal_weixin.jpg "f7ad92eeba24523fd47a6e1a0e691b59")
  
