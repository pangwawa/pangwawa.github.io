---
title: "ElasticSearch Mapping 数据建模规范与建模过程"
tags: ["ElasticSearch", "OLAP","大数据"]
author: "Jack Wu"
date: 2020-02-16T09:41:35+08:00
draft: false
---

##Mapping数据建模

字段类型————是否要搜索及分词————是否要聚合及排序————是否要额外的存储



mapping 设计非常重要，需要从两个维度进行考虑：

功能：搜索、排序、聚合
性能：存储的开锁、内存的开销、搜索的性能
mapping 注意事项：

加入新字段很容易（必要时需要 update_by_query）
更新删除字段不允许（需要 reindex 重建数据）
 
最佳实践

1、不允许自动新增字段，将 dynamic 设置成 strict。默认为 true；

2、不需要分词的字段，将 type 设置成 keyword。默认使用了多字段特性，text、keyword这2种类型都有；

3、不需要检查的字段，将 index 设置成 false。默认为 true；

4、不需要排序和聚合的字段，将 doc_values 设置成false。默认为 true；

5、不需要检查、排序、聚合的字段，将 enable 设置成 false，仅做存储；

6、type = text 的字段，默认不可以排序，如需要排序，将 fielddata 设置成 true，默认为 false；

7、单个索引避免过多字段，默认最大值为1000；

8、避免空值引起的聚合不准确的问题；

9、避免使用正则查询；

10、尽量不要设计成索引关联，可冗余多一些字段，以空间换时间，如实在无法避免，按以下方式处理：