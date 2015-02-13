---
title: Render HTML5 content in WPF frame control
tags: [c#, xaml, wpf, html5]
src: https://social.msdn.microsoft.com/Forums/ru-RU/87e41b0d-145d-4438-958e-c8b3a0a969d3/render-html5-content-in-wpf-frame-control?forum=wpf
---
# Render HTML5 content in WPF frame control
*> I'm trying to render an HTML5 page in the frame control in a WPF app, but somehow this doesn't seem to work.*

use WebBrowser for html5 rendering   
```
<WebBrowser x:Name="myWebBrowser" />
```
*> I've tried the same page with a standard IE9 browser (outside of a WPF application) and the video is displayed correctly.*

could you please provide html which works in IE, but doesn't in WebBrowser?
i'm wondering about !DOCTYPE and about html tag for the player.

*> I was wondering whether anyone knew of a problem with Frame and WebBrowser controls and HTML5 video embedding, but I guess it is a very specific constellation and not something that a lot of people have an issue with.*

there is not problem. just remember to specify the following tag:
```   
<meta http-equiv='X-UA-Compatible' content='IE=9' /> 
```
below is an example. works fine for me.
```xaml
<Window x:Class="WpfApplication7.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication7"
  Title="MainWindow" Height="500" Width="550" FontSize="16">
</Window>
```
```c#
using System.Windows;
using System.Windows.Controls;

namespace WpfApplication7
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      var wb = new WebBrowser();
      this.Content = wb;
      wb.NavigateToString(@"
        <!DOCTYPE html>
        <html>
        <head>
          <meta http-equiv='Content-Type' content='text/html; charset=unicode' />
          <meta http-equiv='X-UA-Compatible' content='IE=9' /> 
          <title></title>
        </head>
        <body>
          <div>
             <video autoplay='autoplay' preload='metadata'>
                <source src='http://html5demos.com/assets/dizzy.mp4' type='video/mp4' />
            </video>
          </div>
        </body>
        </html>");
    }
  }
}
```