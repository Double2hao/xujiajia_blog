# 概述

android 生成aar后，引用此aar的项目不会自动引用 aar中的依赖，一般情况下，需要开发者在使用这个aar的项目中再自己手动添加需要的依赖。

那么如何能让一个项目引用aar后，自动引入相关的依赖呢？

# 实现方式
上传aar到maven仓库前，修改maven中的pom文件，在其中写入aar相关的依赖。

### build.gradle代码
```xml
plugins {
  id 'com.android.library'
  id 'maven'
}

android {
  compileSdkVersion 30
  buildToolsVersion "30.0.3"

  defaultConfig {
    minSdkVersion 24
    targetSdkVersion 30
    versionCode 1
    versionName "1.0"

    testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    consumerProguardFiles "consumer-rules.pro"
  }

  buildTypes {
    release {
      minifyEnabled false
      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
  }
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}

dependencies {
  compileOnly 'com.squareup.okhttp:okhttp:2.7.5'
}

def groupId = "com.test.xujiajia";
def artifactId = "mavenproject"
def version = "0.0.1"

uploadArchives {
  repositories {
    mavenDeployer {
      repository(url: uri('../repo'))

      pom.groupId = groupId
      pom.artifactId = artifactId
      pom.version = version
      pom.withXml {
        //设置本项目的pom
        def rootNode = asNode();
        def projectNode = rootNode.children();
        projectNode.each { item ->
          def nodeToString = item.toString();
          if (nodeToString.contains("groupId")) {
            item.setValue(groupId)
          } else if (nodeToString.contains("artifactId")) {
            item.setValue(artifactId)
          } else if (nodeToString.contains("version")) {
            item.setValue(version)
          }
        }
        //将项目中的compileOnly的依赖都导入pom
        def depsNode = rootNode.appendNode('dependencies')
        configurations.compileOnly.allDependencies.each { dep ->
          def depNode = depsNode.appendNode('dependency')
          depNode.appendNode('groupId', dep.group)
          depNode.appendNode('artifactId', dep.name)
          depNode.appendNode('version', dep.version)
          depNode.appendNode('scope', 'compile')
        }
      }
    }
  }
}
```
# Demo
前提如下：
- app项目依赖了mavenProject的aar
- app项目没有依赖Okhttp，mavenProject依赖了OkHttp
- app项目可以直接使用OKhttp

![在这里插入图片描述](https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/20210313193220882.png)


### app项目的依赖如下
```xml
dependencies {
  implementation 'androidx.appcompat:appcompat:1.2.0'
  implementation 'com.google.android.material:material:1.2.1'
  implementation 'androidx.constraintlayout:constraintlayout:2.0.4'

  //此处只依赖本地的mavenproject，却同时也会依赖mavenproject项目的pom中的okhtto
  implementation 'com.test.xujiajia:mavenproject:0.0.1'
}
```

### demo地址
[https://github.com/Double2hao/aarWithDependencies](https://github.com/Double2hao/aarWithDependencies)
