#android使用gradle优化多module的依赖管理
# 概述

在一个项目有多个module 的时候，不可避免的会存在不同的module会有相同的依赖的问题。 最一般的做法，我们开发要引入一个依赖的时候，就去看一下其他项目依赖了哪个版本，然后复制粘贴。

但是一旦发现其他多项目对同一个库，存在依赖多个版本的时候，也就不知道复制哪个了。碰到这种情况，就只能增加沟通成本了。 针对这种情况，我们可以对依赖增加一个统一的版本控制。这样即使是刚参与开发的同事也能不需要熟悉项目的情况下直接引入需要的依赖。

>  
 如果使用这种方式全局统一依赖的版本，那么就几乎也不会存在版本不同导致依赖冲突的问题了。 


# demo结构

**app** ：主项目。 依赖onetest和gson。 **onetest**：test项目。 依赖gson。 **commen.gradle**：为了统一依赖而加入的gradle文件。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16210039939310.png" alt="在这里插入图片描述">

# demo中的依赖管理
1. common.gradle中统一 gson版本1. 根项目的build.gradle 应用 common.gradle1. app项目中依赖 onetest和gson。1. onetest中依赖 gson
>  
 避免依赖冲突，app项目中依赖gson使用implementation，onetest中依赖gson使用compileOnly。 


### common.gradle

```
ext {<!-- -->

  depend = [
      "gson": 'com.google.code.gson:gson:2.8.2',
  ]
}

```

>  
 ext是Gradle领域对象的一个属性，我们可以将自定义的属性添加到ext对象上。 


### 根项目的build.gradle

需要再根项目的build.gradle 顶部加上如下代码。

```
apply from: "common.gradle"

```

### app项目依赖

build.gradle中的依赖如下：

```
dependencies {<!-- -->
    project (':onetest')
    implementation rootProject.ext.depend["gson"]
}

```

### onetest项目依赖

build.gradle中的依赖如下：

```
dependencies {<!-- -->
  compileOnly rootProject.ext.depend["gson"]
}

```