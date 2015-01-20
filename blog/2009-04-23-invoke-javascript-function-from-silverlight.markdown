---
title: Вызов JavaScript функции из Silverlight
tags: [javascript, silverlight, html, c#]
src: http://cognitex.blogspot.ru/2009/04/javascript-silverlight.html
---
# Вызов JavaScript функции из Silverlight
Представим себе, что требуется периодически обращаться к сервису, например, за какими-то зашифрованными данными, обрабатывать их, результат выводить на страницу.
Понятно, что JavaScript в такой ситуации не поможет, но можно использовать Silverlight. Для этого надо: 
1) в .html- или .aspx-файл в тег body добавить "невидимый" Silverlight:
```html
<object data="data:application/x-silverlight-2," type="application/x-silverlight-2" style="width: 0px; height: 0px;">
      	<param name="source" value="ClientBin/App.xap" /></object>
```
Ширина и высота равны 0. Если указать style="display: none;", то Silverlight не загрузится.
2) в тег head добавить функцию, которая вызывается из Silverlight:
```javascript
<script type="text/javascript">
    	function ShowResult1(val) {
    	}
</script>
```
3) в Silverlight-приложение добавить, например, следующий код:
```c#
ThreadPool.QueueUserWorkItem((state) =>
{
    	for (int i = 0; i < 1000; i++)
    	{
        	// имитация длительной обработки
        	Thread.Sleep(1000);
        	// передача данных в JavaScript функцию
        	this.Dispatcher.BeginInvoke(() => HtmlPage.Window.Invoke("ShowResult1", DateTime.Now.Ticks));
    	}
});
```
В результате открытия веб-страницы в браузере произойдет загрузка Silverlight-приложения, в котором будет запущен рабочий поток с циклом на 1000 итераций. При каждой итерации данные передаются в JavaScript-функцию на веб-странице.
