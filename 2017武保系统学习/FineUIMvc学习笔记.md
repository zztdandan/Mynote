

#UIHelper.Result-----20170419
----------------

![UIHelperImg](./img/UIHelperImg.png)

在 ASP.NET MVC 中并没有 UIHelper 这样一个静态类

这一切的一切还要从 ActionResult 入手。
####ActionResult
作为控制器方法（Action）的返回值（Result），这是一个抽象类。

MVC 内置了很多 ActionResult 的实现类，最常用的可能是下面几个：

1. ViewResult：返回视图。

2. ContentResult：返回一个静态字符串。

3. RedirectResult：重定向到另一个URL，类似于 WebForms 中的 Response.Redirect 函数，返回的HTTP状态码是 302 Redirect。

4. JsonResult：返回一个JSON字符串。

5. HttpNotFoundResult：未找到对象，一般用于URL参数错误，返回的HTTP状态码是 404 Not Found。


但是这里面没有 FineUIMvc 回发时能用的 ActionResult， FineUI 回发时返回的是一串 JavaScript 代码：

        意思就是，这是个特定的返回fineui所需格式的类

------------
FineUIMvc.AjaxResult 会处理回发过程中每个控件的改变，并转化为 JavaScript 代码并返回。

比如这样一段 C# 代码：
```
  Alert.Show("你好 FineUIMvc！", MessageBoxIcon.Warning);
 ```

在返回的 HTTP 响应中，对应于：
```
  F.alert({message:'你好 FineUIMvc！',messageIcon:'warning'});
 ```
这个转化过程就是由 FineUIMvc.AjaxResult 类负责的。
----------
ASP.NET MVC 中的 HttpPost 控制器方法中为啥不需要类似 AjaxResult 的实现：

```
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Create([Bind(Include = "ID,Name,Gender,Major,EntranceDate")] Student student)
{
       if (ModelState.IsValid)
       {
              db.Students.Add(student);
              db.SaveChanges();
              return RedirectToAction("Index");
       }
 
  return View(student);
}
```
可以看到，原生的 HttpPost 控制器方法直接返回的是视图（ViewResult的实例），也就是整个页面的更新，非AJAX过程。
---------
####UIHelper.Button，UIHelper.Grid，UIHelper.Tree... 

下面通过一个例子来详细说明这一过程：

在线示例地址：[http://fineui.com/demo_mvc#/demo_mvc/Button/Button](http://fineui.com/demo_mvc#/demo_mvc/Button/Button)

![btndemo](./img/btndemo.png)

在这个例子中，点击第一个按钮，会不断切换第二个按钮[按下的按钮]的状态，第一个按钮的点击事件处理函数：
```
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult btnChangePressed_Click(bool pressed)
{
    UIHelper.Button("btnPressed").Pressed(!pressed);

    return UIHelper.Result();
}

```
----------------

#MVC Area领域处理以及T4MVC的使用------20170420
-------------
第一部分：（摘选自[http://www.cnblogs.com/HuiTai/archive/2012/07/24/MVC-12.html](http://www.cnblogs.com/HuiTai/archive/2012/07/24/MVC-12.html) 辉太）

（这部分作者写的很详细，直接摘过来了）

MVC框架支持组织一个web应用程序到的区域,每个区域代表应用程序的功能性组比如账单、客户支持,等等,这在一个大的项目是非常有用的,那里有 一套单一的文件夹,所有的控制器,视图和模型可以变得难以管理。每个MVC区域是有自己的文件夹结构,允许您分开管理。这使得它更显而易见哪个项目元素相 互关联应用程序的功能区域，这有助于多个开发人员同事处理项目而没有彼此胡想不干扰。区域是支持主要通过路由机制。

我们从新新建一个MVCweb应用程序("MVCArea")，创建好项目，我们直接演示怎么给项目添加一个区域进来具体如下图3.-4.
![图三](./img/2012072422404876.jpg)
![图四](./img/2012072422411569.jpg)