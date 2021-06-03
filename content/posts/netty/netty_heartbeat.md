---
title: "Netty实战——基于netty实现心跳检测、消息编解码和断连重连"
tags: ["Netty"]
author: "Jack Wu"
date: 2020-11-26T14:43:47+08:00
draft: false
---
 整理下思路
 
 1、客户端向服务端注册客户端  registerClient(String ClientName, String groupName, EsunClient client)
 
 2、客户端定期向服务端发送心跳消息、
 
 3、服务端接收心跳消息，并更新客户端注册列表的心跳时间
 
 4、服务端定期检查注册列表，对心跳超时未超过三次的的客户端发送心跳消息
 
 5、客户端启动后会自动重连
 
 6、服务端启动后会自动重连
 
 简单实现
 
 client端
 ```
 public class HeartBeatClientHandler extends ChannelInboundHandlerAdapter {
     private long reconnectTime=5L;
     @Autowired
     Client client;
     @Override
     public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
         super.channelRegistered(ctx);
     }
 
     @Override
     public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
         //重连
         ctx.channel().eventLoop().schedule(new Runnable() {
             @Override
             public void run() {
                 try {
                     client.connect();
                 } catch (InterruptedException e) {
                     log.error("连接服务端出现异常，message:{}",e.getMessage());
                 }
             }
         },reconnectTime, TimeUnit.SECONDS);
         super.channelUnregistered(ctx);
     }
 
     @Override
     public void channelActive(ChannelHandlerContext ctx) throws Exception {
         CustomMessageProto.CustomMessage.Builder heartBeatMsg=CustomMessageProto.CustomMessage.newBuilder()
                 .setHeader(
                         CustomMessageProto.CustomMessage.CustomHeader.newBuilder()
                         .setTypeValue(0xABEF)
                         .setType(CustomMessageProto.CustomMessage.CustomHeader.MessgeType.PING)
                 );
         ctx.channel().writeAndFlush(heartBeatMsg);
     }
 
     @Override
     public void channelInactive(ChannelHandlerContext ctx) throws Exception {
         log.warn("连接断开，channelInactive() call ");
         super.channelInactive(ctx);
     }
 
     @Override
     public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
         CustomMessageProto.CustomMessage message= (CustomMessageProto.CustomMessage) msg;
         if (message.getHeader().getType().equals(CustomMessageProto.CustomMessage.CustomHeader.MessgeType.PONG)){
             log.info("接收到心跳消息：{}",message);
         }
         super.channelRead(ctx, msg);
     }
 
     @Override
     public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
         super.channelReadComplete(ctx);
     }
 
     @Override
     public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
         if (evt instanceof IdleStateEvent){
             log.info("心跳检测触发了");
             IdleStateEvent idleStateEvent= (IdleStateEvent) evt;
             if (idleStateEvent.state().equals(IdleState.WRITER_IDLE)){
                 log.warn("已经5秒钟没有写操作了，发个心跳消息检测下");
                 CustomMessageProto.CustomMessage.Builder heartBeatReqMsg=CustomMessageProto.CustomMessage.newBuilder()
                         .setHeader(
                                 CustomMessageProto.CustomMessage.CustomHeader.newBuilder()
                                 .setTypeValue(0xABEF)
                                 .setType(CustomMessageProto.CustomMessage.CustomHeader.MessgeType.PING)
                         );
                 ctx.channel().writeAndFlush(heartBeatReqMsg);
             }
         }
     }
 
     @Override
     public void channelWritabilityChanged(ChannelHandlerContext ctx) throws Exception {
         super.channelWritabilityChanged(ctx);
     }
 
     @Override
     public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
         log.error("程序异常，{}",cause.getMessage());
         ctx.channel().close();
     }
 }
 ```
 ```
 public class CustomClientChannelInitializer extends ChannelInitializer<SocketChannel> {
     @Autowired
     HeartBeatClientHandler heartBeatClientHandler;
     @Override
     protected void initChannel(SocketChannel socketChannel) throws Exception {
         socketChannel
                 .pipeline()
                 .addLast(new IdleStateHandler(0,5,0, TimeUnit.SECONDS))
                 .addLast(new ProtobufVarint32FrameDecoder())
                 .addLast(new ProtobufDecoder(CustomMessageProto.CustomMessage.getDefaultInstance()))
                 .addLast(new ProtobufVarint32LengthFieldPrepender())
                 .addLast(new ProtobufEncoder())
                 .addLast(heartBeatClientHandler);
     }
 }
 ```
 
server端

```
public class HeartBeatServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        super.channelRegistered(ctx);
    }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        super.channelUnregistered(ctx);
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        super.channelActive(ctx);
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        super.channelInactive(ctx);
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        CustomMessageProto.CustomMessage message= (CustomMessageProto.CustomMessage) msg;
        if (message.getHeader().getType().getNumber()== CustomMessageProto.CustomMessage.CustomHeader.MessgeType.PING_VALUE){
            log.info("接收心跳消息：{}",message);
            CustomMessageProto.CustomMessage.Builder heartBeatRespMsg=CustomMessageProto.CustomMessage.newBuilder()
                    .setHeader(
                            CustomMessageProto.CustomMessage.CustomHeader.newBuilder()
                            .setTypeValue(0xABEF)
                            .setType(CustomMessageProto.CustomMessage.CustomHeader.MessgeType.PONG)
                    );
            ctx.channel().writeAndFlush(heartBeatRespMsg);
        }else {
            log.warn("消息未处理：{}",message);
        }
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        super.channelReadComplete(ctx);
    }

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        //超过40s没接收心跳消息则强制断开连接，从列表移除
        if (evt instanceof IdleStateEvent){
            IdleStateEvent event= (IdleStateEvent) evt;
            if (event.state().equals(IdleState.READER_IDLE)){
                //判断上一次心跳时间，如果超过40s则直接断开连接
                //否则尝试发送心跳
                log.info("已经5秒没有收到客户端心跳消息了，发送心跳检测给客户端");
                CustomMessageProto.CustomMessage.Builder heartBeatRespMsg=CustomMessageProto.CustomMessage.newBuilder()
                        .setHeader(
                                CustomMessageProto.CustomMessage.CustomHeader.newBuilder()
                                .setTypeValue(0xABEF)
                                .setType(CustomMessageProto.CustomMessage.CustomHeader.MessgeType.PONG)
                        );
                ctx.channel().writeAndFlush(heartBeatRespMsg);
            }
        }
        super.userEventTriggered(ctx, evt);
    }

    @Override
    public void channelWritabilityChanged(ChannelHandlerContext ctx) throws Exception {
        super.channelWritabilityChanged(ctx);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        log.error("发生异常，message：{}",cause.getMessage());
        super.exceptionCaught(ctx, cause);
    }
}
```
```
public class CustomServerChannelInitializer extends ChannelInitializer<SocketChannel> {
    @Autowired
    HeartBeatServerHandler heartBeatServerHandler;
    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {
        socketChannel.pipeline()
                .addLast(new IdleStateHandler(5,0,0, TimeUnit.SECONDS))
                .addLast(new ProtobufVarint32FrameDecoder())
                .addLast(new ProtobufDecoder(CustomMessageProto.CustomMessage.getDefaultInstance()))
                .addLast(new ProtobufVarint32LengthFieldPrepender())
                .addLast(new ProtobufEncoder())
                .addLast(heartBeatServerHandler);
    }
}
```
