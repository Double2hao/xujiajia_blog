#WebPack使用流程小记
>  
 本篇文章记录为主  主要参考WebPack官网： 


# WebPack常用流程

1、安装WebPack  2、准备要打包的文件  3、安装loader  4、配置文件  5、生成文件并运行

# 示例

### 文件目录（WebStorm）

<img src="https://img-blog.csdn.net/20171114181746356?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG91YmxlMmhhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="这里写图片描述" title="">

### 安装WebPack

```
npm install webpack -g
```

### 准备要打包的文件

**entry.js**

```
require("./style.css");
document.write('It works.');
```

**index.js**

```
&lt;html&gt;
&lt;head&gt;
    &lt;meta charset="utf-8"&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;script type="text/javascript" src="bundle.js" charset="utf-8"&gt;&lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;
```

**style.css**

```
body {
    background: yellow;
}
```

### 安装loader

```
npm install css-loader style-loader
```

### 配置文件

**webpack.config.js**

```
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader:'style-loader!css-loader' }
        ]
    }
};
```

### 生成文件并运行

直接在终端输入

```
webpack
```

然后运行index.html，最终效果如下：  <img src="https://img-blog.csdn.net/20171114182355539?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG91YmxlMmhhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="这里写图片描述" title="">
