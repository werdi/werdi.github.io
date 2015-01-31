---
title: Маркеры BulletedList
tags: [asp.net, c#, html, css]
src: https://social.msdn.microsoft.com/Forums/ru-RU/c7d6391c-47f3-42ba-947e-57cca5203e31/-bulletedlist?forum=aspnetru
---
# Маркеры BulletedList
*> Как можно маркеры передвигать к тексту BulletedList?* 

в .bull надо добавить float:left;
```css
.bull
{
  padding: 0px 0px 0px 0px;
  float: left;
}
```
работает при <!DOCTYPE html> 

*> ... я сделал как вы сказали, но BulletedList поместился под картинкой. Посмотрите я создал "чистый проект" [...] .bull{padding:0px 0px 0px 0px;} [...] <asp:BulletedList ID="BulletedList1" runat="server">* 

в asp:BulletedList не указан css. надо добавить CssClass 
```asp
<asp:BulletedList ID="BulletedList1" runat="server" CssClass="bull">
...
</asp:BulletedList>
```
и в .bull надо добавить float:left; 
```css
.bull
{
 padding: 0px 0px 0px 30px;
 float: left;
}
```
p.s.
как вариант используйте table: 
```asp
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head id="Head1" runat="server">
  <title></title>
</head>
<body>
  <form id="form1" runat="server">
  <table>
    <tr>
      <td><img src="1.png" alt="" /></td>
      <td style="padding-left: 20px;">
        <asp:BulletedList ID="BulletedList1" runat="server">
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
          <asp:ListItem>some text</asp:ListItem>
        </asp:BulletedList>
      </td>
    </tr>
  </table>
  </form>
</body>
</html>
```

*> нужно обтекание картинки списком*

для .bull надо указать list-style-position 

![img1] (http://social.msdn.microsoft.com/Forums/getfile/89921)

```asp
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head id="Head1" runat="server">
  <title></title>
  <style type="text/css">
    .img
    {
      width: 50px;
      height: 50px;
      border: 1px solid red;
      float: left;
      margin-right: 10px;
    }
    .bull
    {
      list-style-position: inside;
      margin-left: -40px;
    }
  </style>
</head>
<body>
  <form id="form1" runat="server">
  <div>
    <img src="1.png" class="img" />
    <asp:BulletedList ID="BulletedList1" runat="server" CssClass="bull">
      <asp:ListItem>some text</asp:ListItem>
      <asp:ListItem>some text</asp:ListItem>
      <asp:ListItem>some text</asp:ListItem>
      <asp:ListItem>some text</asp:ListItem>
      <asp:ListItem>some text</asp:ListItem>
      <asp:ListItem>some text</asp:ListItem>
    </asp:BulletedList>
  </div>
  </form>
</body>
</html>
```
