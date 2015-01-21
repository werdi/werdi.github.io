---
title: Как проверить existance JavaScript-функции
tags: [javascript, silverlight, c#]
src: http://cognitex.blogspot.ru/2009/04/javascript_23.html
---
# Как проверить existance JavaScript-функции
Из Silverlight-приложения можно проверить existance (существование) функции с помощью следующего метода:
```c#
bool res = HtmlPage.Window.IsFunctionExists("MyFunc");
```
Метод вернет true для функции, которая определена в теге SCRIPT на странице или определена в подгружаемом .js-файле (например: <script type="text/javascript" src="Silverlight.js"></script>).
Метод определен в следующем классе:
```c#
public static class FunctionHelper
{
    	public static bool IsFunctionExists(this HtmlWindow wnd, string functionName)
    	{
        	return (bool)wnd.Eval("typeof " + functionName + " == \"function\"");
    	}
}
```
