---
title: Netty Epoll 和 Nio
categories:
  - Java
tags:
  - Netty
abbrlink: a8f7cc4d
date: 2018-07-13 17:28:36
---



# 入门

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.ssl.SslContext;
import io.netty.handler.ssl.SslContextBuilder;
import io.netty.handler.ssl.util.SelfSignedCertificate;
import io.netty.util.concurrent.DefaultEventExecutorGroup;
import io.netty.util.concurrent.EventExecutorGroup;

/**
 * Netty 启动器，初始化 spring boot 的时候会自动加载
 */
public class Server {
    private static final Logger LOG = LoggerFactory.getLogger(Server.class);

    /*
     * NioEventLoopGroup 默认创建数量为 CPU * 2，类型为 NioEventLoop 的实例。
     * 每个 NioEventLoop 实例都持有一个线程，以及一个类型为 LinkedBlockingQueue 的任务队列
     */
    EventLoopGroup bossGroup = new NioEventLoopGroup();
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    // 业务线程
    EventExecutorGroup eventExecutorGroup = new DefaultEventExecutorGroup(16);
    static final boolean isSsl = System.getProperty("ssl") != null;

    /**
     * 初始化 Netty
     */
    public void startNetty() {
        try {
            final SslContext sslCtx;
            if (isSsl) {
                SelfSignedCertificate ssc = new SelfSignedCertificate();
                sslCtx = SslContextBuilder.forServer(ssc.certificate(), ssc.privateKey()).build();
            } else {
                sslCtx = null;
            }
            // 引导辅助程序
            ServerBootstrap bs = new ServerBootstrap();
            // 通过NIO方式来接收连接和处理连接
            bs.group(bossGroup, workerGroup);
            // 设置NIO的Channel
            bs.channel(NioServerSocketChannel.class);
            bs.childHandler(new HttpChannelInitializer(sslCtx, eventExecutorGroup));

            bs.option(ChannelOption.SO_BACKLOG, 1024); // 连接数
            // bs.childOption(ChannelOption.SO_KEEPALIVE, true); // 是否长链接
            bs.childOption(ChannelOption.TCP_NODELAY, true); // 是否TCP延迟

            // 配置完成，开始绑定server，通过调用sync同步方法阻塞直到绑定成功
            Channel ch = bs.bind(9999).sync().channel();
            LOG.info("Server startup. listen on {}-{}", (isSsl ? "https" : "http"), 9999);
            // 应用程序会一直等待，直到Channel关闭
            ch.closeFuture().sync();
        } catch (Exception e) {
            LOG.error("server startup failed. {}", e.getMessage(), e);
        } finally {
            LOG.info("释放资源...");
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
            eventExecutorGroup.shutdownGracefully();
            bossGroup = null;
            workerGroup = null;
            eventExecutorGroup = null;
        }
    }

    public static void main(String[] args) {
        new Server().startNetty();
    }
}

```



<!-- more -->



# 优化：Epoll 和 Nio

```java
private static final boolean isEpoll = Epoll.isAvailable();
private EventLoopGroup bossGroup = isEpoll ? new EpollEventLoopGroup() : new 					NioEventLoopGroup();
private EventLoopGroup workerGroup = isEpoll ? new EpollEventLoopGroup() : new 				NioEventLoopGroup();
ServerBootstrap bootstrap = new ServerBootstrap();
bootstrap.group(bossGroup, workerGroup)
         .channel(isEpoll ? EpollServerSocketChannel.class : NioServerSocketChannel.class);
```

