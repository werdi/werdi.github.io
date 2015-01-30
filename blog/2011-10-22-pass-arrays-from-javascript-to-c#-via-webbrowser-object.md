---
title: Pass arrays from JavaScript to C# via WebBrowser object
tags: [xaml, c#, wpf, javascript, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/01cf9667-b420-4c82-a2ba-18b996eac7e2/pass-arrays-from-javascript-to-c-via-webbrowser-object?forum=wpf
---
# Pass arrays from JavaScript to C# via WebBrowser object
*> pass all changed properties in an array*

in JavaScript, objects and arrays are almost identical to each other.
so we can simply use object+properties and dynamic.
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  Width="800" Height="400"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <Grid>
    <WebBrowser x:Name="wb" />
  </Grid>
</Window>
```
```c#
using System.Runtime.InteropServices;
using System.Windows;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      wb.Navigated += (s, e) => wb.ObjectForScripting = new Proxy();
      wb.NavigateToString(@"
      <html>
      <head>
        <script language='javascript'>
        function test()
        {
          var arr = {color:'cyan', top_speed: '200 mph'};
          alert(window.external.getname(arr));
        }
        </script>
      </head>
      <body>hello <button onclick='test()'>test</button></body>
      </html>");
    }
  }

  [ComVisible(true)]
  public class Proxy
  {
    public string GetName(dynamic d)
    {
      var color = (string)d.color;
      var speed = (string)d.top_speed;
      return color + " " + speed;
    }
  }
}
```
