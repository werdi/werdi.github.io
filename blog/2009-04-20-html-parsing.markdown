---
title: Парсинг HTML'а
tags: [feature, html, mshtml, webclient, c#]
src: http://cognitex.blogspot.ru/2009/04/html-regex.html
---
# Парсинг HTML'а
Парсить html, конечно, можно с помощью регулярных выражений (Regex), но ... например, требуется получить содержимое всех тегов, у которых атрибут id начинается с подчеркивания или заканчивается цифрой. При этом веб-сервер возвращает невалидный html, который постоянно меняется:
```c#
string html = @"Hello,
      	<b id='_name'>World</b>!
      	</i>
      	<i id='_name'>Value1</i> Value2
      	</i>
      	<i id='id3'>Value3</i><i id='val'>Value4</i>
      	</b>";
```
В такой ситуации использование регулярных выражений практически невозможно. 
Решение: загрузить html в HtmlDocument. 
```c#
public static IHTMLDocument2 Parse(string html)
{
    	HTMLDocumentClass doc = new mshtml.HTMLDocumentClass();
    	(doc as IHTMLDocument2).write("");  // чтобы создать body и ...
    	MshtmlMarkupServices srv = new MshtmlMarkupServices(doc as IMarkupServicesRaw);
    	var res = srv.ParseString(html);
    	return res.Document;
}
```
С его помощью поставленная выше задача решается достаточно просто:
```c#
IHTMLDocument2 doc = Parse(html);
var linq = from tag in (doc.body.children as IHTMLElementCollection).Cast<IHTMLElement>()
           	where tag.id != null && (tag.id.StartsWith("_") || char.IsDigit(tag.id[tag.id.Length-1]))
           	select tag;

foreach (IHTMLElement tag in linq)
    	System.Diagnostics.Trace.WriteLine(tag.innerHTML);
```
Для компиляции к проекту надо подключить сборки, которые входят в состав Microsoft Live Writer: 
<ol>
  <li>C:\Program Files\Windows Live\Writer\WindowsLive.Writer.Mshtml.dll;</li>
  <li>C:\Program Files\Windows Live\Writer\WindowsLive.Writer.Interop.Mshtml.dll</li>
</ol>
Сборка C:\Program Files\Microsoft.NET\Primary Interop Assemblies\Microsoft.mshtml.dll не используется, потому что метод IMarkupServices.ParseString импортирован таким образом, что лишает возможности передать в него строку.
