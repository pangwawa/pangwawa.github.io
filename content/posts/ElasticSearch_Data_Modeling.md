---
title: "ElasticSearch Mapping 数据建模规范与建模过程"
tags: ["ElasticSearch", "OLAP","大数据"]
author: "Jack Wu"
date: 2020-02-16T09:41:35+08:00
draft: false
---

##Mapping数据建模

数据建模即创建数据模型的过程，它主要分为以下的步骤

概念分析：确定系统的核心需求和范围边界，设计实现和实体间的关系
逻辑模型：进一步梳理业务需求，确定每个实体的属性、关系和约束等
物理模型：结合具体的数据库产品，在满足业务读写性能等需求的前提下确定最终的定义

ElasticSearch索引建立可以遵循一个流程：字段类型——是否要搜索及分词——是否要聚合及排序——是否要额外的存储
#### 是何种类型？
字符串类型：需要分词设定为text类型，否则设置为keyword类型
枚举类型：基于性能考虑将其设定为keyword类型，即便该数据为整型（如状态码）
数值类型：尽量选择贴近的类型，比如byte即可表示所有数值时，即选用byte,不要用long
其他类型：比如布尔类型、日期、地理位置数据等

#### 是否需要检索？
完全不需要检索、排序、聚合分析的字段：enable设置为false
不需要检索的字段：index设置为false
需要检索的字段，可以通过如下配置设定需要的存储粒度
index_options: 结合需要设定
norms: 不需要归一化数据时关闭即可
####是否需要排序和聚合分析？
不需要排序或者聚合分析功能：

doc_values设定为false
fielddata设定为false
####是否需要专门存储当前字段的数据？
store设定为true,即可存储该字段的原始内容（与_source中的不相关）
一般结合_source的enabled设定为false时使用

索引建立Mapping的过程如下图所示

![建模过程](/images/Elasticsearch_mapping_process.png)

mapping 设计非常重要，需要从两个维度进行考虑：

功能：搜索、排序、聚合
性能：存储的开锁、内存的开销、搜索的性能
mapping 注意事项：
加入新字段很容易（必要时需要 update_by_query）
更新删除字段不允许（需要 reindex 重建数据）
 
下面列出Mapping 字段的相关设置：
| 参数 | 取值 | 说明 |
|------|------|------|
|enabled| ture/false | 默认为true, false：仅存储，不做搜索或聚合分析(比如cookie/session字段) |
|index| ture/false | 控制当前字段是否索引，默认为true,即记录索引，false不记录，即不可搜索 |
|index_options| docs/freqs/positions/offsets | 存储倒排索引的哪些信息,text类型默认配置为positions,其他默认为docs ,记录内容越多，占用空间越大。|
|norms|true/false|是否存储归一化相关参数，如果字段仅用于过滤和聚合分析，可关闭|
|doc_values|true/false|是否启用doc_values,用于排序和聚合分析|
|field_data| true/false|是否为text类型启用fielddata,实现排序和聚合分析|
|store|true/false|是否存储该字段值,默认是false|
|coerce|true/false|是否开启自动数据类型转换功能，比如字符串转换为数字、浮点转换为整型等（默认是true|
|multifields|-|多字段-灵活使用多字段特性来解决多样的业务需求|
|dynamic|true/false/strict|控制mapping自动更新|
|date_detection|true/false|是否自动识别日期类型|

常用规则

1. 如果索引不允许自动新增字段，将 dynamic 设置成 strict。默认为 true；

2. 不需要分词的字段，将 type 设置成 keyword。默认使用了多字段特性，text、keyword这2种类型都有；

3. 不需要建立索引的字段，将 index 设置成 false。默认为 true；

4. 不需要排序和聚合的字段，将 doc_values 设置成false。默认为 true；

5. 不需要检查、排序、聚合的字段，将 enable 设置成 false，仅做存储；

6. type = text 的字段，默认不可以排序，如需要排序，将 fielddata 设置成 true，默认为 false；

7. 单个索引避免过多字段，默认最大值为1000；

8. 避免空值引起的聚合不准确的问题；

9. 避免使用正则查询；

10. 尽量不要设计成索引关联，可冗余多一些字段，以空间换时间，如实在无法避免，按以下方式处理：
  - Object:优先考虑Denormalization；
  - Nested： 当数据包含多数值对象，同时有查询请求；
  - Child/Parent:关联文档更新非常频繁时