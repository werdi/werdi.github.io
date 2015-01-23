---
title: Не работает View - Source в IE
tags: [ie, javascript]
src: http://mindberg.blogspot.ru/2007/06/view-source-ie.html
---
# Не работает View - Source в IE
Ситуация: иногда бывает, что IE не может или не хочет отобразить код страниц. Т.е. вызов View – Source  не работает как предполагается.

Решение:
<ol>
  <li>Создать папку, например, Z:\Projects\IE View Source</li>
  <li>Создать в ней файл: ViewSource.htm со следующим текстом:</li>
</ol>
```javascript
<html>
  <body>
    <script language="JavaScript" defer>
      var win = external.menuArguments;
      var szsource = win.document.documentElement.outerHTML;
      var wincounter = 0;
      var mywin = null;
      do
      {
        mywin = window.open("about:blank", "Source" + wincounter++,        
                            "toolbar=no,location=no,menubar=yes,status=yes,scrollbars=yes,resizable=yes");
      }
      while(mywin == null);
      mywin.document.open("text/plain");
      mywin.document.write(szsource);
      mywin.document.close();
      mywin.document.title = "Source for: " + win.location.href;
    </script>
  </body>
</html>
```
<ol start="3">
  <li>Запустить regedit; в ветке  HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\MenuExt создать Key с именем 
View Source. В Default записать: Z:\Projects\IE View Source\ViewSource.htm</li>
  <li>Открыть IE, в контекстном меню выбрать View Source.</li>
</ol>
Подробнее, см. MSDN: "Adding Entries to the Standard Context Menu" (ms-help://MS.VSCC.v80/MS.MSDN.v80/MS.WEBDEV.v10.en/ieext/workshop/browser/ext/tutorials/context.htm)

P.S.
Все сказанное выше не поможет, если IE запущен в режиме Run As с включенным Run this Program with restricted access.
