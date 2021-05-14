#netty实现 socket demo
# 概要

近期学习netty，写了个socket简单demo作记录，也供新手参考。

>  
 由于主要记录netty网络协议的使用，demo中并没有socket粘包和分包的处理，有兴趣的读者可以自己写下，理论可以参考笔者文章： netty相关理论可以参考笔者文章： 


# 简介
1. 使用nodejs模拟客户端发送消息。1. netty中使用SocketInHandler和DealWeiHandler分别负责解析和处理业务逻辑。netty大概结构如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040170010.png" alt="在这里插入图片描述"> 3、协议demo如下：（此协议完全可以自定义，此处只是随便的一个demo）
>  
 M_NET_HEADER_STARTurlM_BLANKhttp://www.iqiyi.com?name=hello userM_BLANKphoneM_NET_HEADER_END{id:123123123,data:{name:aaa}} 


主要分为header和json，header以’M_NET_HEADER_START’为开始标识符，以‘M_NET_HEADER_END’为结束标识符，以‘M_BLANK’为间隔标识符。

最终服务器端打印效果如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210040170571.png" alt="在这里插入图片描述">

# 代码

### nodejs客户端

```
var net = require('net');

var HOST = '127.0.0.1';
var PORT = 6969;

var clientOne = new net.Socket();
clientOne.connect(PORT, HOST, function() {

    console.log('CONNECTED TO: ' + HOST + ':' + PORT);

    // 建立连接后立即向服务器发送数据，服务器将收到这些数据
    clientOne.write('M_NET_HEADER_STARTurlM_BLANKhttp://www.iqiyi.com?name=hello userM_BLANKphoneM_NET_HEADER_END{id:123123123,data:{name:aaa}}');
});

// 为客户端添加“data”事件处理函数
// data是服务器发回的数据
clientOne.on('data', function(data) {

    console.log('DATA: ' + data);
    // 完全关闭连接
    clientOne.destroy();

});

// 为客户端添加“close”事件处理函数
clientOne.on('close', function() {
    console.log('Connection closed');
});

```

### testMain.java

```
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import socketTest.DealWithHandler;
import socketTest.SocketInHandler;

import java.util.Date;

public class testMain {

    public static void main(String[] args) {
        // 创建mainReactor
        NioEventLoopGroup boosGroup = new NioEventLoopGroup();
        // 创建工作线程组
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        final ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap
                // 组装NioEventLoopGroup
                .group(boosGroup, workerGroup)
                // 设置channel类型为NIO类型
                .channel(NioServerSocketChannel.class)
                // 设置连接配置参数
                .option(ChannelOption.SO_BACKLOG, 1024)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childOption(ChannelOption.TCP_NODELAY, true)
                // 配置入站、出站事件handler
                .childHandler(new ChannelInitializer&lt;NioSocketChannel&gt;() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) {

                        ch.pipeline().addLast(new SocketInHandler());
                        ch.pipeline().addLast(new DealWithHandler());

                    }

                });

        // 绑定端口
        int port = 6969;
        serverBootstrap.bind(port).addListener(future -&gt; {
            if (future.isSuccess()) {
                System.out.println(new Date() + ": 端口[" + port + "]绑定成功!");
            } else {
                System.err.println("端口[" + port + "]绑定失败!");
            }
        });
    }
}

```

### SocketInHandler.java

```
package socketTest;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

public class SocketInHandler extends ChannelInboundHandlerAdapter {
    private static final String HEADER_START="M_NET_HEADER_START";
    private static final String HEADER_END="M_NET_HEADER_END";
    private static final String HEADER_BLANK="M_BLANK";
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        NetProtocol netProtocol=new NetProtocol();
        //将byte转化为String
        ByteBuf buf=(ByteBuf)msg;
        byte[] bytes=new byte[buf.readableBytes()];
        buf.readBytes(bytes);
        String body=new String(bytes,"UTF-8");
        //解析头部
        String header=body.substring(body.indexOf(HEADER_START)+HEADER_START.length(),body.indexOf(HEADER_END));
        String[] arrayHeader=header.split(" ");
        for(String s :arrayHeader){
            String[] ss=s.split(HEADER_BLANK);
            if("url".equals(ss[0])){
                netProtocol.url=ss[1];
            }else if("user".equals(ss[0])){
                netProtocol.user=ss[1];
            }
        }
        //解析json
        String json=body.substring(body.indexOf(HEADER_END)+HEADER_END.length());
        netProtocol.json=json;
        //抛给下一个ChannelHandler处理
        ctx.fireChannelRead(netProtocol);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
        cause.printStackTrace();
    }
}


```

### NetProtocol.java

```
package socketTest;

public class NetProtocol {
    public String url;
    public String user;
    public String json;

}

```

### DealWithHandler.java

```
package socketTest;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.text.SimpleDateFormat;
import java.util.Date;

public class DealWithHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        NetProtocol netProtocol=(NetProtocol)msg;
        System.out.println("url:"+netProtocol.url);
        System.out.println("user:"+netProtocol.user);
        System.out.println("json:"+netProtocol.json);

        SimpleDateFormat dateFormat=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String time=dateFormat.format(new Date());
        String res="来自与服务端的回应,时间:"+ time;
        ByteBuf resp= Unpooled.copiedBuffer(res.getBytes());
        ctx.write(resp);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
        cause.printStackTrace();
    }
}


```