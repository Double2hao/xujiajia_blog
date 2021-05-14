#Java 模块化（gradle）
# 模块化

什么是模块化，直接看下面两张图。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039228390.png" alt="在这里插入图片描述"> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039228881.png" alt="在这里插入图片描述"> 上面是非模块化的项目，下面是模块化的项目。

# 非模块化的问题

如果项目本身有多个互相不影响的模块，甚至有多人分开负责各个模块的开发时，非模块化项目的弊端就会暴露出来，主要是有下面几个： 1、依赖难以管理，不同的模块依赖不同的库，甚至是同一个库的不同版本。 2、各个模块单独打包麻烦。 3、增加额外开发成本，开发本身可能只需要开发一个模块，但是由于代码写在一起，所以不得不去了解整个项目。 4、如果一个项目有几十甚至几百个模块，模块化能极大减少编译时间。

# 例子

光讲理论还是让人难以理解，那么用个简单的例子来具体讲一下。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039229812.png" alt="在这里插入图片描述"> 上图是非模块化的一个项目，其中包括base包中的一些基类，然后有Consumer和Producer两个应用类，Main类如下：

```
import implement.Consumer;

public class Main {
    public static void main(String[] args){
        new Consumer().start();
//        new Producer().start();
    }
}

```

>  
 场景是这样：我们有两个服务器，一个是消费者服务器，一个是生产者服务器，他们有相同的基类，但是提供服务器的逻辑是不同的，所以不同服务器上的jar需要分别打包。 


如果是非模块化项目，我们就只能像上面的代码一样做，在打包Consumer的jar的时候将Producer代码注释掉，而在打包Producer的jar的时候将Consumer代码注释掉。 但是，实际开发中，代码并不是这么简单的，Consumer和Producer本身的逻辑可能就会比较复杂，这就会带来每次打包就会带来额外工作量的问题，久而久之，项目越来越大，只有对整个项目都了解了才会知道如何打包。 甚至，之后如果是多人负责这个项目，由于Consumer和Producer的代码都写在一个Project内，Consumer的程序员改动某一个依赖，很可能会给Producer的代码带去风险。 在这个例子中，如果使用模块化，就可以解决一下几个问题： 1、单独打包问题 2、依赖管理问题 3、开发只需要专注自己的模块，降低开发成本

# 如何模块化

主要步骤如下： 1、创建module，将代码放到各个module中 2、setting.gradle 添加module 3、依赖分开管理

## 创建module

如果和笔者一样使用的是IDEA的话，直接File-&gt;New-&gt;Module就可以创建。使用IDEA创建，src、build.gradle等文件，IDEA也会自动帮忙创建好。 创建后如下图： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039230573.png" alt="在这里插入图片描述"> 如果不是使用IDEA，直接在主项目目录下创建BaseProject文件就可以，然后再去创建src等文件。 module都创建完毕后将代码都放到各个module中，效果大致如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039231944.png" alt="在这里插入图片描述">

## setting.gradle 添加module

只有在setting.gradle 中添加了module，gradle构建项目的时候才能识别这几个文件夹是module。 如果是使用IDEA创建的，IDEA会自动在setting.gradle 添加module，如果是自己手动创建的项目，则也需要手动添加，该文件内容如下：

```
rootProject.name = 'TestProject'
include 'BaseProject'
include 'Consumer'
include 'Producer'

```

## 依赖分开管理

创建完module，将原来的类直接移动到module中，我们可以发现是会有error的，效果大致如下： <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039232965.png" alt="在这里插入图片描述"> 可以看到是因为找不到Base类，因此我们要在Consumer和Producer的module中添加BaseProject这个module的依赖，如下图，可以看到error已经消失。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039235046.png" alt="在这里插入图片描述">

如果本身在主项目中，module就有一些依赖，可以将这些依赖分开到各个module中。 就拿例子中的项目来说，共用的一些依赖可以放到BaseProject这个module中，私有的一些依赖就分别放到Consumer和Producer这两个module中。