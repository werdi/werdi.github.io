# Использование JQuery в Silverlight
Вместе с [ASP.NET MVC] (http://www.asp.net/mvc/) поставляется [JQuery] (http://jquery.com/) ([краткое описание] (http://ru.wikipedia.org/wiki/JQuery)).
Предположим, что Silverlight-приложение используется на одной из страниц ASP.NET MVC проекта, на которой есть вызов JQuery. Т.е. в html-коде страницы в head указан тег:
```html
<script type="text/javascript" src="Scripts/jquery-1.3.2.min.js"></script>
```
Требуется получить границы тега button:
```html
<button id="btn">test</button>
```
Решение: использовать JQuery-функцию [offset] (http://docs.jquery.com/CSS/offset).
Для этого в Silverlight-приложение надо добавить следующий extension method:
```c#
public static class JQueryHelper
{
    	public static Rect GetBounds(this HtmlElement el)
    	{
        	ScriptObject jqo = HtmlPage.Window.CreateInstance("$", new[] { el });
        	return GetBounds(jqo);
    	}

    	private static Rect GetBounds(ScriptObject jqo)
    	{
        	ScriptObject res = (ScriptObject)jqo.Invoke("offset");
        	return new Rect(
            	(double)res.GetProperty("left"),
            	(double)res.GetProperty("top"),
            	(double)jqo.Invoke("width"),
            	(double)jqo.Invoke("height")
        	);
    	}
}
```
Координаты кнопки можно получить следующим образом:
```c#
Rect rect = HtmlPage.Document.GetElementById("btn").GetBounds();
System.Diagnostics.Debug.WriteLine(rect);
```
