# Как получить текст script'ов в Silverlight
С помощью метода GetJavaScripts можно получить список скриптов, размещенных на веб-странице:
```c#
foreach (string script in HtmlPage.Document.GetJavaScripts())
{
    	System.Diagnostics.Debug.WriteLine(script);
}
```
Метод находит все теги SCRIPT и отбирает только те, у которых есть атрибут type="text/javascript".
Для использования метода необходимо в Silverlight-приложение добавить следующий код:
```c#
public static class ScriptHelper
{
    	public static IEnumerable<string> GetJavaScripts(this HtmlDocument doc)
    	{
        	return doc.GetScripts("text/javascript");
    	}

    	public static IEnumerable<string> GetScripts(this HtmlDocument doc, string type)
    	{
        	Regex reg = new Regex(@"<script.+?type=" + type + ".*?>", RegexOptions.Singleline | RegexOptions.IgnoreCase);
        	foreach (var scr in doc.GetElementsByTagName("script"))
        	{
            	string html = scr.GetProperty("outerHTML") as string;
            	if (reg.IsMatch(html))
                	yield return html;
        	}
    	}
}
```
Если скрипт находится в отдельном файле, то его код с помощью этого метода недоступен (в html получим, например: "\r\n<SCRIPT type=text/javascript src=\"Silverlight.js\"></SCRIPT>");
