#一篇文章弄懂cookie、session和token
# 概念

## cookie

cookie存储在客户端，HTTP是无状态的，HTTP每次发出的时候会附上该域名下的cookie，从而可以给HTTP附上状态，最常见的就是登录态。

## session和token

session和token算是一类的，他们是两种不同的服务器的验证方式。 通俗来说，cookie会存一个value在客户端本地，然后将value附到HTTP上发给服务器，那么服务器是怎么通过这个value来判断用户是否是登录态的呢？这就是session和token做的事情。

# session过程

请求过程： 1、客户端向服务器请求，发送用户名和密码 2、服务器生成sessionId，绑定用户数据存储在数据库 3、服务器返回sessionId给客户端 4、客户端用cookie存储sessionId，以后的请求都带上这个sessionId 5、服务器如果收到这个sessionId，那么就去数据库查找用户数据，如果找到了说明验证通过 6、服务器把验证结果返回客户端 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910576420.png " alt="在这里插入图片描述">

# token过程

请求过程： 1、客户端向服务器请求，发送用户名和密码 2、服务器根据用户信息通过加密生成token，用户信息包括账号，token过期时间等，具体由服务器自定义。 3、服务器返回token给客户端 4、客户端用cookie存储token，以后的请求都带上这个token 5、服务器把token解密，确认用户信息是否正确，如经过正确就说明验证通过。 6、服务器把验证结果返回客户端 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910578351.png " alt="在这里插入图片描述">

>  
 上图转载自: 


# session、token优劣

## session

由于sessionId和用户信息相互绑定的数据库存在服务器，所以服务器可以随时让发送出去的一个sessionId失效。这是保障安全的一种重要手段。

## token

token的好处是比session更省空间和时间，服务器不需要去管理sessionId和用户信息的数据库，服务器收到token直接解密就可以验证，不需要去数据库查找验证。 但是token发送出去之后，就只能等待它达到过期时间后才会失效，后台无法对其进行控制。