# Как добавить роут в существующий сайт ASP.NET MVC
В ASP.NET MVC проекте добавление роутов происходит в Global.asax.cs, который после компиляции проекта становится недоступен. Если при этом надо добавить свой роут, то это можно сделать в так называемом well-known методе AppInitialize. 
Для этого: 
<ol>
  <li>в корень сайта добавить папку App_Code</li>
  <li>создать cs файл, например, AppStart.cs со следующим кодом:</li>
</ol>
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Hosting;

public class Start
{
    	public static void AppInitialize()
    	{
        	// здесь добавляем свой роут в System.Web.Routing.RouteTable.Routes
    	}
}
```
Добавленный роут будет выше остальных, т.е. можно перехватывать все запросы, например, так:
```c#
routes.MapRoute("Default",
    	"{*request}",
    	new { Controller = "Site", Action = "Test" }
    	);
```
Все запросы попадут в метод Test, определенный в классе SiteController.
```c#
using System;
using System.ComponentModel;
using System.Web.Mvc;

namespace Site
{
    	[HandleError]
    	public class SiteController : Controller
    	{
        	public ActionResult Test(string request)
        	{
             	return View();
        	}
    	}
}
```
