---
title: Обновление xml файла
tags: [c#, xml, aspx, winforms, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7610dd8b-197e-49f7-b809-e0707bcbcdbe/-xml-?forum=aspnetru
---
# Обновление xml файла
*> Есть xml файл [...] нужно обновлять "столбик" res каждый раз на +1*
```c#
using System.Data;
using System.IO;
using System.Linq;
using System.Windows.Forms;
using System.Xml.Linq;
...
var xml = @"<Answers>
  <answer ID='1' res='10' />
  <answer ID='2' res='10' />
  <answer ID='3' res='25' />
</Answers>"; 

// v1
var ds = new DataSet();
ds.ReadXml(new StringReader(xml));
foreach(DataRow row in ds.Tables["answer"].Rows)
  row["res"] = int.Parse((string)row["res"]) + 1;
System.Diagnostics.Trace.WriteLine(ds.GetXml());

// v2
var xe = XElement.Parse(xml);
foreach(var a in xe.Elements("answer").Select(x => x.Attribute("res")))
  a.SetValue(int.Parse((string)a.Value) + 1);
System.Diagnostics.Trace.WriteLine(xe);
```
*> "~/XMLFile.xml" ... пишет "Недопустимые данные на корневом уровне."

xml с ошибкой. в xml должен быть только один тег, все остальные вложенные.

*> как можно сделать так, чтобы обновилась определенная строка ( скажем строка с ид 3)*
```c#   
var ds = new DataSet();
ds.ReadXml(new StringReader(xml));
var row = ds.Tables["answer"].Select("ID = '3'")[0];
row["res"] = int.Parse((string)row["res"]) + 1;
System.Diagnostics.Trace.WriteLine(ds.GetXml());
```
*В xml файле только один корневой элемент (первый пост)*

проверял на следующем xml:
```xml
<?xml version="1.0" encoding="utf-8" ?>
<Answers>
	<answer ID="1" Name="" res="10"/>
	<answer ID="2" Name="" res="10"/>
	<answer ID="3" Name="" res="25"/>
</Answers>
```
*> У меня тот же файл [...] ds.ReadXml(New StringReader(xml)) [...] может быть я не правильно подключаю xml файл?*

для чтения из файла надо вызвать метод .ReadXml(путь к файлу);

*> <asp:XmlDataSource ID="XmlDataSource1" runat="server" DataFile="~/XMLFile2.xml">...*

[App_Data\test.xml] 
```xml
<?xml version="1.0" encoding="utf-8" ?>
<Answers>
	<answer ID="1" Name="" res="10"/>
	<answer ID="2" Name="" res="10"/>
	<answer ID="3" Name="" res="25"/>
</Answers>
``` 
[default.aspx]
```aspx 
<%@ Page Title="Home Page" Language="C#" MasterPageFile="~/Site.master" AutoEventWireup="true"
    CodeBehind="Default.aspx.cs" Inherits="WebApplication1._Default" %>

<asp:Content ID="HeaderContent" runat="server" ContentPlaceHolderID="HeadContent">
</asp:Content>
<asp:Content ID="BodyContent" runat="server" ContentPlaceHolderID="MainContent">
  <asp:XmlDataSource ID="xml" runat="server" DataFile="~/App_Data/test.xml" />
  <asp:DataGrid runat="server" DataSourceID="xml" />
  <asp:DataGrid runat="server" ID="dg2" />
</asp:Content>
```
[default.aspx.cs]
```c#
using System;
using System.Data;

namespace WebApplication1
{
  public partial class _Default : System.Web.UI.Page
  {
    protected void Page_Load(object sender, EventArgs e)
    {
      var ds = new DataSet();
      ds.ReadXml(Server.MapPath("~/App_Data/test.xml"));
      dg2.DataSource =  ds;
      dg2.DataMember = "answer";
      dg2.DataBind();
    }
  }
}
```
