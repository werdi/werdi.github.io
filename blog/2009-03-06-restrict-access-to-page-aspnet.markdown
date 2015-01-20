---
title: Ограничение доступа к страницам в ASP.NET MVC
tags: [asp.net mvc, autorisation]
src: http://cognitex.blogspot.ru/2009/03/blog-post_06.html
---
# Ограничение доступа к страницам в ASP.NET MVC
Предположим, нам необходимо ограничить доступ к определенной странице.
Например, при попытке открыть страницу по ссылке "F1" (см. [пример] (http://cognitex.blogspot.com/2009/03/aspnet-mvc.html)) запрашивать имя и пароль.
Для этого надо определить атрибут Authorize у метода TestController.Help:
```c#
[Authorize]
public ActionResult Help()
{
  ViewData["Text"] = "help";
  return View();
}
```
Теперь при первом обращении к http://localhost:5545/Test/Help будет происходить перенаправление на страницу http://localhost:5545/Account/Login?ReturnUrl=%2fTest%2fHelp
После ввода корректного имени и пароля будет выполнено перенаправление на страницу http://localhost:5545/Test/Help.

Как это работает? ASP.NET MVC считывает у метода атрибут(ы) Authorize примерно следующим образом:
```c#
foreach (var a in this.GetMethodAttributes<AuthorizeAttribute>("About"))
    System.Diagnostics.Trace.WriteLine(a);
```
...
```c#
public static class AtttributeHelper
{
    public static IEnumerable<T> GetMethodAttributes<T>(this object obj, string name)
    {
        if (string.IsNullOrEmpty(name))
            return null;
        Type type = obj is Type ? (Type)obj : obj.GetType();
        var ma = type.GetMember(name);
        return (ma.Length == 0)
	            ? null
	            : ma.First().GetCustomAttributes(typeof(T), true).Cast();
	   }
}
```
P.S.
Для регистрации на сайте должна быть создана БД [(см. здесь)] (http://cognitex.blogspot.com/2009/03/blog-post_1584.html)
