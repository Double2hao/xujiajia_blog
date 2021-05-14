#阿里云+docker搭建简单node.js服务
# 前提概要

实习的时候接触到docker，当时对其理解比较模糊。现在回学校做毕设，正好服务器这一块没人写，于是稍微复习了一下nodejs，买了个阿里云，摆弄了一下docker，搭建了个简单的服务器。

# 最终效果

主要是提供HTTP服务器，直接通过IP访问，提供两个示例：(已停用)      <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910460670.png " alt="这里写图片描述" title="">  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910462761.png " alt="这里写图片描述" title="">

# 主要步骤

1、配置阿里云（此点笔者自己选择，不多描述了）  2、nodejs服务代码  3、拷贝代码到服务器  4、使用dockerfile创建image并且搭建服务

# nodejs服务代码

1、主要express提供服务器  2、数据库使用sqlite  3、图片上传服务器使用到connect-multiparty

主要目录如下：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910463472.png " alt="这里写图片描述" title="">  主要文件只有server.js，代码在下方：

```
var express = require('express');
var fs = require("fs");
var sqlite3 = require('sqlite3');
var multiparty = require('connect-multiparty');
var app = express();

app.use(express.static('public'));
app.use(multiparty({uploadDir: './public/img'}));

app.get('/', function (req, res) {<!-- -->
    res.send('Hello World');
});

app.get('/device', function (req, res) {<!-- -->
    var db = new sqlite3.Database("server.db");
    db.all("SELECT * FROM device", function (err, row) {<!-- -->
        res.send(row);
        db.close();
    });
});

app.get('/device_data', function (req, res) {<!-- -->
    var db = new sqlite3.Database("server.db");
    db.all("SELECT * FROM device_data", function (err, row) {<!-- -->
        res.send(row);
        db.close();
    });
});

app.get('/point_info', function (req, res) {<!-- -->
    var db = new sqlite3.Database("server.db");
    db.all("SELECT * FROM point_info", function (err, row) {<!-- -->
        res.send(row);
        db.close();
    });
});

var multipartMiddleware = multiparty();
app.post('/info_post', multipartMiddleware, function (req, res) {<!-- -->
    console.log(req.body, req.files);

    var body = req.body;
    var name = body.upload_name;
    var type = body.upload_type;
    var longitude = body.longitude;
    var latitude = body.latitude;
    var address = body.upload_address;
    var time = body.upload_time;
    var description = body.upload_description;
    var status = body.approval_status;

    var fileName = req.files.file.path.substring(11);
    var resource = "http://" + req.headers.host + "/img/" + fileName;
    console.log(resource);

    var db = new sqlite3.Database("server.db");
    var add = db.prepare("INSERT OR REPLACE INTO point_info " +
        "(ID,upload_name, upload_type,longitude,latitude,upload_address,upload_time,upload_description," +
        "approval_status,upload_resource) VALUES (?,?,?,?,?,?,?,?,?,?)");
    add.run(null, name, type, longitude, latitude, address, time, description, status, resource);
    add.finalize();
    db.close();


    res.end(JSON.stringify("success"));
});

app.get('/change_status', function (req, res) {<!-- -->
    console.log(req.query);
    var query = req.query;
    var id = query.id;
    var status = query.status;

    var db = new sqlite3.Database("server.db");
    var modify = db.prepare("UPDATE point_info set approval_status=? where id =?");
    modify.run(status, id);
    res.send("success");
    modify.finalize();
    db.close();
});

var server = app.listen(80, function () {<!-- -->
    var host = server.address().address;
    var port = server.address().port;

    console.log("应用实例，访问地址为 http://%s:%s", host, port);

    var db = new sqlite3.Database("server.db");

    setInterval(function () {<!-- -->
        console.log("change device_data");
        for (var i = 0; i &lt; 30; i++) {
            const modify = db.prepare("UPDATE device_data " +
                "set rain_time = ?," +
                "rain_fall = ?," +
                "rain_level = ?," +
                "water_speed = ?," +
                "water_level = ?," +
                "wind_speed = ?," +
                "gas_warn = ?," +
                "general_level = ? " +
                "where id = ? ");

            modify.run(Math.random() * 10,Math.random() * 10,Math.random() * 10,Math.random() * 10,
                Math.random() * 10,Math.random() * 10,Math.random() * 10,Math.random() * 10,i);

            modify.finalize();
        }
    },3000)

});
```

# 拷贝代码到服务器

笔者使用xshell模拟终端，直接通过图形化界面操作，截图如下：  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910464113.png " alt="这里写图片描述" title="">  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910465804.png " alt="这里写图片描述" title="">

# 通过dockefile创建image并且搭建服务

dockerfile内容如下：

```
FROM node
COPY ./project /home
WORKDIR /home/FloodServer
EXPOSE 80
CMD npm install

```

含义如下:

>  
 FROM node：该 image 文件继承官方的 node image。  COPY ./project /home：将当前目录下的project文件夹中所有文件（除了.dockerignore排除的路径），都拷贝进入 image 文件的/home目录。  WORKDIR /home/FloodServer：指定接下来的工作路径为/home/FloodServer。  EXPOSE 80：将容器 80 端口暴露出来， 允许外部连接这个端口。  CMD npm install：在容器启动后执行npm install安装依赖。 


此时当前目录下有dockerfile文件和之前写的nodejs的服务代码。  写完dockerfile文件后，运行如下代码构建image：

```
docker image -t build my_node .
```

-t参数用来指定 image 文件的名字，后面还可以用冒号指定标签。  构建完毕后，我们可以用docker image ls指令查看image。  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910467495.png " alt="这里写图片描述" title="">

image构建完毕，直接通过image创建container，指令如下：

```
docker run -p 80:80 -it my_node /bin/bash
```

各个参数意义如下：

>  
 -p参数：容器的 80端口映射到本机的 80端口。  -it参数：容器的 Shell 映射到当前的 Shell，然后你在本机窗口输入的命令，就会传入容器。  my_node：image 文件的名字（如果有标签，还需要提供标签，默认是 latest 标签）。  /bin/bash：容器启动以后，内部第一个执行的命令。这里是启动 Bash，保证用户可以使用 Shell。 


构建完container之后，容器的Shell映射到了当前的Shell，直接通过node server.js运行服务，服务搭建成功。  <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910469106.png " alt="这里写图片描述" title="">