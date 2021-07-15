---
title: "Flink编程模型详解"
categories: ["技术博客"]
tags: ["Flink","大数据"]
author: "Jack Wu"
date: 2020-10-22T18:19:12+08:00
draft: false
---

Flink 为流式/批式处理应用程序的开发提供了Stateful Stream Processing、DataStream/DataSet ApI 、Table API和SQL这四个不同级别的抽象，如下如所示：

![导图](/images/20201021235817345.png)

#### 1、SQL

这层抽象在语义和程序表达式上都类似于 Table API，但是其程序实现都是 SQL 查询表达式。SQL 抽象与 Table API 抽象之间的关联是非常紧密的，并且 SQL 查询语句可以在 Table API 中定义的表上执行。

#### 2、Table API

以表（Table）为中心的声明式编程（DSL）API，例如在流式数据场景下，它可以表示一张正在动态改变的表。Table API 遵循（扩展）关系模型：即表拥有 schema（类似于关系型数据库中的 schema），并且 Table API 也提供了类似于关系模型中的操作，比如 select、project、join、group-by 和 aggregate 等

#### 3、Datastream、DataSet API

核心 API，包含 DataStream API（应用于有界/无界数据流场景）和 DataSet API（应用于有界数据集场景）两部分。Core APIs 提供的流式 API（Fluent API）为数据处理提供了通用的模块组件，例如各种形式的用户自定义转换（transformations）、联接（joins）、聚合（aggregations）、窗口（windows）和状态（state）操作等。此层 API 中处理的数据类型在每种编程语言中都有其对应的类。

#### 4、Stateful Stream Processing（有状态实时流处理）

允许用户在应用程序中自由地处理来自单流或多流的事件（数据），并提供具有全局一致性和容错保障的状态。此外，用户可以在此层抽象中注册事件时间（event time）和处理时间（processing time）回调方法，从而允许程序可以实现复杂计算。

这4层中，一般用于开发的是第三层，即DataStrem/DataSetAPI。用户可以使用DataStream API处理无界数据流，使用DataSet API处理有界数据流。同时这两个API都提供了各种各样的接口来处理数据。

#### Flink程序的模块结构

如下图，Flink程序主要分为 source-->transformation-->sink 这三个模块

![导图](/images/flink_app_process.png)

在获取执行环境后，程序从数据源中获取数据，并执行Transformation操作，最后将执行结果输出到指定的地方，如消息队列、文件系统、数据库等。

下面这个简单的例子便包含了整个过程：
```
    public static void main(String[] args) throws Exception {

        // Checking input parameters
        final MultipleParameterTool params = MultipleParameterTool.fromArgs(args);

        // set up the execution environment
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // make parameters available in the web interface
        env.getConfig().setGlobalJobParameters(params);

        // get input data
        DataStream<String> text = null;
        if (params.has("input")) {
            // union all the inputs from text files
            for (String input : params.getMultiParameterRequired("input")) {
                if (text == null) {
                    text = env.readTextFile(input);
                } else {
                    text = text.union(env.readTextFile(input));
                }
            }
            Preconditions.checkNotNull(text, "Input DataStream should not be null.");
        } else {
            System.out.println("Executing WordCount example with default input data set.");
            System.out.println("Use --input to specify file input.");
            // get default test text data
            text = env.fromElements(WordCountData.WORDS);
        }

        DataStream<Tuple2<String, Integer>> counts =
                // split up the lines in pairs (2-tuples) containing: (word,1)
                text.flatMap(new Tokenizer())
                        // group by the tuple field "0" and sum up tuple field "1"
                        .keyBy(value -> value.f0).sum(1);

        // emit result
        if (params.has("output")) {
            counts.writeAsText(params.get("output"));
        } else {
            System.out.println("Printing result to stdout. Use --output to specify output path.");
            counts.print();
        }
        // execute program
        env.execute("Streaming WordCount");
    }
```