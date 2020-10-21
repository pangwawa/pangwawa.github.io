---
title: "ElasticSearch 7.x 数据类型详解"
tags: ["ElasticSearch", "OLAP","大数据"]
author: "Jack Wu"
date: 2020-02-10T09:23:44+08:00
draft: false
---

### 基础类型
#### String类型
**keyword**

keyword字符串按原样保留以进行过滤和排序。

**text**

text将对字段值进行分析以进行全文本搜索

#### 数字类型

**long**

**integer**

**short**

**byte**

**double**

**float**

**half float**

**scaled float**

#### 时间类型

**date**

#### 布尔类型

**boolean

#### 二进制类型

**binary**

#### 区间类型

**integer range**

**float range**

**long ranage**

**double range**

**date range**

#### 复杂类型

**Array数组**

**Object对象**

**Nested类型**

#### GEO地理位置类型

**geo point**

**geo shape**

#### IP类型

**ip**

#### 自动补全类型

**completion**

#### String长度类型

**token_count**

#### percolate类型

**mumur3**

#### 父子索引Join类型

**percolator**

#### 别名类型

**alias**