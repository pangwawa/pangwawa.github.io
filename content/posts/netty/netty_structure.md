---
title: "Netty基础——基础概念与架构原理"
tags: ["Netty"]
author: "Jack Wu"
date: 2020-11-25T14:41:07+08:00
draft: false
---

##Netty
一个NIO Java框架，对NIO底层进行了很好的封装
Netty提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序。
快速和简单的开发出一个网络应用，例如实现了某种协议的客户、服务端应用。Netty相当于简化和流线化了网络应用的编程开发过程，例如：基于TCP和UDP的socket服务开发。
etty 是一个吸收了多种协议（包括FTP、SMTP、HTTP等各种二进制文本协议）的实现经验，并经过相当精心设计的项目。最终，Netty 成功的找到了一种方式，在保证易于开发的同时还保证了其应用的性能，稳定性和伸缩性

## 为什么使用Netty
有了Netty，你可以实现自己的HTTP服务器，FTP服务器，UDP服务器，RPC服务器，WebSocket服务器，Redis的Proxy服务器，MySQL的Proxy服务器等等。如果你想知道Nginx是怎么写出来的，
如果你想知道Tomcat和Jetty是如何实现的，如果你也想实现一个简单的Redis服务器，那都应该好好理解一下Netty，它们高性能的原理都是类似的

## Netty组成

核心组件包括：

Bootstrap 和 ServerBootstrap 、
Channel、
ChannelHandler、
ChannelPipeline、
EventLoop、
ChannelFuture

#### .Bootstrap，ServerBootstrap

bootstrap意思是引导，一个Netty应用通常由一个Bootstrap开始，主要作用是配置整个netty程序，串联各个组件。ServerBootstrap是服务端启动引导类，Bootstrap是客户端启动引导类。该类提供了一个 用于应用程序网络层配置的容器

Bootstrap: 用于客户端
 示例：Bootstrap bootstrap=new Bootstrap();
ServerBootstrap：用于服务器端
 示例： ServerBootstrap serverBootstrap=new ServerBootstrap();

#### Channel

Channel是Netty网络通信的通道，通过该通道可以执行网络I/O操作。主要作用是：

    维护当前网络连接的通道的状态（例如是否打开?是否已连接）
    网络连接的配置参数（例如接收和发送缓冲区的大小）
    提供异步的网络I/O操作（如建立连接，读写，绑定端口），异步意味着任何I/O操作都会立即返回，可以通过注册一个监听器来自定义操作结果的事件处理。
    支持关联I/O操作与对应的处理程序
    
底层网络传输 API 必须提供给应用 I/O操作的接口，如读，写，连接，绑定等等。对于我们来说，这是结构几乎总是会成为一个“socket”。 Netty 中的接口 Channel 定义了与 socket 丰富交互的操作集：
bind, close, config, connect, isActive, isOpen, isWritable, read, write 等等。 Netty 提供大量的 Channel 实现来专门使用。这些包括 AbstractChannel，AbstractNioByteChannel，AbstractNioChannel，
EmbeddedChannel， LocalServerChannel，NioSocketChannel 等等。

**一些常用的Channel类型：**

NioSocketChannel 

    异步的客户端 TCP Socket 连接。

NioServerSocketChannel

    异步的服务器端 TCP Socket 连接。与客户端的NioSocketChannel对应

NioDatagramChannel 

    异步的 UDP 连接。

NioSctpChannel

    异步的客户端 Sctp 连接。

NioSctpServerChannel，异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及文件 IO。

##### EpollDomainSocketChannel 、EpollServerDomainSocketChannel、EpollSocketChannel、EpollServerSocketChannel、EpollDatagramChannel

##### EmbeddedChannel

##### KQueueDatagramChannel、KQueueDomainSocketChannel、KQueueServerDomainSocketChannel、KQueueServerSocketChannel、KQueueSocketChannel

##### SocketWritableByteChannel

#####    netty的 channel 实现机制
Channel网络层读写的抽象

　　AbstractChannel网络层读写的具体实现

　　　　AbstractNioChannel主要采用selector实现io事件监听

　　　　　　AbstractNioByteChannel 客户端channel的抽象，包含NioByteUnsafe，调用构造方法时传入的注册事件不一致read事件。客户端的读是读取数据

　　　　　　　　NioSocketChannel，包含NioSocketChannelConfig

　　　　　　AbstractNioMessageChannel服务端的抽象，包含NioMessageUnsafe，调用构造方法时传入的注册事件不一致accept事件。服务端的读是读取一条新连接

　　　　　　　　NioServerSocketChannel包含NioServerSocketChannelConfig


#### ChannelHandler

它是一个接口，用于处理I/O事件或拦截I/O事件，并将其转发给对应的channelPipeline中的下一个处理程序。

ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类：

ChannelInboundHandler 用于处理入站 I/O 事件。
ChannelOutboundHandler 用于处理出站 I/O 操作。
或者使用以下适配器类：

ChannelInboundHandlerAdapter 用于处理入站 I/O 事件。
ChannelOutboundHandlerAdapter 用于处理出站 I/O 操作。
ChannelDuplexHandler 用于处理入站和出站事件。

7.ChannelHandlerContext

保存Channel相关的上下文信息，同时关联一个ChannelHandler对象

#### channelPipeline

保存 ChannelHandler 的 List，用于处理或拦截 Channel 的入站事件和出站操作。

在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下：


一个 Channel 包含了一个 ChannelPipeline，而 ChannelPipeline 中又维护了一个由 ChannelHandlerContext 组成的双向链表，并且每个 ChannelHandlerContext 中又关联着一个 ChannelHandler。

入站事件和出站事件在一个双向链表中，入站事件会从链表 head 往后传递到最后一个入站的 handler，出站事件会从链表 tail 往前传递到最前一个出站的 handler，两种类型的 handler 互不干扰。

**Selector**

Netty 基于 Selector 对象实现 I/O 多路复用，通过 Selector 一个线程可以监听多个连接的 Channel 事件。

当向一个 Selector 中注册 Channel 后，Selector 内部的机制就可以自动不断地查询(Select) 这些注册的 Channel 是否有已就绪的 I/O 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地管理多个 Channel 。

#### EventLoop
NioEventLoop

NioEventLoop中维护了一个线程和任务队列，支持异步提交执行任务，线程启动时会调用NioEventLoop的run方法，执行I/O任务和非I/O任务：

I/O任务，即selectionKey中就绪事件，例如read，write，accept，connect等，由processSelectedKeys方法触发。

非I/O任务，添加到taskQueue中的任务，如register()，bind()等任务

5.NioEventLoopGroup

NioEventLoopGroup，主要管理 eventLoop 的生命周期，可以理解为一个线程池，内部维护了一组线程，每个线程(NioEventLoop)负责处理多个 Channel 上的事件，而一个 Channel 只对应于一个线程。