---
title: Displaying a specific area of a webpage
tags: [c#, xaml, web, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/11974aed-73e8-471c-898a-ac243e1d23f7/displaying-a-specific-area-of-a-webpage?forum=wpf
---
# Displaying a specific area of a webpage
*> I would like to be able to ignore some parts of the html and only display certain parts of the page. For example, using the html code below, I would like to display only the second div tag*
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication9"
  Title="MainWindow" Height="400" Width="280" FontSize="16">
  <Grid>
    <Button Content="Test" Click="Button_Click" VerticalAlignment="Top"
      HorizontalAlignment="Left" />
    <WebBrowser x:Name="wb" Margin="0,30,0,0" />
  </Grid>
</Window>
```
```c#
using System.Windows;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      wb.NavigateToString(@"
      <html>
        <head></head>
        <body>
          <div id=""ignore"">Hello</div>
          <div id=""display"">World</div>
        </body>
      </html>");
    }
    private void Button_Click(object sender, RoutedEventArgs e)
    {
      dynamic doc = wb.Document;
      dynamic div = doc.getElementById("display");
      doc.body.innerHTML = div.outerHTML;
    }
  }
}
```
*> Is there a way of loading the whole page*

sure, with a Navigate Method
```
wb.Navigate(new Uri("http://www.bing.com"));
```
*> ... keep all of the interactivity?*

interactivity could be complicated, if it based on javascript.
so there is no general answer.

*> Only problem with this is that it hides all of the child elements in "element_to_display".*

getElementsByTagName returns all tags from hierarchy.
so, at some point, elem is a parent of element_to_display. and "elem.style.display = "none";" hides elem and all descendants.

*> I've found that I can do it like the following, but then I have to know every element in the HTML that I do not want to show, instead of just knowing the element that I do want to show*

try using the following code
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication9"
  Title="MainWindow" Height="400" Width="1000" FontSize="16">
  <Grid>
    <Button Content="Test" Click="Button_Click" VerticalAlignment="Top" HorizontalAlignment="Left" />
    <WebBrowser x:Name="wb" Margin="0,30,0,0" />
  </Grid>
</Window>
```
```c#
using System;
using System.Collections;
using System.Windows;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      wb.Navigate(
        new Uri(
          "http://social.msdn.microsoft.com/Forums/en-us/wpf/thread/11974aed-73e8-471c-898a-ac243e1d23f7"));
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      dynamic doc = wb.Document;
      dynamic div = doc.getElementById("11974aed-73e8-471c-898a-ac243e1d23f7");
      doc.body.insertAdjacentElement("afterBegin", div);
      foreach(dynamic cn in doc.body.childNodes) cn.style.display = "none";
      div.style.display = "";
    }
  }
}
```