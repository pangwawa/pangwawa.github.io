---
title: "Elasticsearch的DSL——常用检索、复合检索、高级检索"
tags: ["ElasticSearch", "OLAP","大数据"]
date: 2020-10-20T17:49:40+08:00
draft: false
---



## 简单查询

#### 精准查询term （完全匹配，不使用分词器）

**term单值查询**
```
{
  "query": {
    "term": {
      "productType": {
        "value": "101"
      }
    }
  }
}
```
**term多值查询**
```
{
  "query": {
    "terms": {
      "productType":["101","102"]
    }
  }
}
```

**terms查询多值**
```
{
  "query": {
    "terms": {
      "productType":["101","102"]
    }
  }
}
```

#### 模糊查询 （模糊匹配，使用分词器）

_match 、 match_all、multi_match、match_phrase_

**match_all**
 匹配所有的， 当不给查询条件时，默认全查
match
```
 {
   "query": {
     "match": {
       "username": "张三"
     }
   }
 }
 ```

 **multi_match**
 同时对查询的关键词，多个字段同时进行匹配，即多个字段是AND的关系
 更丰富的查询
 按brandName（品牌名）、sortName（分类名）、productName（商品名）productKeyword（商品关键字），搜索“牛仔 弹力”关键词，
 brandName源值、拼音值、关键字值都是100分，sortName源值、拼音值80分，productName源值60分，productKeyword值20分，分值由高到低优先级搜索
 
 ```
 {
   "query": {
     "multi_match": {
       "query": "牛仔 弹力",
       "fields": [
         "brandName^100",
         "brandName.brandName_pinyin^100",
         "brandName.brandName_keyword^100",
         "sortName^80",
         "sortName.sortName_pinyin^80",
         "productName^60",
         "productKeyword^20"
       ],
       "type": <multi-match-type>,
       "operator": "AND"
     }
   }
 }
 ```

#### match_phrase 
  查询分析文本，并从分析文本中创建短语查询。（即对查询的条件也进行分词，然后使用分词进行匹配查询）
 
#### wildcard：通配符查询
* 表示全匹配，？ 表示单一匹配，etc： aaa* 或者 a?b；

#### prefix：前缀匹配
搜索框如果输入aa,那么可能匹配到的字段值为 aab,aavb等；

#### fuzzy min_similarity：弹性模糊匹配
有两个搜索框，第一个搜索框为搜索匹配值，会自动纠错，比如输入 ggjk,那么可能会匹配到ggjo，第二个框为最小相似度，采用的算法是Damerau-Levenshtein（最佳字符串对齐）算法，不建议填写这个框，我到发稿前也是被搞的头皮发麻，等我完全吃透再更新；

#### fuzzy max_expansions ：弹性模糊匹配
有两个搜索框，第一个搜索框为搜索匹配值，会自动纠错，比如输入gjk,那么可能会匹配到ggjo，第二个框是最大扩展匹配数，比如是1，那么ggjk只会随机模糊匹配到一种可能结果，即使它会出现2种或者更加多，也只会搜索一种；

#### range：范围查询
gt为大于，gte为大于等于，lt小于，lte小于等于，所搜索的字段值在两个搜索框标识数值之间；

####  query_string：字符片段查询，
如果是数字，则严格匹配数字，如果是字符串，则按照自身或者分片词匹配；

#### text：分片词查询，等确定后更新；

#### missing：查询没有定义该字段或者该字段值为null的数据。

  
##  compound query(复合查询)bool query、boosting query、constant_score query、dis_max query、 function_score 的区别

#### bool query（多查询或复合查询的默认查询方式）
     
用于组合多个叶子或复合查询子句的默认查询，包含must, should, must_not, or filter 。must和should子句将它们的分数组合在一起 - 匹配子句越多越好 - 而must_not和filter子句在过滤器上下文中执行。

　　must：返回的文档必须满足must子句的条件，并且参与计算分值

　　filter：返回的文档必须满足filter子句的条件。不参与计算分值

　　must_not：返回的文档必须不满足must_not定义的条件。不参与评分。

　　should：返回的文档可能满足should子句的条件。在一个Bool查询中，如果没有must或者filter，有一个或者多个should子句，那么只要满足一个就可以返回。minimum_should_match参数定义了至少满足几个子句。

注1：评分计算
    bool 查询会为每个文档计算相关度评分 _score ， 再将所有匹配的 must 和 should 语句的分数 _score 求和，最后除以 must 和 should 语句的总数。
    must_not 语句不会影响评分； 它的作用只是将不相关的文档排除。

注2：控制精度
    所有 must 语句必须匹配，所有 must_not 语句都必须不匹配，但有多少 should 语句应该匹配呢？ 默认情况下，没有 should 语句是必须匹配的，只有一个例外：那就是当没有 must 语句的时候，至少有一个 should 语句必须匹配。
    就像我们能控制 match 查询的精度 一样，我们可以通过 minimum_should_match 参数控制需要匹配的 should 语句的数量， 它既可以是一个绝对的数字，又可以是个百分比：
     
#### boosting query
     
提升查询可用于有效降级与给定查询匹配的结果。与bool查询中的“NOT”子句不同，这仍然会选择包含不良术语的文档，但会降低其总分。
     
#### constant_score query
含另一个查询但在过滤器上下文中执行的查询。所有匹配的文档都给出相同的“常量”_score
     
#### dis_max query
     
一个接受多个查询的查询，并返回与任何查询子句匹配的任何文档。虽然bool查询组合了所有匹配查询的分数，但dis_max查询使用单个最佳匹配查询子句的分数。
     
一个查询，它生成由其子查询生成的文档的并集，并为每个文档评分由任何子查询生成的该文档的最大分数，以及任何其他匹配子查询的平局增量。
     
当在具有不同增强因子的多个字段中搜索单词时，这非常有用（因此不能将字段等效地组合到单个搜索字段中）。我们希望主要分数是与最高提升相关联的分数，而不是字段分数的总和（如布尔查询所给出的）。如果查询是“albino elephant”，则这确保匹配一个字段的“albino”和匹配另一个的“elephant”获得比匹配两个字段的“albino”更高的分数。要获得此结果，请同时使用Boolean Query和DisjunctionMax Query：对于每个术语，DisjunctionMaxQuery在每个字段中搜索它，而将这些DisjunctionMaxQuery的集合组合成BooleanQuery。　
     
#### function_score query
     
使用函数修改主查询返回的分数，以考虑流行度、最近度、距离或使用脚本实现的自定义算法等因素。
     	
#### 地理坐标的查询

**矩形范围内 查询 geo_bounding_box query


坐标距离范围 查询 geo_distance query
```
{
  "query": {
    "geo_distance": {
      "distance": "100km",
      "location": {
        "lat": 40,
        "lon": -70
      }
    }
  }
} 
```

多边形范围内查询 geo_polygon query


geo_shape query

  geo-shapes 与指定的几何形状相交，包含于其中或不与指定的几何形状相交的对象
  geo-points 与指定的地理形状相交
 
 #### 查询中Query context和Filter context的区别、什么是相关性得分
 
1、查询上下文中，查询操作不仅仅会进行查询，还会计算分值，用于确定相关度；在过滤器上下文中，查询操作仅判断是否满足查询条件

2、 过滤器上下文中，查询的结果可以被缓存。
  