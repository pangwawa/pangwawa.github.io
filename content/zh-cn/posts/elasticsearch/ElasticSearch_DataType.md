---
title: "ElasticSearch 7.9 数据类型和Mapping参数详解"
categories: ["技术博客"]
tags: ["ElasticSearch", "OLAP","大数据"]
author: "Jack Wu"
date: 2020-02-10T09:23:44+08:00
draft: false
---
# ElasticSearch数据类型
### 基础类型
#### String
keyword字段通常用于排序， 聚合和术语级查询

**keyword类型**

（keyword类型可设置属性：boost、doc_values、eager_global_ordinals、fields、ignore_above、index、index_options、norms、null_value、store、similarity、normalizer、split_queries_on_whitespace、meta）
	
**constant_keyword类型**

提交的值只能是固定的或者没有该值
（constant_keyword类型可设置的属性：meta、value）
	
**wildcard类型**

存储为通配符grep式查询优化的值
	（可设置的参数：ignore_above）
	wildcard 字段像关键字字段一样是未标记的，因此不支持依赖词位置的查询，例如短语查询

**text类型**

text将对字段值进行分析以进行全文本搜索
（可设置属性：analyzer、boost、eager_global_ordinals、fielddata、fielddata_frequency_filter、fields、index、index_options、index_prefixes、index_pharses、norms、position_increment_gap、store、search_analyzer、search_quote_analyzer、similarity、term_vector、meta）

#### 数字

**long类型**

**integer类型**

**short类型**

**byte类型**

**double类型**
    
**float类型**

**half float类型**

**scaled float类型**

（数字类型可设置参数：coerce、boost、doc_values、ignore_malformed、index、null_value、source、meta；scaled_float 接受一个附加参数：scaling_factor）
#### 时间

**date类型**

（date类型可设置的属性：boost、doc_values、index、null_value、store、meta、format、locale、ignore_malformed）
 	**日期格式可以自定义，但是如果未format指定，则使用默认格式：“ strict_date_optional_time || epoch_millis” 
 	
**date_nanos类型**

 date_nanos类型（是date的补充，增加了纳秒级别的时间戳和格式时间支持，1420070400 ）
 	**日期格式可以自定义，但是如果未format指定，则使用默认格式：“ strict_date_optional_time || epoch_millis”
#### 布尔
**boolean类型**

（boolean可设置属性：boost、doc_values、index、null_value、store、meta）
#### 二进制
**binary类型**

（binary类型可设置的属性，doc_values，store）

#### 区间

**integer range类型**

**float range类型**

**long ranage类型**

**double range类型**

**date range类型**

**ip_range类型**

（可设置属性：coerce、boost、index、store）
#### 复杂文档

**Array数组类型**

没有特定的mapping类型，elasticsearch默认支持相同类型的多个值

**Object类型**

对象类型，可包含内部对象	
	（object对象可设置属性：dynamic、enabled、properties、）
	
**Nested类型**

Neste是object数据类型的专用版本
nested字段可设置的属性：dynamic、properties、include_in_parent、include_in_root； 设置限制：index.mapping.nested_fields.limit，index.mapping.nested_objects.limit）
如果您需要索引对象数组并保持数组中每个对象的独立性，请使用nested数据类型而不是 object数据类型。
#### GEO地理位置

**geo point类型**

存储经纬度
	（可设置的属性参数：ignore_malformed、ignore_z_value、null_value）
	场景：
		以在边界框内，中心点一定距离内，多边形内或geo_shape查询中找到地理位置。
		以地理位置 或距中心点的距离汇总文档。
		将距离整合到文档的相关性分数中。
		按距离对文档 进行排序。

**geo shape类型**

#### IP
**ip类型**

存储ip4、ip6地址
	（ip类型可设置属性:boost、doc_values、index、null_values、store）
#### 自动补全

**completion类型**

#### String长度

**token_count类型**

#### 父子索引Join

**Join类型**：存储存在父子关系的文档，树状结构等

#### 别名

**alias类型**

alias类型：字段名称的别名搜索
	示例：
	
	PUT trips
	{
	  "mappings": {
	    "properties": {
	      "distance": {
	        "type": "long"
	      },
	      "route_length_miles": {
	        "type": "alias",
	        "path": "distance" 
	      },
	      "transit_mode": {
	        "type": "keyword"
	      }
	    }
	  }
	}
#### Dense vector
**dense_vector类型**
密集矢量类型
#### Histogram
**histogram直方图数据类型**

存储直方图数据，两个数组必须等长
####Flattened
**flattened类型**

此数据类型对于索引具有大量或未知数量的唯一键的对象很有用。将整个对象映射为单个字段，仅为整个JSON对象创建一个字段映射
flattened类型可设置的属性：boost、depth_limit、doc_values、eager_global_ordinals、ignore_above、index、index_options、null_value、similarity，split_queries_on_whitespace）

#### Percolator
**percolator类型**

#### Point
**point类型**

二维坐标点
（可设置属性：ignore_malformd、ignore_z_value、null）

#### Rank feature
**rank_featur类型**

用于rank_feature查询
#### Rank features
**rank_features类型**

用户多个属性的rank_feature查询
	
#### Sparse vector
**sparse_vector类型**

#### Shape
**shape类型**

#### Search-as-you-type
**search_as_you_type类型**

用于特定搜素的数据存储，前缀、中缀搜索
	（特定设置属性：max_shingle_size）

# ElasticSearch的Mapping参数
以下映射参数对于某些或所有字段数据类型是通用的：

#### analyzer
  仅text字段支持analyzer映射参数，用于指定文本分析的分析器，   
#### boost
   修改字段的得分权重（不建议使用）
#### coerce
   true|false,将数据以合适的值进行存储
   字符串将被强制转换为数字。
   浮点将被截断为整数值。
#### copy_to
   true|false ,将多个字段的值复制到组字段中，然后可以将其作为单个字段进行查询，提高搜索嘟嘟
#### doc_values
true|false: 字段排序或聚合

几乎所有字段类型都支持Doc值，但textandannotated_text字段除外。

默认情况下，所有支持doc值的字段均已启用它们,不需要可关闭从而节约磁盘空间
   
#### dynamic
true|false|strict

该dynamic设置控制是否可以动态添加新字段。
#### eager_global_ordinals
如果您正在优化索引速度，那么这是正确的方法，但是如果搜索性能是优先考虑的情况，建议您急切地在将用于聚合的字段上急切加载全局序号：
#### enabled
enabled设置仅可应用于顶级映射定义和object字段，设置是否建立索引
#### fielddata
text字段使用称为的查询时内存数据结构 fielddata
#### fields
为不同的目的以不同的方式对同一字段建立索引
#### format
设置日期的格式
#### ignore_above
长于ignore_above设置的字符串将不会被索引或存储。
#### ignore_malformed
是否拒接错误数据类型的文档
#### index_options
该index_options参数控制将哪些信息添加到反向索引中以进行搜索和突出显示，这个参数只能在text中使用
#### index_phrases
加快词语搜索
#### index_prefixes
加快前缀搜索
#### index
是否对字段值建立索引
#### meta
设置字段限制
#### normalizer
#### norms
字段是否参与评分
#### null_value
使用指定的值替换空值，以便搜索
#### position_increment_gap
#### properties
#### search_analyzer
#### similarity
相似度算法
#### store
对字段值进行索引时，是否存储数据
#### term_vector
存储特定术语以方便文档检索