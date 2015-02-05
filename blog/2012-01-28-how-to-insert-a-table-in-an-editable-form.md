---
title: HTML составная форма или как вставить редактируемую таблицу в форму
tags: [html, jquery]
src: https://social.msdn.microsoft.com/Forums/ru-RU/300461d8-91e5-4169-8f16-d0286150ca7e/html-?forum=aspnetru
---
# HTML составная форма или как вставить редактируемую таблицу в форму
*> Есть делание вставить таблицу с функцией Ajax-редактирования [...] Submit для строки формы перстает работать. Вопрос в том, как бы это обойти?*

с помощью JQuery. примерно так:
```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta http-equiv='X-UA-Compatible' content='IE=9' />
    <style type="text/css">
      td { border: 1px dotted red; }
      table, span { width: 100%; }
    </style>
    <script type="text/javascript" src='http://code.jquery.com/jquery-latest.js'></script>
    <script type="text/javascript">
        function response(d, status) {
          $('#res').html(status + ": " + d);
        }
        function clear(str) {
          return str.replace(/\x0A+/g, '').replace(/\s{2,}/g, ' ').replace(/^\s+|\s+$/g, "");
        }
        $(document).ready(function () {
          // поместить контент td в поле-редактор
          $('#tbl .editable').html(function (index, old) {
            var ci = $(this).get(0).cellIndex;
            var ri = $(this).parent().get(0).rowIndex;
            return '<span contenteditable="true" id="ce'+ri+'_'+ci+'">'+clear(old)+'</span>';
          });

          var org = {};   // хранит оригинальный контент td

          // при фокусе поля сохранить его контент
          $('[contenteditable=true]').focus(function () {
            var id = $(this).attr('id');
            if ((id in org) == false)
              org[id] = $(this).text();
          });

          // поставить фокус на первое поле
          $('#tbl .editable:first > span').focus();

          // отправить изменения
          $('#frm').submit(function () {
            // найти изменения
            var req = {};
            for (var c in org) {
              var txt = $('#' + c).text();
              if (org[c] != txt)
                req[c] = clear(txt);
            }
            // отправить изменения POST-запросом на url: /dif
            jQuery.post("/dif/", req, response);
            return false;  // остановить submit
          });
        });
    </script>
</head>
<body>
  <form method='post' action='' id='frm'>
    <table id='tbl'>
      <tr>
        <td class='editable'>12</td>
        <td class='editable'>34</td>
      </tr>
      <tr>
        <td class='editable'>56</td>
        <td class='editable'>78</td>
      </tr>
    </table>
    <br />
    <input type='submit' />
    <br />
    <div id='res'></div>
  </form>
</body>
</html>
```
