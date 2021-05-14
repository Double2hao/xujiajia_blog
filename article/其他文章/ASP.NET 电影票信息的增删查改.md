#ASP.NET 电影票信息的增删查改
# 题目

1、使用Code First技术创建一个Movie数据模型。

```
public class Movie
 {
        public int ID { get; set; }      //电影编号
        public string Title { get; set; }     //电影名称
        public DateTime ReleaseDate { get; set; }    //上映时间
        public string Genre { get; set; }      //电影类型
        public decimal Price { get; set; }    //电影票价
        public string Rating { get; set; }     //电影分级
 }

```

2、使用MVC相关技术实现数据的列表显示和新增功能。

3、完成数据的编辑、删除、明细和条件查询等功能。

4、完成如下查询： （1）查询尚未上映电影的信息 （4）查询票价在某个区间的电影信息

# 效果

## （源码在文章结尾）

<img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910415790.png " alt="这里写图片描述"> <img src="https://raw.githubusercontent.com/Double2hao/xujiajia_blog/main/img/16209910421971.png " alt="这里写图片描述">

# 主要涉及知识点

1、 WEB MVC下的目录结构以及基础编程 2、Linq查询操作 3、Code First 4、各模板View的建立和使用

# 主要代码

## MovieController.cs

```
using ProjectThree.Models;
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace ProjectThree.Controllers
{

    public class MovieController : Controller
    {
        MovieDBContext db = new MovieDBContext();

        // GET: Movie
        public ActionResult Index(string movieOn, string movieGenre,
            string searchString, string lowPrice, string highPrice)
        {
            //初始化电影是否上映下拉
            var GenreLst1 = new List&lt;string&gt;();
            GenreLst1.Add("是");
            GenreLst1.Add("否");
            ViewBag.movieOn = new SelectList(GenreLst1);
            //初始化电影类型下拉
            var GenreLst2 = new List&lt;string&gt;();
            var GenreQry = from d in db.Movies orderby d.Genre select d.Genre;
            GenreLst2.AddRange(GenreQry.Distinct()); //去重
            ViewBag.movieGenre = new SelectList(GenreLst2);

            var movies = from m in db.Movies select m;
            if (!String.IsNullOrEmpty(movieOn))
            {
                DateTime dtNow = DateTime.Now;
                if (movieOn.Equals("是"))
                { movies = movies.Where(s =&gt; DateTime.Compare(dtNow, s.ReleaseDate) &gt; 0); }
                else if (movieOn.Equals("否"))
                { movies = movies.Where(s =&gt; DateTime.Compare(dtNow, s.ReleaseDate) &lt;= 0); }
            }
            if (!String.IsNullOrEmpty(movieGenre))
            { movies = movies.Where(x =&gt; x.Genre == movieGenre); }
            if (!String.IsNullOrEmpty(searchString))
            { movies = movies.Where(s =&gt; s.Title.Contains(searchString)); }

            if ((!String.IsNullOrEmpty(lowPrice)) &amp;&amp; (!String.IsNullOrEmpty(highPrice)))
            {
                try
                {
                    Decimal low = Decimal.Parse(lowPrice);
                    Decimal high = Decimal.Parse(highPrice);
                    if (high &lt; low)
                    {
                        Response.Write("&lt;script&gt;alert('左边价格不可大于右边!');&lt;/script&gt;");
                    }
                    else
                    {
                        movies = movies.Where(s =&gt; s.Price &gt;= low);
                        movies = movies.Where(s =&gt; s.Price &lt;= high);
                    }
                }
                catch
                {
                    Response.Write("&lt;script&gt;alert('必须输入数字!');&lt;/script&gt;");
                    return View(movies);
                }
               
               
            }

            return View(movies);
        }


        public ActionResult Create()
        {
            return View();
        }

        [HttpPost]
        public ActionResult Create(Movie m)
        {
            if (ModelState.IsValid)
            {
                db.Movies.Add(m);
                db.SaveChanges();
                return RedirectToAction("Index", "Movie");
            }
            return View(m);
        }


        public ActionResult Delete(int? id)
        {
            Movie m = db.Movies.Find(id);
            if (m != null)
            {
                db.Movies.Remove(m);
                db.SaveChanges();
            }

            return RedirectToAction("Index", "Movie");
        }

        public ActionResult Edit(int id)
        {
            Movie stu = db.Movies.Find(id);
            return View(stu);
        }

        [HttpPost]
        public ActionResult Edit(Movie stu)
        {
            db.Entry(stu).State = EntityState.Modified;
            db.SaveChanges();

            return RedirectToAction("Index", "Movie");
        }

    }


}

```

## Movie.cs

```
using System;
using System.ComponentModel.DataAnnotations;

namespace ProjectThree.Models
{
    public class Movie
    {
        [Display(Name = "电影编号")]
       
        public int ID { get; set; } //电影编号

        [Display(Name = "电影名称")]
        [Required(ErrorMessage = "必填")]
        [StringLength(60, MinimumLength = 3, ErrorMessage = "必须是[3,60]个字符")]
        public string Title { get; set; } //电影名称

        [Display(Name = "上映时间")]
        [DataType(DataType.Date)]
        [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}",ApplyFormatInEditMode = true)]
        public DateTime ReleaseDate { get; set; } //上映时间

        [Display(Name = "电影类型")]
        [Required]

        public string Genre { get; set; } //电影类型

        [Display(Name = "电影票价")]
        [Range(1, 100)]
        [DataType(DataType.Currency)]
        public decimal Price { get; set; } //电影票价

        [Display(Name = "电影分级")]
        [StringLength(5)]
        [Required]

        public string Rating { get; set; } //电影分级

    }

}

```

## MovieDBContext.cs

```

using System.Data.Entity;


namespace ProjectThree.Models
{
    public class MovieDBContext : DbContext

    {
        public DbSet&lt;Movie&gt; Movies { get; set; }
    }
}

```

## Index.cshtml

```
@model IEnumerable&lt;ProjectThree.Models.Movie&gt;

@{
    ViewBag.Title = "Index";
}

&lt;p&gt;
    @Html.ActionLink("新建", "Create")
    @using (Html.BeginForm("Index", "Movie", FormMethod.Get))
    {
    &lt;p&gt;
        电影是否上映:@Html.DropDownList("movieOn", "all")
        电影类型:@Html.DropDownList("movieGenre", "all")
        电影名称:@Html.TextBox("SearchString")
        票价区间:@Html.TextBox("lowPrice")~@Html.TextBox("highPrice")
        &lt;input type="submit" value="查询" /&gt;
    &lt;/p&gt;
    }
&lt;/p&gt;

&lt;table class="table"&gt;
    &lt;tr&gt;
        &lt;th&gt;
            @Html.DisplayNameFor(model =&gt; model.Title)
        &lt;/th&gt;
        &lt;th&gt;
            @Html.DisplayNameFor(model =&gt; model.ReleaseDate)
        &lt;/th&gt;
        &lt;th&gt;
            @Html.DisplayNameFor(model =&gt; model.Genre)
        &lt;/th&gt;
        &lt;th&gt;
            @Html.DisplayNameFor(model =&gt; model.Price)
        &lt;/th&gt;
        &lt;th&gt;
            @Html.DisplayNameFor(model =&gt; model.Rating)
        &lt;/th&gt;
        &lt;th&gt;&lt;/th&gt;
    &lt;/tr&gt;

@foreach (var item in Model) {
    &lt;tr&gt;
        &lt;td&gt;
            @Html.DisplayFor(modelItem =&gt; item.Title)
        &lt;/td&gt;
        &lt;td&gt;
            @Html.DisplayFor(modelItem =&gt; item.ReleaseDate)
        &lt;/td&gt;
        &lt;td&gt;
            @Html.DisplayFor(modelItem =&gt; item.Genre)
        &lt;/td&gt;
        &lt;td&gt;
            @Html.DisplayFor(modelItem =&gt; item.Price)
        &lt;/td&gt;
        &lt;td&gt;
            @Html.DisplayFor(modelItem =&gt; item.Rating)
        &lt;/td&gt;
        &lt;td&gt;
            @Html.ActionLink("编辑", "Edit", new { id=item.ID }) |
            @Html.ActionLink("详情", "Details", new { id=item.ID }) |
            @Html.ActionLink("删除", "Delete", new { id=item.ID }, new { onclick = "return confirm('确认删除吗？')" })
        &lt;/td&gt;
    &lt;/tr&gt;
}

&lt;/table&gt;


```

## Create.cshtml

```
@model ProjectThree.Models.Movie

@{
    ViewBag.Title = "Create";
}


@using (Html.BeginForm()) 
{
    @Html.AntiForgeryToken()
    
    &lt;div class="form-horizontal"&gt;
        &lt;h4&gt;Movie&lt;/h4&gt;
        &lt;hr /&gt;
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })
        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.Title, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.Title, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.Title, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.ReleaseDate, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.ReleaseDate, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.ReleaseDate, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.Genre, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.Genre, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.Genre, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.Price, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.Price, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.Price, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.Rating, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.Rating, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.Rating, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            &lt;div class="col-md-offset-2 col-md-10"&gt;
                &lt;input type="submit" value="Create" class="btn btn-default" /&gt;
            &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
}

&lt;div&gt;
    @Html.ActionLink("Back to List", "Index")
&lt;/div&gt;


```

## Edit.cshtml

```
@model ProjectThree.Models.Movie

@{
    ViewBag.Title = "Edit";
}

&lt;h2&gt;Edit&lt;/h2&gt;

@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()
    
    &lt;div class="form-horizontal"&gt;
        &lt;h4&gt;Movie&lt;/h4&gt;
        &lt;hr /&gt;
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })
        @Html.HiddenFor(model =&gt; model.ID)

        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.Title, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.Title, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.Title, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.ReleaseDate, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.ReleaseDate, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.ReleaseDate, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.Genre, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.Genre, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.Genre, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.Price, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.Price, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.Price, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            @Html.LabelFor(model =&gt; model.Rating, htmlAttributes: new { @class = "control-label col-md-2" })
            &lt;div class="col-md-10"&gt;
                @Html.EditorFor(model =&gt; model.Rating, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model =&gt; model.Rating, "", new { @class = "text-danger" })
            &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="form-group"&gt;
            &lt;div class="col-md-offset-2 col-md-10"&gt;
                &lt;input type="submit" value="Save" class="btn btn-default" /&gt;
            &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
}

&lt;div&gt;
    @Html.ActionLink("Back to List", "Index")
&lt;/div&gt;


```

# 源码地址

