---
title: Control Properties Stored Database Idea
tags: [c#, xaml, wpf, web service]
src: https://social.msdn.microsoft.com/Forums/ru-RU/0e058644-db73-49a4-b7cd-2af5fe8b9400/control-properties-stored-database-idea?forum=wpf
---
# Control Properties Stored Database Idea
*> I'm wondering if it is a good idea to store XAML control properties in the database.*

so you will need to download the properties and insert in xaml or in controls. 
much easier to store xaml in Sql Server in the column of xml type.
then you can request database, get xaml and create controls from xaml with [XamlReader](http://msdn.microsoft.com/en-us/library/system.windows.markup.xamlreader.aspx)
another way is to store assembly with compiled controls.

*What I was referring to is just the values for the properties stored. So my XAML would look like:<TextBlock Text="{Binding QuestionText}" ...*

as i said above we can request xaml and update UI.
below is example of rest-service which returns xaml ...
 
to compile the code you have to start Visual Studio in Administrator mode (or change the service settings using netsh, see [here](http://blogs.msdn.com/b/amitlale/archive/2007/01/29/addressaccessdeniedexception-cause-and-solution.aspx)).
then ensure in project properties, that the framework's target is .NET Framework 4.0 (not Client Profile).
also in project add references: System.ServiceModel.dll, System.ServiceModel.Web.dll
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="400" Width="500">
  <Grid>
    <Button  Content="Load Xaml" Click="Button_Click" HorizontalAlignment="Left" VerticalAlignment="Top" />
    <ListBox x:Name="trg" ScrollViewer.VerticalScrollBarVisibility="Visible" Margin="0,30,0,0" />
  </Grid>
</Window>
```
```c#
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.ServiceModel;
using System.ServiceModel.Web;
using System.Windows;
using System.Windows.Markup;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      var host = new WebServiceHost(new Service(), new Uri("http://localhost:8899"));
      host.Open();
      this.Closing += (s, e) => host.Close();

      this.DataContext = 123456789;   // test
    }

    private void Button_Click(object sender, RoutedEventArgs _)
    {
      Func<string, object> Xaml = str =>
      {
        var pc = new ParserContext();
        pc.XmlnsDictionary.Add("", "http://schemas.microsoft.com/winfx/2006/xaml/presentation");
        pc.XmlnsDictionary.Add("x", "http://schemas.microsoft.com/winfx/2006/xaml");
        return XamlReader.Parse(str, pc);
      };

      Action<object, DownloadStringCompletedEventArgs> cb = (s, a) =>
      {
        var o = Xaml(a.Result);
        var e = o as IEnumerable<object>;
        if (e == null)
          trg.Items.Add(o);
        else
          foreach (var itm in e)
            trg.Items.Add(itm);
      };

      var c = new WebClient();
      c.DownloadStringCompleted += (s, a) => cb(s, a);
      c.DownloadStringAsync(new Uri("http://localhost:8899/xaml/textbox"));
    }
  }

  [ServiceContract, ServiceBehavior(InstanceContextMode = InstanceContextMode.Single)]
  public class Service
  {
    [OperationContract, WebGet(UriTemplate = "/xaml/{id}", ResponseFormat = WebMessageFormat.Xml)]
    public Stream Load(string id)
    {
      var ret = @"<TextBlock Text=""{Binding StringFormat='binding: {0}; server response: " 
        + DateTime.Now.Millisecond + @"'}"" />";
      return ret.ToStream();
    }
  }

  public static class StringHelper
  {
    public static Stream ToStream(this string s)
    {
      var ret = new MemoryStream();
      var bytes = System.Text.Encoding.Default.GetBytes(s);
      ret.Write(bytes, 0, bytes.Length);
      ret.Position = 0L;
      return ret;
    }
  }
}
```
 
