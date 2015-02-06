---
title: Элементы из папки для listview
tags: [c#, asp, linq, listview]
src: https://social.msdn.microsoft.com/Forums/ru-RU/6d6e7f12-1d46-4ea8-b298-8e90578b09b6/-listview?forum=aspnetru
---
# Элементы из папки для listview
*> Есть listview, который привязан к базе данных. Как можно сделать так чтобы для каждого элемента listview до вывода информации из базы данных была картинка из папки.*
```c#
using System;
using System.Collections.Generic;
using System.Linq;

namespace WebApplication1
{
  public partial class _Default : System.Web.UI.Page
  {
    protected void Page_Load(object sender, EventArgs e)
    {
      if (this.IsPostBack)
        return;

      var values = LoadData();
      var imgs = new[] { "~/imgs/p1.png", "~/imgs/p2.gif", "~/imgs/p3.jpg" };

      _lst.DataSource = values
        .OrderBy(v => v) 
        .Zip(imgs, (v, i) => new { Value = v, Img = i });

      _lst.DataBind();
    }

    IEnumerable<int> LoadData() // загрузка из бд или MemoryCache
    {
      return new [] { 20, 10 };
    }
  }
}
```
```asp
<%@ Page Title="Home Page" Language="C#" MasterPageFile="~/Site.master" AutoEventWireup="true"
  CodeBehind="Default.aspx.cs" Inherits="WebApplication1._Default" %>

<asp:Content ID="HeaderContent" runat="server" ContentPlaceHolderID="HeadContent" />
<asp:Content ID="BodyContent" runat="server" ContentPlaceHolderID="MainContent">
  <asp:ListView runat="server" ID="_lst">
    <ItemTemplate>
      <%# Eval("Value") %>,
      <asp:Image ImageUrl='<%# Eval("Img") %>' runat="server" />
    </ItemTemplate>
    <ItemSeparatorTemplate>
      <br />
    </ItemSeparatorTemplate>
  </asp:ListView>
</asp:Content>
```
