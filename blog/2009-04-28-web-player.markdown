# Веб-проигрыватель
Например, при загрузке веб-страницы в браузер надо начать проигрывание .mp3-файла. Сделать это можно с помощью Silverlight:
```html
<html xmlns="http://www.w3.org/1999/xhtml" >
<head>
    	<title></title>
    	<script type='text/xaml' id="Ag">
        	<?xml version='1.0' encoding='utf-8' ?>
        	<MediaElement
            	xmlns='http://schemas.microsoft.com/winfx/2006/xaml/presentation'
            	xmlns:x='http://schemas.microsoft.com/winfx/2006/xaml'
            	x:Name='media' Source='http://ipv4.fiddler:4139/Song.mp3' />
    	</script>
</head>
<body>
    	<object data="data:application/x-silverlight-2," type="application/x-silverlight-2" style="height: 0px; width: 0px;">
        	<param name="source" value="#Ag" />
    	</object>
</body>
</html>
```
На странице есть вызов Silverlight, которому передается xaml, записанный в тег script. 
Silverlight поддерживает [Streaming] (http://msdn.microsoft.com/en-us/library/cc189080(VS.95).aspx) и множество форматов, а также [Server-Side Playlists] (http://msdn.microsoft.com/en-us/library/cc645037(VS.95).aspx).
Про адрес ipv4.fiddler - [здесь] (http://cognitex.blogspot.com/2009/04/aspnet-development-server.html).
