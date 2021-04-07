#netty 长连接  demo
# 概要

近期接触netty长连接方面，实现简单demo记录，主要实现以下内容：
1. 使用HashMap存储Channel，保证需要的时候可以根据key获取到。1. 被HashMap存储的长连接实现心跳机制。1. demo中是以id为key存储在HashMap中，解析内容用到了Gson。
>  
 为了让demo更容易理解，没有实现粘包分包的逻辑，一旦发生粘包分包demo会有问题。 对粘包和分包有兴趣的可以看下笔者的另一篇文章： 


# 主要代码

代码主要有两部分，netty服务器和nodejs客户端。

## nodejs客户端

建立连接的时候会将自己的id发给服务器。 每次收到服务器的ping，会返回pong以作回应。

```
var net = require('net');

var HOST = '127.0.0.1';
var PORT = 6969;

var clientA = new net.Socket();
clientA.connect(PORT, HOST, function() {

    console.log('CONNECTED TO: ' + HOST + ':' + PORT);

    var content={id:"clientA"};
    clientA.write(JSON.stringify(content));
});

clientA.on('data', function(data) {

    console.log('clientA DATA: ' + data);

    var json=JSON.parse(data);
    if(json.type&amp;&amp;json.type==='ping'){
        var content={id:"clientA",type:"pong"};
        clientA.write(JSON.stringify(content));
    }
});

```

## java服务器

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/2730.png" alt="在这里插入图片描述">

### build.gradle

依赖netty和Gson

```
dependencies {
    compile group: 'io.netty', name: 'netty-all', version: '4.0.4.Final'
    compile group: 'com.google.code.gson', name: 'gson', version: '2.8.5'
}

```

### testMain.java

搭建简单netty服务，在ChannelPipeline加入自定义的ProtocolInHandler。

```
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

                        ch.pipeline().addLast(new ProtocolInHandler());

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

### ProtocolInHandler.java

读取客户端发来的消息，如果在消息中解析出id，那么会根据id找到channel，pong的次数加1。

```
public class ProtocolInHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf=(ByteBuf)msg;
        byte[] bytes=new byte[buf.readableBytes()];
        buf.readBytes(bytes);
        String s=new String(bytes);
        System.out.println(s);


        try {
            JsonObject jsonObject = new JsonParser().parse(s).getAsJsonObject();
            String id=jsonObject.get("id").getAsString();

            if(id!=null) {
                ChannelPool.getInstance().pongChannel(id, ctx.channel());
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```

### ChannelPool.java

使用HashMap来存储Channel。 每个Channel会记录ping和pong的次数，一旦ping的次数大于pong的次数 2次，那么就认为连接断开。

```
public class ChannelPool {
    private static final int HEART_PERIOD=3000;//每隔HEART_PERIOD的时间ping一次
    private static final int BREAK_TIMES=2;//如果BREAK_TIMES次没有ping通，就断开连接

    private static ChannelPool instance=new ChannelPool();
    private Timer timer;
    private HashMap&lt;String, ChannelData&gt; pool;

    private static class ChannelData{
        Channel channel;
        int pingCount;
        int pongCount;

        private ChannelData(Channel channel){
            this.channel=channel;
            this.pingCount=0;
            this.pongCount=0;
        }
    }

    private ChannelPool(){
        pool=new HashMap&lt;&gt;();
        timer=new Timer();
    }

    public static ChannelPool getInstance(){
        return instance;
    }

    public Channel getChannel(String key){
        return pool.get(key).channel;
    }

    public void setChannel(String key,Channel channel){
        ChannelData channelData=new ChannelData(channel);
        pool.put(key,channelData);
        startWithChannel(key,channelData);
    }

    public void pongChannel(String key,Channel channel){
        if(pool.containsKey(key)){
            pool.get(key).pongCount++;
        }else{
            setChannel(key,channel);
        }
    }

    public void startWithChannel(String key,ChannelData channelData){
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                //如果有几次没有ping通，就关闭task，移除channel
                if(channelData.pingCount-channelData.pongCount&gt;=BREAK_TIMES){
                    this.cancel();
                    pool.remove(key);
                }else{
                    channelData.pingCount=0;
                    channelData.pongCount=0;
                }

                //ping 逻辑
                JsonObject jsonObject=new JsonObject();
                jsonObject.addProperty("type","ping");
                jsonObject.addProperty("id",key);

                ByteBuf byteBuf= Unpooled.buffer();
                String json=jsonObject.toString();
                byteBuf.writeBytes(json.getBytes());

                //如果本地的channel已经断开，那就直接回收，这属于服务器端断了
                if(channelData.channel.isActive()) {
                    channelData.channel.writeAndFlush(byteBuf);
                    channelData.pingCount++;
                }else{
                    this.cancel();
                    pool.remove(key);
                }
            }
        },0,HEART_PERIOD);
    }
}


```