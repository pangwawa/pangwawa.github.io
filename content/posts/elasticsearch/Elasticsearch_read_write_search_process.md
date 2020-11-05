---
title: "Elasticsearch数据的写入、读取与检索过程详解"
tags: ["ElasticSearch", "OLAP","大数据"]
author: "Jack Wu"
date: 2020-03-07T18:25:18+08:00
draft: false
---

## ElasticSearch写过程

**分片**

   一个分片就是一个运行的Lucenes实例，个节点可以包含多个分片，Es中所有数据均衡的存储在集群中各个节点的分片中
   
   主分片和副本分片

**注意**

1、默认索引是5个分片

2、分片一定设置是不可以修改的，只能新建新索引解决


数据写入过程:

ES的任意节点都可以作为协调节点(coordinating node)接受请求，当协调节点接受到请求后进行一系列处理，然后通过_routing字段找到对应的primary shard，并将请求转发给primary shard, primary shard完成写入后，
将写入并发发送给各replica， raplica执行写入操作后返回给primary shard， primary shard再将请求返回给协调节点

数据持久化步骤如下：write -> refresh -> flush -> merge

   write:一个新文档过来，会存储在 in-memory buffer 内存缓存区中，顺便会记录 Translog。(这时候数据还没到 segment ，是搜不到这个新文档的。数据只有被 refresh 后，才可以被搜索到,设置了refresh时间，所以是准实时)
   
   refresh: 1、in-memory buffer 中的文档写入到新的 segment 中，但 segment 是存储在文件系统的缓存中。此时文档可以被搜索到;
            2、最后清空 in-memory buffer。注意: Translog 没有被清空，为了将 segment 数据写到磁盘
   
   flush:将segment从文件系统缓存写入磁盘。最后清空translog。translog 作用很大：保证文件缓存中的文档不丢失；系统重启时，从 translog 中恢复；新的 segment 收录到 commit point 中
   
   merge： 当磁盘中的segment越来越多，会导致搜索速度变慢，通过merge将小段文件合并为大文件。
   
   
## ElasticSearch读过程
客户端发送请求到任意一个node，成为coordinate node

coordinate node对document进行路由，将请求转发到对应的node，此时会使用round-robin随机轮询算法，在primary shard以及其所有replica中随机选择一个，让读请求负载均衡

接收请求的node返回document给coordinate node

coordinate node返回document给客户端

   写入document时，每个document会自动分配一个全局唯一的id即doc id，同时也是根据doc id进行hash路由到对应的primary shard上。也可以手动指定doc id，比如用订单id，用户id。
   
   读取document时，你可以通过doc id来查询，然后会根据doc id进行hash，判断出来当时把doc id分配到了哪个shard上面去，从那个shard去查询）
 
 ## ElasticSearch 数据搜索过程
 
   客户端发送请求到一个coordinate node
   
   协调节点将搜索请求转发到所有的shard对应的primary shard或replica shard也可以
   
   query phase：每个shard将自己的搜索结果（其实就是一些doc id），返回给协调节点，由协调节点进行数据的合并、排序、分页等操作，产出最终结果

   fetch phase：接着由协调节点，根据doc id去各个节点上拉取实际的document数据，最终返回给客户端