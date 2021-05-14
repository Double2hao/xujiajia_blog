#ASP.NET 个人信息注册页面,并跳转显示
# 题目

新建一个MVC项目，利用HTML、CSS、JS、jQuery、Ajax、jQuery UI等技术设计一个个人信息注册页面。当点击“提交”按钮时，跳转到新的页面显示录入信息。 **基本要求：** 用户名为6-10个小写字母（小写使用正则式验证，且用户名不能为“wustzz” –用Ajax技术来检测）；密码为6位数字，确认密码不一致时有提示；籍贯使用级联（jquery实现）；Email必须符合Email格式；手机是11位（假设规定以1569开头）；出生年月使用jQuery UI日历组件设置；图片要传递到新的页面显示。 <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910794500.png " alt="这里写图片描述">

# 实现效果

## （源码在文章结尾）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910794841.png " alt="这里写图片描述"> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910798912.png " alt="这里写图片描述">

# 主要涉及知识点

1、基本的html界面编程 2、JavaScript语言 3、jQuery、jQuery UI的使用 4、ASP.NET Request相关操作 5、了解ASP.NET WEB MVC下的目录结构以及基础编程

# 代码

### ProjectController.cs

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace ProjectOne.Controllers
{
    public class ProjectController : Controller
    {
        // GET: Project
        public ActionResult Index()
        {
            return View();
        }

        public ActionResult Show()
        {
            //获取图片文件
            HttpPostedFileBase file = Request.Files["filename"];
            if(file != null)
            {
                //将图片存储在/Content/UpLoad/目录下，名字为111.png
                var fileName = Request.MapPath("~/Content/UpLoad/") + "111.png";
                file.SaveAs(fileName);
            }
             

            return View();
        }
    }
}

```

### Index.cshtml

```

@{
    ViewBag.Title = "Index";
}

&lt;script src="~/Scripts/my_script.js"&gt;&lt;/script&gt;

&lt;script src="~/jquery-ui-1.11.1.custom/external/jquery/jquery.js"&gt;&lt;/script&gt;


&lt;script&gt;
    $(document).ready(function () {
        $("#native_place").change(function () {
            switch ($("#native_place").val()) {
                case "江苏":
                    $("#major").empty();
                    $("#major").append("&lt;option value=''&gt;&lt;/option&gt;");
                    $("#major").append("&lt;option value='江阴'&gt;江阴&lt;/option&gt;");
                    $("#major").append("&lt;option value='无锡'&gt;无锡&lt;/option&gt;");
                    $("#major").append("&lt;option value='常州'&gt;常州&lt;/option&gt;");
                    break;
                case "湖北":
                    $("#major").empty();
                    $("#major").append("&lt;option value=''&gt;&lt;/option&gt;");
                    $("#major").append("&lt;option value='武汉'&gt;武汉&lt;/option&gt;");
                    $("#major").append("&lt;option value='武昌'&gt;武昌&lt;/option&gt;");
                    $("#major").append("&lt;option value='荆州'&gt;荆州&lt;/option&gt;");
                    break;
            }
        });
    });


&lt;/script&gt;

@section scripts{
&lt;script src="~/jquery-ui-1.11.1.custom/jquery-ui.min.js"&gt;&lt;/script&gt;
&lt;link href="~/jquery-ui-1.11.1.custom/jquery-ui.min.css" rel="stylesheet" /&gt;
    &lt;script&gt;
        $(document).ready(function () {
            $("#birthday").datepicker({
                dateFormat: "yy-mm-dd",
                inline: true
            });
        });
    &lt;/script&gt;
}


&lt;h2 style="color:red;font-family:楷体;font-size:30px;"&gt;请输入个人详细信息&lt;/h2&gt;

&lt;form onsubmit="return checkAll()" action="~/Project/Show" method="post" enctype="multipart/form-data"&gt;
    &lt;table&gt;
        &lt;tr&gt;
            &lt;th&gt;用户名&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="text" onblur="checkName()" name="username" id="username" /&gt;
                &lt;span style="color:red;" id="tip_name"&gt;*&lt;/span&gt;
            &lt;/th&gt;
        &lt;/tr&gt;


        &lt;tr&gt;
            &lt;th&gt;密码&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="text" onblur="checkPassword()" name="psd" id="psd" /&gt;
                &lt;span style="color:red;" id="tip_psd"&gt;*&lt;/span&gt;
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;
            &lt;th&gt;确认密码&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="text" onblur="checkPasswordAgain()" name="psd_again" id="psd_again" /&gt;
                &lt;span style="color:red;" id="tip_psd_again"&gt;*&lt;/span&gt;
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;
            &lt;th&gt;性别&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="radio" name="gender" value="男" checked="checked" /&gt; 男
                &lt;input type="radio" name="gender" value="女" /&gt;女
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;
            &lt;th&gt;籍贯&lt;/th&gt;
            &lt;th&gt;
                &lt;select id="native_place" name="native_place"&gt;
                    &lt;option value=""&gt;&lt;/option&gt;
                    &lt;option value="江苏"&gt;江苏&lt;/option&gt;
                    &lt;option value="湖北"&gt;湖北&lt;/option&gt;
                &lt;/select&gt;
                &lt;select id="major" name="major"&gt;&lt;/select&gt;
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;
            &lt;th&gt;Email&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="text" onblur="checkEmail()" id="email" name="email" value="如 xujiajia@qq.com" /&gt;
                &lt;span style="color:red;" id="tip_email"&gt;*&lt;/span&gt;
            &lt;/th&gt;
        &lt;/tr&gt;
        &lt;tr&gt;
            &lt;th&gt;手机号&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="text" onblur="checkPhone()" id="phone" name="phone" value="手机是11位以1569开头的数字" /&gt;
                &lt;span style="color:red;" id="tip_phone"&gt;*&lt;/span&gt;
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;
            &lt;th&gt;专业擅长&lt;/th&gt;
            &lt;th&gt;
                &lt;select name="speciality" multiple="multiple"&gt;
                    &lt;option value="Windows编程"&gt;Windows编程&lt;/option&gt;
                    &lt;option value="单片机编程"&gt;单片机编程&lt;/option&gt;
                    &lt;option value="ASP.NET编程"&gt;ASP.NET编程&lt;/option&gt;
                    &lt;option value="J2EE编程"&gt;J2EE编程&lt;/option&gt;
                    &lt;option value="JAVA编程"&gt;JAVA编程&lt;/option&gt;
                &lt;/select&gt;
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;
            &lt;th&gt;业余爱好&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="checkbox" name="hobby" value="足球" /&gt;足球
                &lt;input type="checkbox" name="hobby" value="篮球" /&gt;篮球
                &lt;input type="checkbox" name="hobby" value="排球" /&gt;排球
                &lt;input type="checkbox" name="hobby" value="唱歌" /&gt;唱歌
                &lt;input type="checkbox" name="hobby" value="其他" /&gt;其他
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;
            &lt;th&gt;个人照片&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="file" id="filename" name="filename" /&gt;
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;
            &lt;th&gt;出生年月&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="text" id="birthday" name="birthday" readonly="readonly" /&gt;
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;
            &lt;th&gt;备注信息&lt;/th&gt;
            &lt;th&gt;
                &lt;textarea name="more_info" cols="40" rows="8"&gt;
                    可以补充一下
                &lt;/textarea&gt;
            &lt;/th&gt;
        &lt;/tr&gt;

        &lt;tr&gt;

            &lt;th&gt;&lt;/th&gt;
            &lt;th&gt;
                &lt;input type="submit" value="提交" /&gt;
                &amp;nbsp;
                &lt;input type="reset" value="重置" /&gt;
            &lt;/th&gt;
        &lt;/tr&gt;

    &lt;/table&gt;
&lt;/form&gt;






```

### Show.cshtml

```

@{
    ViewBag.Title = "Show";
}

&lt;h2 style="color:red;font-family:楷体;font-size:30px;"&gt;个人信息展示&lt;/h2&gt;

&lt;table&gt;
    &lt;tr&gt;
        &lt;th&gt;用户名&lt;/th&gt;
        &lt;th&gt;@Request["username"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;密码&lt;/th&gt;
        &lt;th&gt;@Request["psd"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;确认密码&lt;/th&gt;
        &lt;th&gt;@Request["psd_again"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;性别&lt;/th&gt;
        &lt;th&gt;@Request["gender"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;籍贯&lt;/th&gt;
        &lt;th&gt;@Request["native_place"]&lt;/th&gt;
        &lt;th&gt;@Request["major"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;Email&lt;/th&gt;
        &lt;th&gt;@Request["email"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;手机号&lt;/th&gt;
        &lt;th&gt;@Request["phone"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;专业擅长&lt;/th&gt;
        &lt;th&gt;@Request["speciality"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;业余爱好&lt;/th&gt;
        &lt;th&gt;@Request["hobby"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;个人照片&lt;/th&gt;
        &lt;th&gt;&lt;img id="img" src="~/Content/UpLoad/111.png" alt="" /&gt;&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;出生年月&lt;/th&gt;
        &lt;th&gt;@Request["birthday"]&lt;/th&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
        &lt;th&gt;备注信息&lt;/th&gt;
        &lt;th&gt;@Request["more_info"]&lt;/th&gt;
    &lt;/tr&gt;
&lt;/table&gt;



```

### my_script.js

```
function checkName() {
    var u = document.getElementById("username");
    var t = document.getElementById("tip_name");
    var reg = /^[a-z]{6,10}$/;
    if (!reg.test(u.value)) {
        t.innerHTML = "用户名为6-10个小写字母";
        return false;
    } else {
        if (u.value == "wustzz") {
            t.innerHTML = "用户名不可以为wustzz";
            return false;
        }
        t.innerHTML = "用户名填写正确";
        return true;
    }

}

function checkPassword() {
    var p = document.getElementById("psd");
    var t = document.getElementById("tip_psd");
    var reg = /^\d{6}$/;
    if (!reg.test(p.value)) {
        t.innerHTML = "密码为6位数字";
        return false;
    } else {
        t.innerHTML = "密码填写正确";
        return true;
    }

}

function checkPasswordAgain() {
    var p1 = document.getElementById("psd");
    var p2 = document.getElementById("psd_again");
    var t = document.getElementById("tip_psd_again");
    if (p1.value != p2.value) {
        t.innerHTML = "密码前后不一致"
        return false;
    } else {
        t.innerHTML = "密码确认一致";
        return true;
    }

}

function checkEmail() {
    var e = document.getElementById("email");
    var t = document.getElementById("tip_email");
    var reg = /^[\w-]+(\.[\w-]+)*@[\w-]+(\.[\w-]+)+$/;
    if (!reg.test(e.value)) {
        t.innerHTML = "必须填写Email格式";
        return false;
    } else {
        t.innerHTML = "Email填写正确";
        return true;
    }

}

function checkPhone() {
    var p = document.getElementById("phone");
    var t = document.getElementById("tip_phone");
    var reg = /^1569\d{7}$/;
    if (!reg.test(p.value)) {
        t.innerHTML = "手机是11位以1569开头的数字";
        return false;
    } else {
        t.innerHTML = "填写手机正确";
        return true;
    }

}

function checkAll() {
    if (checkName() &amp;&amp; checkPassword() &amp;&amp; checkPasswordAgain() &amp;&amp;
        checkEmail() &amp;&amp; checkPhone()) {
        return true;
    }
    return false;
}

```

## 源码地址：

http://download.csdn.net/detail/double2hao/9691584