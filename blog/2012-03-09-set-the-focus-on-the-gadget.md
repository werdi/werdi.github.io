---
title: Как установить фокус на гаджет
tags: [c#, html, winforms, webbrowser]
src: https://social.msdn.microsoft.com/Forums/ru-RU/e72e1031-9b22-429a-aa18-c8953b381e9c/-?forum=aspnetru 
---
# Как установить фокус на гаджет
*> Как установить фокус не кликая?*

надо найти поле ввода и вызвать метод focus()
примерно так:
```html
<!DOCTYPE html>
<html>
<head>
  <title></title>
  <script type="text/javascript">
  function init()
  {
    document.getElementById("_txt2").focus();
  }
  </script>
</head>
<body onload="init()">
  <input type="text" id="_txt1" />
  <input type="text" id="_txt2" />
</body>
</html>
```
*> Хочу что-то сделать в гаджете по событию onmousewheel, но для того, чтобы это событие сработало, гаджет должен быть в фокусе. для установки фокуса по гаджету нужно кликнуть мышкой.*

фокус устанавливать не обязательно
onmousewheel срабатывает когда мышь находится над тегом.
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var html = @"
        <!doctype html>
        <html>
        <head>
          <script type='text/javascript'>
          var c = 0;
          function wheel() {
            document.getElementById('res').innerHTML = c++;
          }
          function focusing(over) {
            var el = document.getElementById('res');
            el.innerHTML = over ? 'wheel now' : '';
          }
          </script>
        </head>
        <body>
          <div onmouseout='focusing(false)' 
            onmouseover='focusing(true)' 
            onmousewheel='wheel()' id='mainsquare' 
            style='position:absolute; top: 100px; width:200px; height:50px; background-color:silver;'>
              <div style='margin:5px;width:100px; background-color:gray;height:30px;'></div>
            </div>
          <div id='res'></div>
        </body>
        </html>";
      new WebBrowser { DocumentText = html, Parent = this, Dock = DockStyle.Fill };
    }
  }
}
```
