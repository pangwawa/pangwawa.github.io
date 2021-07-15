---
title: "Netty基础——从简单Echo程序分析Netty组件"
categories: ["技术博客"]
tags: ["Netty"]
author: "Jack Wu"
date: 2020-11-26T14:43:47+08:00
draft: false
---

Bootstrap or ServerBootstrap

EventLoop

EventLoopGroup

ChannelPipeline

Channel

Fture or ChannelFuture

ChannelInitializer

ChannelHandler

详解：

Bootstrap，一个Netty应用通常由一个Bootstrap开始，它主要作用是配置整个Netty程序，串联起各个组件。

Handler，为了支持各种协议和处理数据的方式，便诞生了Handler组件。Handler主要用来处理各种事件，这里的事件很广泛，比如可以是连接、数据接收、异常、数据转换等。

ChannelInboundHandler，一个最常用的Handler。这个Handler的作用就是处理接收到数据时的事件，也就是说，我们的业务逻辑一般就是写在这个Handler里面的，ChannelInboundHandler就是用来处理我们的核心业务逻辑。

ChannelInitializer，当一个链接建立时，我们需要知道怎么来接收或者发送数据，当然，我们有各种各样的Handler实现来处理它，那么ChannelInitializer便是用来配置这些Handler，它会提供一ChannelPipeline，并把Handler加入到ChannelPipeline。ChannelPipeline，一个Netty应用基于ChannelPipeline机制，这种机制需要依赖于EventLoop和EventLoopGroup，因为它们三个都和事件或者事件处理相关。 ChannelPipeline负责安排Handler的顺序及其执行EventLoops的目的是为Channel处理IO操作，一个EventLoop可以为多个Channel服务。EventLoopGroup会包含多个EventLoop。

Channel代表了一个Socket链接，或者其它和IO操作相关的组件，它和EventLoop一起用来参与IO处理。

Future，在Netty中所有的IO操作都是异步的，因此，你不能立刻得知消息是否被正确处理，但是我们可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过Future和ChannelFutures,他们可以注册一个监听，当操作执行成功或失败时监听会自动触发。总之，所有的操作都会返回一个ChannelFuture。

我们来看看如何配置一个Netty应用？
– BootsStrapping我们利用BootsStrapping来配置netty  应用，它有两种类型，一种用于Client端：BootsStrap，另一种用于Server端：ServerBootstrap，要想区别如何使用它们，你仅需要记住一个用在Client端，一个用在Server端。下面我们来详细介绍一下这两种类型的区别：1.第一个最明显的区别是，ServerBootstrap用于Server端，通过调用bind()方法来绑定到一个端口监听连接；Bootstrap用于Client端，需要调用connect()方法来连接服务器端，但我们也可以通过调用bind()方法返回的ChannelFuture中获取Channel去connect服务器端。

###### 一个Netty 简单Echo服务端程序实例
```
public class EchoServer {
    public static void main(String[] args) throws InterruptedException {
        /**
         * Bootstrap是应用程序的开始，作用是配置整个netty程序，串联各个组件
         */
        ServerBootstrap serverBootstrap=new ServerBootstrap();
        NioEventLoopGroup nioEventLoopGroup=new NioEventLoopGroup();
        serverBootstrap
                .group(nioEventLoopGroup)
                /**
                 *  Channel  代表一个Socket连接，或者其他和IO操作相关的组件，它和EventLoop一起参加IO处理
                 */
                .channel(NioServerSocketChannel.class)
                .localAddress(new InetSocketAddress(9999))
                /**
                 * Handler 是为了支持各种协议和处理数据的方式; 主要是处理连接、数据接收、异常、数据转换等事件
                 *
                 * ChannelInitializer 用于配置Handler， 它提供ChannelPipeline,并把设置的Handler加到ChannelPipeline
                 */
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        /**
                         * ChannelPipeline  一个Netty应用基于ChannelPipeline机制，这种机制依赖于EventLoop和EventLoopGroup 
                         */
                        socketChannel.pipeline().addLast(new EchoServerHandler());
                    }
                });
        try {
            /**
             *  Future和ChannelFuture ，注册监听，当操作成功或失败时自动触发，所有的操作都会返回一个ChannelFuture
             */
            ChannelFuture channelFuture= serverBootstrap.bind().sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            /**
             * 关闭连接
             */
            nioEventLoopGroup.shutdownGracefully().sync();
        }
    }
}
```

##### 一个Netty简单Echo客户端程序实例
```
public class EchoClient {
    public static void main(String[] args) throws InterruptedException {
        NioEventLoopGroup nioEventLoopGroup=new NioEventLoopGroup();
        Bootstrap bootstrap=new Bootstrap();
        bootstrap
                .group(nioEventLoopGroup)
                .channel(NioSocketChannel.class)
                /**
                 * 与服务端的localAddress不同，这里是remoteAddress，配置远程连接 服务器地址和端口
                 */
                .remoteAddress(new InetSocketAddress(9999))
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        /**
                         * 这里配置的Handler是客户端需要使用的Handler
                         */
                        socketChannel.pipeline().addLast(new EchoClientHandler());
                    }
                });
        try {
            ChannelFuture channelFuture=bootstrap.connect().sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            nioEventLoopGroup.shutdownGracefully().sync();
        }

    }
}
```