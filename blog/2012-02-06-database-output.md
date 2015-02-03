---
title: Возможно ли такой вывод из базы?
tags: [c#, web, data binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/88070817-49ee-405d-b3f9-3a8738c75de0/-?forum=fordataru
---
# Возможно ли такой вывод из базы?
*> Можно ли выводить свойства как отдельные элементы с помощью контрола bulletlist, то есть так а1 а2 а3*
[test.aspx]
```aspx
...
<asp:BulletedList ID="bl" runat="server" />
...
```
[test.aspx.cs]
```c#
using System;

namespace WebApplication1
{
  public partial class Test : System.Web.UI.Page
  {
    protected void Page_Load(object sender, EventArgs e)
    {
      var data = new { Product = "А1", Props = "а1 а2 а3" };
      bl.DataSource = data.Props.Split(new[] { ' ' }, StringSplitOptions.RemoveEmptyEntries);
      bl.DataBind();
    }
  }
}
```
или
```c#
using System;
using System.Data;

namespace WebApplication1
{
  public partial class Test : System.Web.UI.Page
  {
    protected void Page_Load(object sender, EventArgs e)
    {
      var data = LoadData();
      bl.DataSource = ((string)data.Rows[0]["Props"]).Split(new[] { ' ' }, StringSplitOptions.RemoveEmptyEntries);
      bl.DataBind();
    }
    DataTable LoadData()
    {
      var dt = new DataTable();
      dt.Columns.Add("Id");
      dt.Columns.Add("Props");
      dt.Rows.Add("А1", "а1 а2 а3");
      return dt;
    }
  }
}
```
*> Я так понял что это будет работать только для определенного продукта*

BulletedList можно привязать к любой коллекции.
если в DataTable загружаются строки: [имя продукта | параметры ]
и данные надо вывести в виде:
  имя продукта:
  - параметр 1
  - ...
  - параметр N

то можно использовать Repeater:
```aspx
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Test.aspx.cs" Inherits="WebApplication1.Test" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title></title>
</head>
<body>
  <form runat="server">
  <asp:Repeater runat="server" ID="prods">
    <ItemTemplate>
      <asp:Label ID="Label1" runat="server" Text='<%# Eval("Id") %>' />
      <asp:BulletedList ID="bl" runat="server" DataSource='<%# Split(Eval("Props")) %>' />
      <br />
    </ItemTemplate>
  </asp:Repeater>
  </form>
</body>
</html>
```
```c#
using System;
using System.Data;

namespace WebApplication1
{
  public partial class Test : System.Web.UI.Page
  {
    protected void Page_Load(object sender, EventArgs e)
    {
      var data = LoadData();
      prods.DataSource = data;
      prods.DataBind();
    }
    public object Split(object data)
    {
      var str = data as string;
      if (str == null) return null;
      return str.Split(new[] { ' ' }, StringSplitOptions.RemoveEmptyEntries);
    }
    DataTable LoadData()
    {
      var dt = new DataTable();
      dt.Columns.Add("Id");
      dt.Columns.Add("Props");
      dt.Rows.Add("А1", "а1 а2 а3");
      dt.Rows.Add("B1", "b1 b2 b3");
      return dt;
    }
  }
}
```
