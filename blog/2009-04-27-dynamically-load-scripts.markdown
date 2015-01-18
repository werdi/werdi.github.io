# Динамическая загрузка скриптов
Например, на веб-сайте в папке Script есть файл JScript.js:
```js
function Test()
{
    	alert("TEST");
}
```
В какой-то момент времени требуется из Silverlight вызвать загрузку .js-файла и выполнение функции Test. При этом загрузка файла должна происходить после каждого его изменения. Это можно сделать следующим образом в Silverlight:
```c#
ScriptHelper.Invoke("/Scripts/JScript.js?guid=" + Guid.NewGuid(), () =>
{
    	HtmlPage.Window.Invoke("Test");
});
```
В метод Invoke передается путь к файлу. Guid используется для того, чтобы файл не кешировался браузером.
Принцип работы метода: создается тег SCRIPT; добавляется на веб-страницу в тег HEAD; происходит перехват события onreadystatechanged и в его обработчике проверяется состояние SCRIPT.readyState; если loaded, то вызвать метод в Silverlight.
Ниже код класса с extension method'ами:
```c#
public static class ScriptHelper
{
    	public static HtmlElement FindByUri(string uri)
    	{
        	foreach (var script in HtmlPage.Document.GetElementsByTagName("script"))
        	{
            	if (string.Equals((string)script.GetProperty("src"), uri, StringComparison.OrdinalIgnoreCase))
                	return (HtmlElement)script;
        	}
        	return null;
    	}

    	public static ScriptReadyState ReadyState(this HtmlElement scriptTag)
    	{
        	return (ScriptReadyState)Enum.Parse(typeof(ScriptReadyState), (string)scriptTag.GetProperty("readyState"), true);
    	}

    	public enum ScriptReadyState
    	{
        	Uninitialized = 0,
        	Loading = 1,
        	Loaded = 2,
        	Interactive = 3,
        	Complete = 4
    	}

    	public static void OnLoaded(this HtmlElement scriptTag, Action callback)
    	{
        	if ((int)scriptTag.ReadyState() > 1)
        	{
            	callback();
        	}
        	else
        	{
            	// обработчик событий
            	EventHandler<HtmlEventArgs> eh = (s, e) =>
            	{
                	// проверить событие и состояние и вызвать handler
                	if (e.EventType == "readystatechange" && (int)scriptTag.ReadyState() > 1)
                    	callback();
            	};

            	// подключить обработчик
            	scriptTag.AttachEvent("onreadystatechange", eh);
        	}
    	}

    	// handler - в IE вызовется раньше, чем скрипт будет выполнен;
    	public static void Invoke(string scriptUri, Action callback)
    	{
        	HtmlElement script = FindByUri(scriptUri);
        	if (script == null)
        	{
            	// создать тег и добавить на страницу в тег HEAD;
            	script = Create(scriptUri);
            	var head = (HtmlElement)HtmlPage.Document.GetElementsByTagName("head").FirstOrDefault();
            	head.AppendChild(script);
        	}

        	script.OnLoaded(callback);
    	}

    	public static HtmlElement Create(string scriptUri)
    	{
        	// создать тег скрипта
        	var js = HtmlPage.Document.CreateElement("script");
        	js.SetProperty("type", ScriptType);
        	js.SetProperty("src", scriptUri);
        	return js;
    	}

    	public static string ScriptType = "text/javascript";
}
```
Если js-функции не существует, то получим ошибку ... см. [здесь] (http://cognitex.blogspot.com/2009/04/javascript_23.html).
Код работает в IE.
