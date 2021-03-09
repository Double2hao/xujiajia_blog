#HTTP与HTTPS总结
# 前言

很久之前在小组做过HTTPS相关的分享，后续这方面的知识也是用到比较多，总是会去翻以前的文档，于是便打算将之前的分享总结成博客，当做记录。 大概内容如下：
1. HTTP通信过程1. HTTPS（包括对称和非对称加密）1. 中间人攻击1. HTTP各个版本改动（包括长连接和多路复用）
# HTTP通信过程

<img src="https://img-blog.csdnimg.cn/20181124102412319.png" alt="在这里插入图片描述">
1. 首先客户机与服务器需要建立连接。只要单击某个超级链接，HTTP的工作开始。1. 建立连接后，客户机发送一个请求给服务器，请求方式的格式为：统一资源标识符（URL）、协议版本号，后边是MIME信息包括请求修饰符、客户机信息和可能的内容。建立连接后，客户机发送一个请求给服务器，请求方式的格式为：统一资源标识符（URL）、协议版本号，后边是MIME信息包括请求修饰符、客户机信息和可能的内容。1. 服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是MIME信息包括服务器信息、实体信息和可能的内容。服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是MIME信息包括服务器信息、实体信息和可能的内容。1. 客户端接收服务器所返回的信息通过浏览器显示在用户的显示屏上，然后客户机与服务器断开连接。客户端接收服务器所返回的信息通过浏览器显示在用户的显示屏上，然后客户机与服务器断开连接。
其主要原理就是TCP的三次握手和四次挥手，如下图： <img src="https://img-blog.csdnimg.cn/20181124102615969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> <img src="https://img-blog.csdnimg.cn/20181124102559199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# HTTPS

由于HTTP是明文发送，所以不安全，因此有了更安全的HTTPS。而要理解HTTPS的原理，首先要理解对称和非对称加密（公钥加密）。

>  
 知识注意点 
 - 公钥加密使发送方和接收方无密钥的保密通信成为可能。它是目前网络电商、支付等的理论基础。（假设网络已经不安全了，被监听了，如何通信？在没有公钥加密的时候是不行的）- 公钥加密的速度比对称加密慢100倍，消息大的时候非常耗时。- 公钥加密和对称加密是互相补充，而不是替代。 


# 对称加密

对称加密很好理解，直接看下面例子就可以：

```
明文：I LOVE YOU
密文：K NQXG AQW
(H I J K L M N O P Q R S T U V W)
加密（解密）算法：移位
加密（解密）密钥：2（-2）   

```

对称加密由于加密和解密密钥关联性很大，所以一般认为只有一个密钥。

# 非对称加密（公钥加密）

```
知识点：
公钥加密算法和公钥公开，私钥保密。

公钥：所有人都可以获得任何人的公钥
私钥：自己的私钥只有自己有

公钥加密的数据可以用私钥解密
私钥加密的数据可以用公钥解密

```

例子如下图：

发送给Bob数据过程： 1、获得Bob的公钥，用公钥加密数据。 2、Bob接收到数据后用自己的私钥解密。 <img src="https://img-blog.csdnimg.cn/20181124104005450.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# 公钥加密原理

<img src="https://img-blog.csdnimg.cn/20181124104345881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> <img src="https://img-blog.csdnimg.cn/20181124104334869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

## 例子

```
p=17 q=11
N=q*p=17*11=187
φ(N)=(p-1)(q-1)=16*10=160

gcd(e,160)=1        e=7
d*e mod 160=1 并且d&lt; 160
d=23    因为23*7=161=160+1

公钥：{7,187}
私钥：{23,187 }

设明文M=88,密文为C；
C= 88^7 mod 187=11
M=11^23 mod 187=88

```

>  
 实际p、q取值在10<sup>100和10</sup>200之间 求这样的数的因子称为大整数分解，不可解。 


# 数字签名

公钥既然任何人可以获得，那么甲乙两方通信，乙方如果确认甲方的身份？ <img src="https://img-blog.csdnimg.cn/20181124104641122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

# 消息认证（不加密或者对称加密）

使用场景： 这个可以用在电视广播之类，发送方的内容是可以公开的，但是接收方一定要确定发送方的身份，并且要确定内容没有被修改过。

如果密文被破解并且被修改，接收方如何知道数据被修改过？
1. 发送方对明文做哈希1. 发送方用私钥对哈希加密1. 发送发把加密后的哈希与明文一起发送给接收方 (此处也可以把明文与加密后的哈希再做一次对称加密，这样接收方收到后还要再做一次解密)1. 接收方拿到数据后，把哈希取出，用接收方的公钥解密。然后把明文做哈希，两个哈希对比，如果结果一样，说明内容没有被修改过，同时也确认了发送方的身份。 <img src="https://img-blog.csdnimg.cn/20181124104949752.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">
>  
 为什么对哈希公钥加密而不是直接对明文加密？ 因为哈希短啊，明文是可能很大的，公钥加密本来就慢，如果明文很大就要加密很久。 


# HTTPS过程

下图是网上很经典的例子，生成对话密钥过程：
1. 爱丽丝给出协议版本号、一个客户端生成的随机数，以及客户端支持的加密方法。1. 鲍勃确认双方使用的加密方法，并给出数字证书、以及一个服务器生成的随机数。1. 爱丽丝确认数字证书有效，然后生成一个新的随机数并使用数字证书中的公钥，加密这个随机数，发给鲍勃。1. 鲍勃使用自己的私钥，获取爱丽丝发来的随机数。1. 爱丽丝和鲍勃根据约定的加密方法，使用前面的三个随机数，生成"对话密钥"，用来加密接下来的整个对话过程。 <img src="https://img-blog.csdnimg.cn/20181124105925410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">
简单过程：
1. 双方通过公钥加密交换三个随机数。1. 双方用三个随机数计算出“对称秘钥”。1. 双方采用对称加密进行加密通信。
# 中间人攻击

## SSL证书欺骗攻击

首先通过ARP欺骗、DNS劫持甚至网关劫持等等，将客户端的访问重定向到攻击者的机器，让客户端机器与攻击者机器建立HTTPS连接（使用伪造证书），而攻击者机器再跟服务端连接。 这样用户在客户端看到的是相同域名的网站，但浏览器会提示证书不可信，用户不点击继续浏览就能避免被劫持的。所以这是最简单的攻击方式，也是最容易识别的攻击方式。 <img src="https://img-blog.csdnimg.cn/20181124110221963.png" alt="在这里插入图片描述"> <img src="https://img-blog.csdnimg.cn/20181124110230356.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

## SSL剥离攻击

该攻击方式主要是利用用户并不会每次都直接在浏览器上输入https://xxx.xxx.com来访问网站，或者有些网站并非全网HTTPS，而是只在需要进行敏感数据传输时才使用HTTPS的漏洞。 中间人攻击者在劫持了客户端与服务端的HTTP会话后，将HTTP页面里面所有的https://超链接都换成http://，用户在点击相应的链接时，是使用HTTP协议来进行访问。 这样，就算服务器对相应的URL只支持HTTPS链接，但中间人一样可以和服务建立HTTPS连接之后，将数据使用HTTP协议转发给客户端，实现会话劫持。 <img src="https://img-blog.csdnimg.cn/20181124110308986.png" alt="在这里插入图片描述">

# HTTP发展史
1. HTTP1.0 19961. HTTP1.1 19991. HTTPS 1994提出，2000左右广泛使用1. SPDY 20121. HTTP2.0 2015
## HTTP1.0-&gt;HTTP1.1
1. **缓存处理。** 1.0只是用一个头文件的缓存策略，1.1引入了更多。1. **带宽优化及网络连接的使用**，HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206，这样就方便了开发者自由的选择以便于充分利用带宽和连接。1. **错误通知的管理**，在HTTP1.1中新增了24个错误状态响应码，如409表示请求的资源与资源的当前状态发生冲突；410表示服务器上的某个资源被永久性的删除。1. **Host头处理**，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机，并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误。1. **长连接（最大区别）**，HTTP 1.1支持长连接和请求的流水线处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection：keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。很多浏览器可以设置允许最大并发连接数提高浏览速度，如果只有一个连接，上一次的Response没有收到，那么下一次的请求不能处理。空闲时会自动断开，通常300s左右。
## SPDY即HTTP2.0前身

<img src="https://img-blog.csdnimg.cn/20181124103202464.png" alt="在这里插入图片描述">
1. **多路复用（单一长连接）**，降低延迟。针对HTTP高延迟的问题，SPDY优雅的采取了多路复用。多路复用通过多个请求stream共享一个tcp连接的方式，解决了HOL blocking的问题，降低了延迟同时提高了带宽的利用率。1. **请求优先级。** 多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。SPDY允许给每个request设置优先级，这样重要的请求就会优先得到响应。比如浏览器加载首页，首页的html内容应该优先展示，之后才是各种静态资源文件，脚本文件等加载，这样可以保证用户能第一时间看到网页内容。1. **header压缩**，使用静态字典压缩。前面提到HTTP1.x的header很多时候都是重复多余的。选择合适的压缩算法可以减小包的大小和数量。（比如只要从服务器拉一个int数字，但是头部却有几十个字节。）1. **基于HTTPS的加密协议传输，大大提高了传输数据的可靠性**。（HTTP2.0中不限制一定要HTTPS）1. **双端交互，服务器推送**。采用了SPDY的网页，例如我的网页有一个sytle.css的请求，在客户端收到sytle.css数据的同时，服务端会将sytle.js的文件推送给客户端，当客户端再次尝试获取sytle.js时就可以直接从缓存中获取到，不用再发请求了。
## 长连接和多路复用区别

长连接能够在客户端和服务器交互频繁的情况下，不用反复得建立和断开连接。但是长连接还有两个问题： 1、如果只允许存在一个长连接，第一个请求发送后，第一个Response还没有返回，这时候第二个请求来了，第二个请求要等到第一个Response返回后才能发送出去。 2、如果允许存在多个长连接，第一个请求发送后，第一个Response还没有返回，这时候第二个请求来了，那么第二个请求就可能会再开一个长连接。之后如果请求不会这么频繁了，那么这个完全就是资源浪费啊，因此长连接的管理也是一个很麻烦的问题。（下图是不同浏览器对长连接数目的限制） <img src="https://img-blog.csdnimg.cn/20181124103417187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">

多路复用则在长连接的基础上进一步优化，多路复用情况下，不必等待第一次的Response返回就可以发送第二次的请求，如下图： <img src="https://img-blog.csdnimg.cn/20181124103336561.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RvdWJsZTJoYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述">
