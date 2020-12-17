---
title: "Netty进阶——基于Netty协议栈进行Protobuf私有协议的定制开发 "
tags: ["Netty"]
author: "Jack Wu"
date: 2020-11-24T14:40:15+08:00
draft: false
---


## 私有协议栈和Netty协议栈

绝大多数数私有协议的传输层是基于TCP/IP，利用Netty的TCP/IP协议栈可以非常方便的进行私有协议栈的定制开发

Netty协议栈用于内部各模块间的通信，基于TCP/IP协议，是个类HTTP协议的应用层协议栈，比传统的标准协议栈更轻巧灵活和实用


Netty协议栈承载了业务内部各模块之间的消息交互和服务调用，它的主要功能如下。
(1）基于Netty 的NIO通信框架，提供高性能的异步通信能力;
(2）提供消息的编解码框架，可以实现POJO的序列化和反序列化;
(3)提供基于IP地址的白名单接入认证机制;
(4）链路的有效性校验机制;
(5)链路的断连重连机制。

### Netty协议 通信模型与步骤
考虑到安全，链路建立需要通过基于IP地址或者号段的黑白名单安全认证机制
为样例，本协议使用基于IP地址的安全认证，如果有多个IP，通过逗号进行分割。在实际商用项目中，安全认证机制会更加严格，例如通过密钥对用户名和密码进行安全认证。

### 消息定义

#### header
   crcCode : int 32 位 netty消息的校验码，由三部分组成 ： 1、0xABEF  （固定值，表面是netty协议，2个字节） ；2、主版本号 1~255，1个字节；3、次版本号，1~255，1个字节。也就是： crcCode= 0xABEF + 主版本号 + 次版本号
   
   length : int 32
   
   sessionID : long 64  节点内全局唯一，由会话ID生成器生成
   
   type : Byte 8 
   
        0:业务请求消息
        1:业务响应消息
        2:业务ONE WAY消息（既是请求又是响应消息)
        3:握手请求消息
        4:握手应答消息
        5:心跳请求消息
        6:心跳应答消息
        
   
   priority: Byte 8 消息优先级 0~255
   
   attachment : Map<String , Object>  可选字段，用于扩展请求头
   
    
#### body  Object   消息体，
对于请求消息，是参数数据，对于相应消息，是返回的数据

### 链路的建立与关闭
#### 链路建立
考虑到安全，链路建立需要通过基于IP地址或者号段的黑白名单安全认证机制
为样例，本协议使用基于IP地址的安全认证，如果有多个IP，通过逗号进行分割。在实际商用项目中，安全认证机制会更加严格，例如通过密钥对用户名和密码进行安全认证。

客户端与服务端链路建立成功之后，由客户端发送握手请求消息，

握手请求消息的定义如下。

(1）消息头的type字段值为3;
(2）可选附件为个数为0;
(3)消息体为空;
(4)握手消息的长度为22个字节。
服务端接收到客户端的握手请求消息之后，如果IP校验通过，返回握手成功应答消息给客户端，应用层链路建立成功。

握手应答消息定义如下。

(1）消息头的type字段值为4;
(2)可选附件个数为0;
(3）消息体为 byte类型的结果，“0”表示认证成功;“-1”表示认证失败。链路建立成功之后，客户端和服务端就可以互相发送业务消息了。

#### 链路关闭
由于采用长连接通信，在正常的业务运行期间，双方通过心跳和业务消息维持链路，任何一方都不需要主动关闭连接。
但是，在以下情况下，客户端和服务端需要关闭连接。
(1）当对方宕机或者重启时，会主动关闭链路，另一方读取到操作系统的通知信号，得知对方REST 链路，需要关闭连接，释放自身的句柄等资源。由于采用TCP全双工通信，通信双方都需要关闭连接，释放资源;
(2）消息读写过程中，发生了1/O异常，需要主动关闭连接;
( 3）心跳消息读写过程中发生了IO异常，需要主动关闭连接;
(4）心跳超时，需要主动关闭连接;
(5）发生编码异常等不可恢复错误时，需要主动关闭连接。
### 可靠性设计
心跳检测机制

重连机制

重复登录保护

消息缓存重发

### 安全性设计
为了保证整个集群环境的安全，内部长连接采用基于IP地址的安全认证机制，服务端对握手请求消息的IP地址进行合法性校验:如果在白名单之内，则校验通过;否则,拒绝对方连接。

如果将Netty协议栈放到公网中使用，需要采用更加严格的安全认证机制，例如基于密钥和AES 加密的用户名+密码认证机制，也可以采用SSL/TSL安全传输。

作为示例程序，Netty协议栈采用最简单的基于IP地址的白名单安全认证机制。
### 可扩展性设计
Netty协议需要具备一定的扩展能力，业务可以在消息头中自定义业务域字段，例如消息流水号、业务自定义消息头等。通过Netty消息头中的可选附件 attachment字段,
务可以方便地进行自定义扩展。
Netty 协议栈架构需要具备一定的扩展能力，例如统一的消息拦截、接口日志、安全、加解密等可以被方便地添加和删除，不需要修改之前的逻辑代码，类似Servlet 的FilterChain和AOP，但考虑到性能因素，不推荐通过AOP来实现功能的扩展。

## 自定义消息
```
syntax="proto3";
option java_package="fun.codenow.netty.privateprotocol.protobuf";
option java_outer_classname="CustomMessageProto";
import "google/protobuf/any.proto";

message CustomMessage{
	CustomHeader header=1;
	google.protobuf.Any body=2;
	message CustomHeader{
		int32 crcCode=1;
		int32 length=2;
		int64 sessionId=3;
		enum MessgeType{
			SERVICE_REQ = 0;  
		SERVICE_RESP = 1;  
		ONE_WAY = 2;  
		LOGIN_REQ = 3; 
	    LOGIN_RESP = 4; 
	    PING =  5; 
	    PONG =  6; 
		}
		MessgeType type=4;
		int32 priority=5;
		map<string,google.protobuf.Any>  attachment=6;
	}
}
```
Protobuf编码
```
public class ProtobufDecoder extends MessageToMessageDecoder<ByteBuf> {
    private static final boolean HAS_PARSER;
    private final MessageLite prototype;
    private final ExtensionRegistryLite extensionRegistry;

    public ProtobufDecoder(MessageLite prototype) {
        this(prototype, (ExtensionRegistry)null);
    }

    public ProtobufDecoder(MessageLite prototype, ExtensionRegistry extensionRegistry) {
        this(prototype, (ExtensionRegistryLite)extensionRegistry);
    }

    public ProtobufDecoder(MessageLite prototype, ExtensionRegistryLite extensionRegistry) {
        this.prototype = ((MessageLite)ObjectUtil.checkNotNull(prototype, "prototype")).getDefaultInstanceForType();
        this.extensionRegistry = extensionRegistry;
    }

    protected void decode(ChannelHandlerContext ctx, ByteBuf msg, List<Object> out) throws Exception {
        int length = msg.readableBytes();
        byte[] array;
        int offset;
        if (msg.hasArray()) {
            array = msg.array();
            offset = msg.arrayOffset() + msg.readerIndex();
        } else {
            array = ByteBufUtil.getBytes(msg, msg.readerIndex(), length, false);
            offset = 0;
        }

        if (this.extensionRegistry == null) {
            if (HAS_PARSER) {
                out.add(this.prototype.getParserForType().parseFrom(array, offset, length));
            } else {
                out.add(this.prototype.newBuilderForType().mergeFrom(array, offset, length).build());
            }
        } else if (HAS_PARSER) {
            out.add(this.prototype.getParserForType().parseFrom(array, offset, length, this.extensionRegistry));
        } else {
            out.add(this.prototype.newBuilderForType().mergeFrom(array, offset, length, this.extensionRegistry).build());
        }

    }

    static {
        boolean hasParser = false;

        try {
            MessageLite.class.getDeclaredMethod("getParserForType");
            hasParser = true;
        } catch (Throwable var2) {
        }

        HAS_PARSER = hasParser;
    }
}
```
解码
```
public class ProtobufDecoder extends MessageToMessageDecoder<ByteBuf> {
    private static final boolean HAS_PARSER;
    private final MessageLite prototype;
    private final ExtensionRegistryLite extensionRegistry;

    public ProtobufDecoder(MessageLite prototype) {
        this(prototype, (ExtensionRegistry)null);
    }

    public ProtobufDecoder(MessageLite prototype, ExtensionRegistry extensionRegistry) {
        this(prototype, (ExtensionRegistryLite)extensionRegistry);
    }

    public ProtobufDecoder(MessageLite prototype, ExtensionRegistryLite extensionRegistry) {
        this.prototype = ((MessageLite)ObjectUtil.checkNotNull(prototype, "prototype")).getDefaultInstanceForType();
        this.extensionRegistry = extensionRegistry;
    }

    protected void decode(ChannelHandlerContext ctx, ByteBuf msg, List<Object> out) throws Exception {
        int length = msg.readableBytes();
        byte[] array;
        int offset;
        if (msg.hasArray()) {
            array = msg.array();
            offset = msg.arrayOffset() + msg.readerIndex();
        } else {
            array = ByteBufUtil.getBytes(msg, msg.readerIndex(), length, false);
            offset = 0;
        }

        if (this.extensionRegistry == null) {
            if (HAS_PARSER) {
                out.add(this.prototype.getParserForType().parseFrom(array, offset, length));
            } else {
                out.add(this.prototype.newBuilderForType().mergeFrom(array, offset, length).build());
            }
        } else if (HAS_PARSER) {
            out.add(this.prototype.getParserForType().parseFrom(array, offset, length, this.extensionRegistry));
        } else {
            out.add(this.prototype.newBuilderForType().mergeFrom(array, offset, length, this.extensionRegistry).build());
        }

    }

    static {
        boolean hasParser = false;

        try {
            MessageLite.class.getDeclaredMethod("getParserForType");
            hasParser = true;
        } catch (Throwable var2) {
        }

        HAS_PARSER = hasParser;
    }
}
```
粘包拆包问题
```
public class ProtobufVarint32FrameDecoder extends ByteToMessageDecoder {
    public ProtobufVarint32FrameDecoder() {
    }

    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        in.markReaderIndex();
        int preIndex = in.readerIndex();
        int length = readRawVarint32(in);
        if (preIndex != in.readerIndex()) {
            if (length < 0) {
                throw new CorruptedFrameException("negative length: " + length);
            } else {
                if (in.readableBytes() < length) {
                    in.resetReaderIndex();
                } else {
                    out.add(in.readRetainedSlice(length));
                }

            }
        }
    }

    private static int readRawVarint32(ByteBuf buffer) {
        if (!buffer.isReadable()) {
            return 0;
        } else {
            buffer.markReaderIndex();
            byte tmp = buffer.readByte();
            if (tmp >= 0) {
                return tmp;
            } else {
                int result = tmp & 127;
                if (!buffer.isReadable()) {
                    buffer.resetReaderIndex();
                    return 0;
                } else {
                    if ((tmp = buffer.readByte()) >= 0) {
                        result |= tmp << 7;
                    } else {
                        result |= (tmp & 127) << 7;
                        if (!buffer.isReadable()) {
                            buffer.resetReaderIndex();
                            return 0;
                        }

                        if ((tmp = buffer.readByte()) >= 0) {
                            result |= tmp << 14;
                        } else {
                            result |= (tmp & 127) << 14;
                            if (!buffer.isReadable()) {
                                buffer.resetReaderIndex();
                                return 0;
                            }

                            if ((tmp = buffer.readByte()) >= 0) {
                                result |= tmp << 21;
                            } else {
                                result |= (tmp & 127) << 21;
                                if (!buffer.isReadable()) {
                                    buffer.resetReaderIndex();
                                    return 0;
                                }

                                result |= (tmp = buffer.readByte()) << 28;
                                if (tmp < 0) {
                                    throw new CorruptedFrameException("malformed varint.");
                                }
                            }
                        }
                    }

                    return result;
                }
            }
        }
    }
}

```

```
public class ProtobufVarint32LengthFieldPrepender extends MessageToByteEncoder<ByteBuf> {
    public ProtobufVarint32LengthFieldPrepender() {
    }

    protected void encode(ChannelHandlerContext ctx, ByteBuf msg, ByteBuf out) throws Exception {
        int bodyLen = msg.readableBytes();
        int headerLen = computeRawVarint32Size(bodyLen);
        out.ensureWritable(headerLen + bodyLen);
        writeRawVarint32(out, bodyLen);
        out.writeBytes(msg, msg.readerIndex(), bodyLen);
    }

    static void writeRawVarint32(ByteBuf out, int value) {
        while((value & -128) != 0) {
            out.writeByte(value & 127 | 128);
            value >>>= 7;
        }

        out.writeByte(value);
    }

    static int computeRawVarint32Size(int value) {
        if ((value & -128) == 0) {
            return 1;
        } else if ((value & -16384) == 0) {
            return 2;
        } else if ((value & -2097152) == 0) {
            return 3;
        } else {
            return (value & -268435456) == 0 ? 4 : 5;
        }
    }
}
```