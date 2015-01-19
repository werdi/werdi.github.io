---
title: Вызов DomainService с веб-страницы
tags: [fiddler, javascript, ria services, xmlhttprequest]
src: http://cognitex.blogspot.ru/2009/04/domainservice.html
---
# Вызов DomainService с веб-страницы
Класс DomainService входит в состав .NET RIA Services и используется в качестве базового, для классов, обменивающихся данными с Silverlight-приложениями. Но данные можно передавать и получать также с помощью XMLHttpRequest.
<ol>
  <li>Создать проект: ASP.NET Web Application</li>
  <li>Добавить в References: System.Web.Ria и System.Web.DomainServices</li>
  <li>Добавить в проект файл Class1.cs (см. ниже)</li>
</ol>
<ol start="4">
  <li>В Web.config в httpHandlers добавить:</li>
</ol>
  ```xml
  <add path="DataService.axd" verb="GET,POST" 
       type="System.Web.Ria.DataServiceFactory, System.Web.Ria, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" 
       validate="false"/>
  ```
  <ol start="5">
  <li>В Default.aspx заменить содержимое (см. ниже)</li>
  <li>Запустить приложение. В IE откроется веб-страница с двумя кнопками Operation1 и Operation2. В результате нажатия на кнопку будет создан POST запрос, который попадет в соответствующий метод в классе Test.
  Результат запроса возвращается в JSON-формате:</li>
</ol>
```json
  {"__type":"DataServiceResult:DomainServices", "IsDomainServiceException":false, "ReturnValue":"Hello 123"}
```
[Class1.cs]
```c#
using System.Web.DomainServices;
using System.Web.Ria;
using System;

namespace Cognitex.Web
{
    	[EnableClientAccess]
    	public class Test : DomainService
    	{
        	[ServiceOperation]
        	public string Operation1()
        	{
            	return "Ticks: " + DateTime.Now.Ticks;
        	}

        	[ServiceOperation]
        	public string Operation2(string value)
        	{
            	return "Hello " + value;
        	}
    	}
}
```
[Default.aspx]
```aspx
<%@ Page Language="C#" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head id="Head1" runat="server">
    	<title></title>

    	<script language="javascript" type="text/javascript">
        	function Operation1() {
            	var r = new XMLHttpRequest();
            	r.open("POST", "/DataService.axd/Cognitex-Web-Test/Operation1", false);
            	r.setRequestHeader("Content-Type", "text/json");
            	r.send("[]");
            	return r;
        	}
        	function Operation2(value) {
            	var r = new XMLHttpRequest();
            	r.open("POST", "/DataService.axd/Cognitex-Web-Test/Operation2", false);
            	r.setRequestHeader("Content-Type", "text/json");
            	r.send("[\"" + value + "\"]");
            	return r;
        	}
        	function Trace(r) {
            	var str = "";
            	for (var o in r) str += o + ": " + r[o] + "<br/>";
            	_Msg.innerHTML = str + "<hr/>" + eval("res=" + r.responseText).ReturnValue;
        	}
    	</script>

</head>
<body>
    	<button onclick="Trace(Operation1())">Operation1</button>
    	<br />
    	<input id="_Data" value="123" /><button onclick="Trace(Operation2(_Data.value))">Operation2</button>
    	<div id="_Msg" style="margin-top: 50px" />
</body>
</html>
```
Чтобы вызвать DomainService из [Fiddler'а] (http://cognitex.blogspot.com/2009/04/aspnet-development-server.html): 
<ol>
  <li>открыть закладку Request Builder</li>
  <li>в выпадающем списке выбрать POST</li>
  <li>указать адрес метода, например, http://ipv4.fiddler:5966/DataService.axd/Services-Test/Operation1</li>
  <li>в поле Request Headers указать Content-Type: text/json</li>
  <li>в поле Request Body указать</li>
  <li>нажать Execute.</li>
</ol>
Для вызова метода Operation2 в поле Request Body надо указать, например, ["wow"]





