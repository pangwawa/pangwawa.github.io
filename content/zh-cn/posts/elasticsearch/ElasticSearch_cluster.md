---
title: "ElasticSearch集群与调优"
categories: ["技术博客"]
tags: ["ElasticSearch", "OLAP","大数据"]
author: "Jack Wu"
date: 2020-02-29T11:10:33+08:00
draft: false
---

1.1、设计阶段调优 

（1）根据业务增量需求，采取基于日期模板创建索引，通过roll over API滚动索引； 
（2）使用别名进行索引管理； 
（3）每天凌晨定时对索引做force_merge操作，以释放空间； 
（4）采取冷热分离机制，热数据存储到SSD，提高检索效率；冷数据定期进行shrink操作，以缩减存储； 
（5）采取curator进行索引的生命周期管理； 
（6）仅针对需要分词的字段，合理的设置分词器； 
（7）Mapping阶段充分结合各个字段的属性，是否需要检索、是否需要存储等。…….. 

1.2、写入调优 

（1）写入前副本数设置为0； （2）写入前关闭refresh_interval设置为-1，禁用刷新机制； （3）写入过程中：采取bulk批量写入； （4）写入后恢复副本数和刷新间隔； （5）尽量使用自动生成的id。 

1.3、查询调优 

（1）禁用wildcard； （2）、禁用批量terms（成百上千的场景）； （3）充分利用倒排索引机制，能keyword类型尽量keyword； （4）数据量大时候，可以先基于时间敲定索引再检索； （5）设置合理的路由机制。 

1.4、其他调优 部署调优，业务调优等。 上面的提及一部分，面试者就基本对你之前的实践或者运维经验有所评估了。