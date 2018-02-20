# Netty

> Netty is \_an asynchronous event-driven network application framework \_for rapid development of maintainable high performance protocol servers & clients. -- [https://netty.io](https://netty.io/index.html)

### Hello world

Lets create a simple HTTP server using Netty.

Fist lets get the dependencies using Gradle.

```
plugins {
    id 'java'
    id 'application'
}

ext {
    vertxVersion = '3.5.0'
}

repositories {
    mavenLocal()
    jcenter()
}

version = '1.0.0-SNAPSHOT'
sourceCompatibility = '1.8'

dependencies {
    compile 'io.netty:netty-all:5.0.0.Alpha2'
}

mainClassName = 'com.example.demo.NettyApp'

task wrapper(type: Wrapper) {
    gradleVersion = '4.0'
}
```

Now lets create code needed to handle request using Netty.

```
package com.example.demo;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.http.*;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import io.netty.util.CharsetUtil;

public class NettyApp {

    public static void main(String[] args) {
        EventLoopGroup eventLoopGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap bootstrap = new ServerBootstrap()
                .group(eventLoopGroup)
                .handler(new LoggingHandler(LogLevel.INFO))
                .childHandler(new HttpServerInitializer())
                .channel(NioServerSocketChannel.class);

            Channel ch = bootstrap.bind(8888).sync().channel();
            ch.closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            eventLoopGroup.shutdownGracefully();
        }
    }
}

class HttpServerInitializer extends ChannelInitializer<Channel> {

    @Override
    protected void initChannel(Channel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();

        pipeline.addLast(new HttpServerCodec());
        pipeline.addLast(new HttpServerHandler());
    }
}

class HttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    protected void messageReceived(ChannelHandlerContext ctx, HttpObject msg) throws Exception {
        if (msg instanceof LastHttpContent) {
            ByteBuf content = Unpooled.copiedBuffer("Hello World", CharsetUtil.UTF_8);
            FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, content);
            response.headers().set("Content-Type", "text/plain");
            response.headers().set("Content-Length", String.valueOf(content.readableBytes()));
            ctx.write(response);
        }
    }
}
```

> Now we can compare code required to start HTTP server using Netty and "couple" of lines required by Vert.x.

When we run the server and call `curl localhost:8888` then we will see these logs. 

```
gradle run

> Task :run
Feb 19, 2018 10:14:04 PM io.netty.handler.logging.LoggingHandler channelRegistered
INFO: [id: 0xb0f46b9e] REGISTERED
Feb 19, 2018 10:14:04 PM io.netty.handler.logging.LoggingHandler bind
INFO: [id: 0xb0f46b9e] BIND: 0.0.0.0/0.0.0.0:8888
Feb 19, 2018 10:14:04 PM io.netty.handler.logging.LoggingHandler channelActive
INFO: [id: 0xb0f46b9e, /0:0:0:0:0:0:0:0:8888] ACTIVE
Feb 19, 2018 10:15:03 PM io.netty.handler.logging.LoggingHandler channelRead
INFO: [id: 0xb0f46b9e, /0:0:0:0:0:0:0:0:8888] RECEIVED: [id: 0x77ab38e7, /0:0:0:0:0:0:0:1:52239 => /0:0:0:0:0:0:0:1:8888]
```

The curl returns `Hello World` message we have implemented in `HttpServerHandler`.

```
$ curl localhost:8888
Hello World
```



