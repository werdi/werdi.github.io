---
title: How to display multiple entries from xml to dropdownlist
tags: [c#, winforms, asp.net, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f30e5a7f-c724-4780-bc26-659f46cb9e13/how-to-display-multiple-entries-from-xml-to-dropdownlist?forum=xmlandnetfx
---
# How to display multiple entries from xml to dropdownlist
*> how can i display multiple mobile numbers to a dropdownlist when i select Team_A below*
```c#
using System.Data;
using System.IO;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var xml = @"
        <contacts_list>
          <contacts>
            <id>1</id>
            <sender>TEAM_A</sender>
            <mobile>A123456789</mobile>
            <mobile>A123456789</mobile>
          </contacts>
          <contacts>
            <id>2</id>
            <sender>TEAM_B</sender>
            <mobile>B123456789</mobile>
            <mobile>B123456789</mobile>
          </contacts>
        </contacts_list>";

      var ds = new DataSet();
      ds.ReadXml(new StringReader(xml));

      new ComboBox
      {
        Parent = this,
        Dock = DockStyle.Top,
        DisplayMember = "contacts.sender",
        DataSource = ds,
      };
      new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        DataMember = "contacts.contacts_mobile",
        DataSource = ds,
      };
    }
  }
}
```
*> I tried to put it my Default.aspx.cs but it doesn't recognize the DisplayMember.*

oh, seems you need solution for asp.net 
you can bind control to data something like this
```
<asp:DropDownList ID="ddl" runat="server" />  ... var ds = new DataSet();
ds.ReadXml(new StringReader(xml));
            
ddl.DataSource = ds;
ddl.DataMember =  "contacts";
ddl.DataTextField = "sender";
ddl.DataBind();
```
*> its already working but what happens when select TEAM A I tried below but it doesnt work*

try the following example.
```
<%@ Page Title="Home Page" Language="C#" MasterPageFile="~/Site.master" AutoEventWireup="true"
  CodeBehind="Default.aspx.cs" Inherits="WebApplication1._Default" %>

<asp:Content ID="HeaderContent" runat="server" ContentPlaceHolderID="HeadContent">
</asp:Content>
<asp:Content ID="BodyContent" runat="server" ContentPlaceHolderID="MainContent">
  <asp:DropDownList ID="dd1" runat="server" AutoPostBack="true" OnSelectedIndexChanged="dd1_SelectedIndexChanged" />
  <asp:DropDownList ID="dd2" runat="server" />
</asp:Content>
```
```c# 
using System;
using System.Collections;
using System.Linq;
using System.Web.Caching;
using System.Web.UI;
using System.Xml.Linq;
using System.Xml.XPath;

namespace WebApplication1
{
  public partial class _Default : System.Web.UI.Page
  {
    Hashtable Data
    {
      get
      {
        var key = "contacts_list";
        var data = Cache[key];
        if(data != null) return (Hashtable) data;
        var xml = @"
        <contacts_list>
          <contacts>
            <id>1</id>
            <sender>TEAM_A</sender>
            <mobile>A123456789</mobile>
            <mobile>A123456789</mobile>
          </contacts>
          <contacts>
            <id>2</id>
            <sender>TEAM_B</sender>
            <mobile>B123456789</mobile>
            <mobile>B123456789</mobile>
          </contacts>
        </contacts_list>";
        var xe = XElement.Parse(xml);
        var ret = new Hashtable(StringComparer.OrdinalIgnoreCase);
        foreach(var xn in xe.XPathSelectElements("contacts"))
        {
          var sender = xn.Element("sender").Value;
          var id = xn.Element("id").Value;
          var phones = xn.Elements("mobile").Select(m => new { Mobile = m.Value });
          ret.Add(id, new { Sender = sender, Phones = phones, Id = id }); 
        }
        Cache.Insert(key, ret);
        return ret;
      }
    }

    protected void Page_Load(object sender, EventArgs e)
    {
      if (this.IsPostBack == false)
      {
        dd1.DataSource = this.Data.Values;
        dd1.DataTextField = "Sender";
        dd1.DataValueField = "Id";
        dd1.DataBind();
        this.UpdatePhones();
      }
    }

    public void dd1_SelectedIndexChanged(object sender, EventArgs e)
    {
      this.UpdatePhones();
    }

    private void UpdatePhones()
    {
      dd2.DataSource = DataBinder.GetPropertyValue(this.Data[dd1.SelectedValue], "Phones");
      dd2.DataTextField = "Mobile";
      dd2.DataBind();
    }
  }
}
```
*> can i use this Server.MapPath("~/App_Data/sender_list.xml and maybe load the xml to var?*

yes, you can use
```
var xe = XElement.Load(Server.MapPath("~/App_Data/sender_list.xml"));
``` 
instead of
```
var xml = @"... 
var xe = XElement.Parse(xml);
```
also you can create cache dependency with using
```
Cache.Insert(key, ret, new CacheDependency(Server.MapPath("~/App_Data/sender_list.xml")));
```
instead of 
```
Cache.Insert(key, ret);
```
*> it doesn't display the contents of my xml file•var xe = XElement.Load(Server.MapPath("~/App_Data/sender_list.xml"));*

i have tested the code. works fine for me. please, make sure the file sender_list.xml exists in the Solution Explorer, folder /App_Data; also make sure the file content exists.