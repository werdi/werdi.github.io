---
title: Кнопка, передача информации от страницы к странице + javascript
tags: [c#, html, aspx]
src: https://social.msdn.microsoft.com/Forums/ru-RU/08dca7a9-8c7d-4dfb-a6ce-5b4fc6f091f7/-javascript?forum=aspnetru 
---
# Кнопка, передача информации от страницы к странице + javascript
*> Есть кнопка [...] как можно сделать, чтобы в этом случае также сработал серверный код для кнопки?*
```aspx
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Test1.aspx.cs" Inherits="WebApplication1.Test1" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
  <title></title>
  <script type="text/javascript">
    function openWin()
    {
      alert(1);
    }
  </script>
</head>
<body>
  <form runat="server">
  <asp:Button OnClick="Test_Click" runat="server" Text="Test" ID="tst" OnClientClick="openWin()" />
  </form>
</body>
</html>
```
```c#
using System;

namespace WebApplication1
{
  public partial class Test1 : System.Web.UI.Page
  {
    protected void Test_Click(object sender, EventArgs e)
    {
    }
  }
}
```
*> Все равно не передается информация, просто открывается страница*

при нажатии на кнопку вызывается функция openWin(), затем на сервер посылается запрос.
какая информация должна передаваться на сервер?

*> чтобы сначала выполнялась серверная часть, а потом функция openWin()*

[Test1.aspx]
```aspx
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Test1.aspx.cs" Inherits="WebApplication1.Test1" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
  <title></title>
  <script type="text/javascript">
    function openWin()
    {
      setTimeout("alert(1)", 1000);
    }
  </script>
</head>
<body>
  <form runat="server">
  <asp:TextBox runat="server" ID="tb" Text="0" />
  <asp:Button OnClick="Test_Click" runat="server" Text="Test" ID="tst" />
  <a href="/Test2.aspx">test2</a>
  </form>
</body>
</html>
```

[Test1.aspx.cs]
```c#
using System;

namespace WebApplication1
{
  public partial class Test1 : System.Web.UI.Page
  {
    protected void Test_Click(object sender, EventArgs e)
    {
      tb.Text += "+1";
      ClientScript.RegisterClientScriptBlock(this.GetType(), "openWin", "openWin()", true);
      this.Session["someid"] = tb.Text;
    }
  }
}
```
[Test2.aspx]
```aspx
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Test2.aspx.cs" Inherits="WebApplication1.Test2" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
  <title></title>
</head>
<body>
  <%= this.Session["someid"] %>
</body>
</html>
```
[Test2.aspx.cs]
```c#
using System;
using System.Web.UI;

namespace WebApplication1
{
  public partial class Test2 : Page
  {
  }
}
```
