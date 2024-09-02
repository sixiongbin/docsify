## PostgreSQL 应用开发解决方案最佳实践系列课程 - 3. 人脸识别和向量相似搜索      
        
### 作者        
digoal        
        
### 日期        
2021-05-06         
        
### 标签        
PostgreSQL , 解决方案 , 最佳实践 , 课程         
        
----        
        
## 背景        
      
      
PostgreSQL 应用开发解决方案最佳实践系列课程 - 3. 人脸识别和向量相似搜索     
        
[视频回放](xx)      
      
## 课程对象        
数据库架构师, 应用开发者, DBA.         
        
## 1、应用场景        
        
1、新零售、酒店、餐饮、会所等, 会员到店提醒和营销系统     
2、公共场所安全    
3、刷脸打卡、支付等场景     
        
## 2、业界解决方案        
1、暴力计算  
2、应用程序自行实现向量搜索加速  
        
## 3、业界解决方案的挑战或痛点        
采用暴力计算搜索效率低  
应用实现加速的弊端1: 数据加载时间久, 每次应用启动需要全量加载数据并构建加速结构  
应用实现加速的弊端2: 数据维护非常麻烦, 更新后要通知所有应用, 并且应用需要重构加速结构, 无法实现全局一致性.   
应用实现加速的弊端3: 实现效率取决于技术团队, 掌握在少数企业, 甲方的议价能力弱同时风险高. 
  
        
## 4、PG的解决方案        
        
pase 向量加速索引.  
将图片特征转换为高维浮点数组, 对浮点数组创建向量索引, 按向量距离排序返回. 实现高效率相似搜索.   
        
## 5、PG的解决方案原理        
pase插件  
ivfflat  
hnsw  
        
## 6、PG的解决方案 VS 业界解决方案        
PG 优势:  
效率高  
无需提前加载数据   
保证数据一致性  
扩展方便, 增加只读实例即可  
采用蚂蚁的人脸识别算法, 效率非常高, 精度有保障, 有大规模应用实践. 方案具有通用性, 不依赖开发商技术能力.  
        
  
## 7、DEMO        
        
准备工作        
ECS , Linux + PostgreSQL 客户端软件        
阿里云 RDS PostgreSQL 11 (目前只有RDS PG 11 支持pase 插件)      
  
        
### 7.1、ivfflat 向量搜索      
创建pase插件  
建表      
生成数据    
ivfflat 索引    
使用 ivfflat 索引进行向量搜索    
性能      
      
### 7.2、hnsw 向量搜索      
创建pase插件      
建表      
生成数据    
hnsw 索引    
使用 hnsw 索引进行向量搜索    
性能   
      
  
        
## 8、知识点回顾        
        
pase插件  
ivfflat索引  
hnsw索引  
        
## 9、参考文档        
[《PostgreSQL 15大惊奇应用场景实践 - 直播预告》](../202009/20200903_02.md)        
    
https://help.aliyun.com/document_detail/147837.html  
  
[有个开源插件, 根据PASE提供的论文, 实现了ivfflat向量索引, 有兴趣的用户也可以看看](https://pgxn.org/dist/vector/0.1.2/)  
      
[《阿里云PostgreSQL案例精选2 - 图像识别、人脸识别、相似特征检索、相似人群圈选》](202002/20200227_01.md)    
  
[《PostgreSQL 阿里云rds pg发布高维向量索引，支持图像识别、人脸识别 - pase 插件》](201912/20191219_02.md)    
  
[《[直播]刷脸支付会不会刷到别人的钱包?》](202009/20200919_01.md)    
    
  
#### [PostgreSQL 许愿链接](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216")
您的愿望将传达给PG kernel hacker、数据库厂商等, 帮助提高数据库产品质量和功能, 说不定下一个PG版本就有您提出的功能点. 针对非常好的提议，奖励限量版PG文化衫、纪念品、贴纸、PG热门书籍等，奖品丰富，快来许愿。[开不开森](https://github.com/digoal/blog/issues/76 "269ac3d1c492e938c0191101c7238216").  
  
  
#### [9.9元购买3个月阿里云RDS PostgreSQL实例](https://www.aliyun.com/database/postgresqlactivity "57258f76c37864c6e6d23383d05714ea")
  
  
#### [PostgreSQL 解决方案集合](https://yq.aliyun.com/topic/118 "40cff096e9ed7122c512b35d8561d9c8")
  
  
#### [德哥 / digoal's github - 公益是一辈子的事.](https://github.com/digoal/blog/blob/master/README.md "22709685feb7cab07d30f30387f0a9ae")
  
  
![digoal's wechat](../pic/digoal_weixin.jpg "f7ad92eeba24523fd47a6e1a0e691b59")
  
