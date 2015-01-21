---
title: Вызов REST сервиса с веб-страницы
tags: [rest, services, soap, web, html, javascript]
src: http://cognitex.blogspot.ru/2009/03/rest.html
---
# Вызов REST сервиса с веб-страницы
Например, REST-служба доступна по адресу http://localhost:1799/Test2.svc и в ней определен WebGet-метод GetData, который принимает какое-то значение.
Чтобы вызвать GetData c web-страницы пишем следующее:
```html
<button onclick="alert(GetREST('http://localhost:1799/Test2.svc/GetData/2'))">REST</button>
```
...
```javascript
<script language="jscript">
function GetREST(uri)
{
  var x = new XMLHttpRequest();
  x.open("GET", uri, false, null, null);
  x.send();
  return x.responseText;
}
</script>
```
Очевидно, что работать с REST намного проще, чем с SOAP.
