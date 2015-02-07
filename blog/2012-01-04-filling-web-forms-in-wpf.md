---
title: Filling web forms in WPF
tags: [c#, xaml, wpf, web browser]
src: https://social.msdn.microsoft.com/Forums/ru-RU/69f1a75e-1c45-48c0-92b7-0796db958320/filling-web-forms-in-wpf?forum=wpf
---
# Filling web forms in WPF
*> But how to fill out a web form in WPF. Just do not haveto WPF elements GetElementById etc ...*
```c#
use dynamic:
dynamic d = webBrowser1.Document;
var el = d.GetElementsByTagName("input");

or add reference to the Microsoft.mshtml.dll
and use something like that: 
var els = ((mshtml.IHTMLDocument3)webBrowser1.Document).getElementsByTagName("input");
```
*> An error, what am I doing wrong?*

instead of SetAttribute use value property. and instead of InvokeMember use click method.
below is an example.
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="450" Width="900">
  <Grid>
    <Button Content="Test" Click="Button_Click" VerticalAlignment="Top" 
        HorizontalAlignment="Left" x:Name="btn" IsEnabled="False" />
    <WebBrowser x:Name="wb" Margin="0,30,0,0"/>
  </Grid>
</Window>
```
```c#
using System.Windows;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      wb.Navigate("http://bing.com");
      wb.Navigated += (s, e) => btn.IsEnabled = true;
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      dynamic d = wb.Document;
      if (d.readyState == "complete")
      {
        btn.IsEnabled = false;
        d.getElementById("sb_form_q").value = "malobukv";
        d.getElementById("sb_form_go").click();
      }
    }
  }
}
```
