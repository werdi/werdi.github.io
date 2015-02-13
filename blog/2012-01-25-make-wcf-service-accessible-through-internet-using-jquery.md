---
title: Правильная работа с WCF-сервисом из интернета с помощью jQuery $.ajax 
tags: [c#, javascript, html, web service]
src: https://social.msdn.microsoft.com/Forums/ru-RU/4846d27e-bc65-4e59-99a7-e6d19a6633c2/-wcf-jquery-ajax?forum=aspnetru
---
# Правильная работа с WCF-сервисом из интернета с помощью jQuery $.ajax
*> Правильная работа с WCF-сервисом из интернета [...] Метод обьявлен так: [OperationContract] public string Method1(string p1, string p2, string p3, string p4)*
надо указать WebGet и в UriTemplate указать имена параметров в фигурных скобках.
примерно так:
[Data.svc]
```
<%@ ServiceHost Service="Test.Provider"
      Factory="System.ServiceModel.Activation.WebServiceHostFactory" %>
```
[Test.cs]
```c#
using System; 
using System.ServiceModel; 
using System.ServiceModel.Web; 

namespace Test 
{ 
  [ServiceContract] 
  public class Provider 
  { 
    [OperationContract] 
    [WebGet(UriTemplate = "/M1/", ResponseFormat = WebMessageFormat.Xml)] 
    public long Now() { return DateTime.Now.Ticks; } 

    [OperationContract] 
    [WebGet(UriTemplate = "/M2/{value}={key}")] 
    public string GetSome(string key, string value) 
    { 
     return "hello: " + key + " " + value; 
    } 
  } 
}
```
[js]
```javascript
<script language="javascript"> 
function Get(uri) { 
  var x = new XMLHttpRequest(); 
  x.open("GET", uri, false, null, null); 
  x.send(); 
  return x.responseText; 
} 
</script>
```
[html]
```html
<a href="/Data.svc/M1">test1</a>
<br />
<a href="/Data.svc/M2/id=123">test2</a>
<button onclick="alert(Get('http://localhost:1799/Data.svc/M1'))">test3</button>
```

p.s.
пример вызова метода из c# см. [здесь] (http://social.msdn.microsoft.com/Forums/eu/wpf/thread/0e058644-db73-49a4-b7cd-2af5fe8b9400/#d9a2ec63-8444-49e0-8220-8042d3b3a0f1)
