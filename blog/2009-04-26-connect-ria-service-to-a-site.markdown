---
title: Подключение RIA-сервиса к сайту
tags: [asp.net, ria services, xml, aspx, c#]
src: http://cognitex.blogspot.ru/2009/04/ria.html
---
# Подключение RIA-сервиса к сайту
Ситуация: на сервере работает ASP.NET Web Application. Требуется подключить к нему RIA-сервис. 
Для этого надо: 
<ol>
  <li>в папку \bin сайта добавить сборки (см. C:\Program Files\Microsoft SDKs\RIA Services\v1.0\Libraries\Server\):</li>
    <ul>
      <li>System.Web.Ria.dll;</li>
      <li>System.Web.DomainServices.dll;</li>
      <li>System.ComponentModel.DataAnnotations.dll;</li>
    </ul>
</ol> 
<ol start="2">
  <li>в файл web.config в тег httpHandlers добавить:</li>
</ol>
```xml
<add path="DataService.axd" verb="GET,POST" type="System.Web.Ria.DataServiceFactory, System.Web.Ria, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" validate="false"/>
```
Примечание: изменения web.config и добавление/удаление файлов из папки \bin приводит к остановке веб-приложения, но приложение запускается при первом же запросе. 
В этом можно убедиться, если в проект сайта добавить страницу (.aspx-файл) со следующим кодом:
```aspx
<%@ Page Language="C#"  %>

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    	<meta http-equiv="Refresh" content="1; url=/" />
</head>
<body>
    	<%= DateTime.Now.Ticks %><br />
    	Domain.Id:
    	<%= AppDomain.CurrentDomain.Id %>
</body>
</html>
```
RIA-сервис - это проект типа Class Library; у которого в References добавлены сборки, указанные выше. 
Ниже код RIA-сервиса:
```c#
using System.Web.DomainServices;
using System.Web.Ria;

namespace ClassLibrary1
{
    	[EnableClientAccess()]
    	public class DomainService1 : DomainService
    	{
        	[ServiceOperation]
        	public string GetData(string id)
        	{
            	return "Data: " + id;
        	}
    	}
}
```
После компиляции полученную сборку, например, ClassLibrary1.dll надо также скопировать в папку \bin сайта.
RIA-сервис готов к работе. 
Если сайт расположен по адресу http://myhost:3456, то адрес метода GetData - http://myhost:3456/DataService.axd/ClassLibrary1-DomainService1/GetData
Выполнить проверку работы RIA-сервиса можно с помощью Fiddler'а (см. [здесь] (http://cognitex.blogspot.com/2009/04/domainservice.html)).

C RIA-сервисами в отличие от других типов сервисов никаких [сложностей на хостинге] (http://cognitex.blogspot.com/2009/04/blog-post_5560.html) не возникает.


