---
title: "Netty_reactor_model"
tags: ["Netty"]
author: "Jack Wu"
date: 2020-11-26T14:50:48+08:00
draft: false
---

## Netty线程模型
   
Netty主要是基于主从Reactors多线程模型(如下图)做了一些修改，其中主从reactor多线程模型有多个reactor：

MainReactor负责客户端的连接请求，并将请求转交给SubReactor
SubReactor负责相应通道的IO读写请求
非IO请求(具体逻辑处理)的任务则会直接进入写入队列，等到worker threads进行处理

特别说明的是：虽然Netty的线程模型基于主从Reactor多线程，借用了MainReactor和SubReactor的结构。但是实际实现上SubReactor和Worker线程在同一个线程池中

bossGroup线程池则只是在bind某个端口后，获得其中一个线程作为MainReactor，专门处理端口的Accept事件，每个端口对应一个boss线程
workerGroup线程池会被各个SubReactor和Worker线程充分利用

异常处理

异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者
Netty中的IO操作是异步的，包括Bind、Write、Connect等操作会简单的返回一个channelFuture
调用者并不能立刻获得结果，而是通过Future-Listener机制，用户可以方便的主动获取或通过通知机制来获得IO操作结果
当future对象刚刚创建时，处于非完成状态，调用者可以通过返回的ChannelFuture来获取操作执行的状态，注册监听函数来执行完成后的操作

## ChannelHandler的方法
通过ChannelHandler处理IO操作

ChannelPipeline可以动态添加、删除、替换其中的ChannelHandler，这样的机制可以提高灵活性。

ChannelInitializer用来初始化ChannelHandler，将自定义的各种ChannelHandler添加到ChannelPipeline中。

ChannelPipeline提供的方法

• addFirst(...)，添加ChannelHandler在ChannelPipeline的第一个位置
• addBefore(...)，在ChannelPipeline中指定的ChannelHandler名称之前添加ChannelHandler
• addAfter(...)，在ChannelPipeline中指定的ChannelHandler名称之后添加ChannelHandler
• addLast(ChannelHandler...)，在ChannelPipeline的末尾添加ChannelHandler
• remove(...)，删除ChannelPipeline中指定的ChannelHandler
• replace(...)，替换ChannelPipeline中指定的ChannelHandler

  Netty中有3个实现了ChannelHandler接口的类，其中2个是接口（ChannelInboundHandler用来处理入站数据也就是接收数据、ChannelOutboundHandler用来处理出站数据也就是写数据），一个是抽象类ChannelHandlerAdapter类。
      ChannelHandler提供了在它的生命周期内添加或从ChannelPipeline中删除的方法：
      1.handlerAdded:ChannelHandler添加到实际上下文中准备处理事件。
      2.handlerRemoved：将ChannelHandler从实际上下文中删除，不再处理事件。
      3.exceptionCaught：处理跑出的异常。
  
  ChannelInboundHandler类的用法
         它提供了一些方法来接收数据或Channel状态改变时被调用，下面是一些常用方法：
         1.channelRegistered:ChannelHandlerContext的Channel被注册到EventLoop中。
         2.channelUnregistered：ChannelHandlerContext的channel从eventloop中注销。
         3.channelActive方法：ChannelHandlerContext的channel已被激活。
         4.channelInactive方法：ChannelHandlerContext的channel结束生命周期。
         5.channelRead方法：从当前Channel的对端读取消息。
         6.channelReadComplete方法：消息读取完毕有执行。
         7.userEventTriggered方法：一个用户事件被触发。
         8.channelWritabilityChanned方法：改变通道的可写状态，可以使用Channel.isWritable检查。
         9.exceptionCaught，重写父类ChannelHandler的方法，处理异常.
         
         
## Netty自带 Handler


SslHandler:负责对请求进行加密和解密，是放在ChannelPipeline中的第一个ChannelHandler

HttpClientCodec和HttpServerCodec:HttpClientCodec负责将请求字节解码为HttpRequest、HttpContent和LastHttpContent消息，以及对应的转为字节；HttpServerCodec负责服务端中将字节码解析成HttpResponse、HttpContent和LastHttpContent消息，以及对应的将它转为字节
HttpServerCodec 里面组合了HttpResponseEncoder和HttpRequestDecoder

HttpClientCodec 里面组合了HttpRequestEncoder和HttpResponseDecoder

HttpObjectAggregator: 负责将http聚合成完整的消息，而不是原始的多个部分
HttpContentCompressor和HttpContentDecompressor:HttpContentCompressor用于服务器压缩数据，HttpContentDecompressor用于客户端解压数据
IdleStateHandler:连接空闲时间过长，触发IdleStateEvent事件
ReadTimeoutHandler:指定时间内没有收到任何的入站数据，抛出ReadTimeoutException异常,并关闭channel
WriteTimeoutHandler:指定时间内没有任何出站数据写入，抛出WriteTimeoutException异常，并关闭channel
DelimiterBasedFrameDecoder:使用任何用户提供的分隔符来提取帧的通用解码器
FixedLengthFrameDecoder:提取在调用构造函数时的定长帧
ChunkedWriteHandler：将大型文件从文件系统复制到内存【DefaultFileRegion进行大型文件传输】
WebSocketServerProtocolHandler：处理websocket协议，将HttpServerCodec转为websocketFrame,处理websocket握手


线程的工作模式：

在此类中，有两种类型的线程，一种是boss线程，另一种是worker线程

 

Boss线程：

每个server服务器都会有一个boss线程，每绑定一个InetSocketAddress都会产生一个boss线程，比如：我们开启了两个服务器端口80和443，则我们会有两个boss线程。一个boss线程在端口绑定后，会接收传进来的连接，一旦连接接收成功，boss线程会指派一个worker线程处理连接。

Worker线程：

一个NioServerSocketChannelFactory会有一个或者多个worker线程。一个worker线程在非阻塞模式下为一个或多个Channels提供非阻塞 读或写

线程的生命周期和优雅的关闭

在NioServerSocketChannelFactory被创建的时候，所有的线程都会从指定的Executors中获取。Boss线程从bossExecutor中获取，worker线程从workerExecutor中获取。因此，我们应该准确的指定Executors可以提供足够数量的线程，最好的选择就是指定一个cached线程池（It is the best bet to specify a cached thread pool）。

此处发现所有源码中的例子(example)中均设置为Executors.newCachedThreadPool()

Boss线程和worker线程都是懒加载，没有程序使用的时候要释放掉。当boss线程和worker线程释放掉的时候，所有的相关资源如Selector也要释放掉。因此，如果想要优雅的关闭一个服务，需要做一下事情：

对factory创建的channels执行解绑（unbind）操作
关闭所有的由解绑的channels处理的子channels（这两步目前通常通过ChannelGroup.close()来操作）
调用releaseExternalResources()方法
请确保在所有的channels都关闭前不要关闭executor，否则，会报RejectedExecutionException异常而且相关资源可能不会被释放掉。

## Netty ChannelFutureListener

添加异步回调事件
