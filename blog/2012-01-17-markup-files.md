---
title: Два файла разметки, один файл cs
tags: [c#, aspx, web]
src: https://social.msdn.microsoft.com/Forums/ru-RU/79473c9e-d912-49e9-8f72-b3be6e68af3d/-cs?forum=aspnetru 
---
# Два файла разметки, один файл cs
*> Можно как то прописать логику работы серверной части из этого одного файла на два файла разметки?*

в проект WebApplication1 добавить файл, например, Shared.cs
```c#
namespace WebApplication1.Shared
{
  public class Test
  {
    public string Run(string name)
    {
      return "hello, " + name;
    }
  }
}
```
метод Run можно вызвать из любого aspx-файла:
```aspx
<% var t = new WebApplication1.Shared.Test(); %>
<%= t.Run("test1") %>
```
*> Есть две страницы с разными разметками [...] а у них(этих страниц серверный код точь в точь один и тот же - внутри *.cs)*

можно создать базовый класс и наследовать страницы от него.
примерно так:
```c#
public class BasePage : System.Web.UI.Page
{
  protected override void OnLoad(EventArgs e)
  {
    base.OnLoad(e);
    this.Controls.Add(new Label { Text = "BASE" });    
  }
}
```
[Test1.aspx.cs]
```c#
using System;
using System.Web.UI.WebControls;

namespace WebApplication1
{
  public partial class Test1 : BasePage
  {
    protected void Page_Load(object sender, EventArgs e)
    {
      this.Controls.Add(new Label { Text = "Test1" });
    }
  }
}
```
[Test1.aspx]
```aspx
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Test1.aspx.cs" Inherits="WebApplication1.Test1" %>
...
```
