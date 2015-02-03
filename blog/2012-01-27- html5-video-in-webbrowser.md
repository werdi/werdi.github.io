---
title: Video in HTML5 won't play in webbrowser control... 
tags: [c#, xaml, wpf, html]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ffbdedd8-7194-4aa6-b4b6-b8f4fdb453cf/video-in-html5-wont-play-in-webbrowser-control?forum=wpf
---
# Video in HTML5 won't play in webbrowser control...
*> I am using the webbrowser control for my app, and have the source set to a local HTML5 document with a video tag. The video plays in IE, but not in the webbrowser control within my WPF app.*

take a look at the example [here] (http://social.msdn.microsoft.com/Forums/en-us/wpf/thread/87e41b0d-145d-4438-958e-c8b3a0a969d3/#f64a413c-9d3a-4fef-88cf-27db4652e810). works fine in a WPF WebBrowser

*> In the example you gave, they are embedding the HTML in C# in the application itself. I am not doing that*

there is no difference. you can load html from a file.
below is an example. works fine too.
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="250" Width="250" FontSize="16">
  <WebBrowser x:Name="wb" />
</Window>
```
```c#
using System;
using System.IO;
using System.Windows;

namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      wb.Source = GetPageUri();
    }

    Uri GetPageUri()
    {
      // create html-file; this is used for the demo only
      var file = @"C:\Users\" + Environment.UserName + @"\AppData\LocalLow\Temp\testpage.htm";
      File.WriteAllText(file, @"<!DOCTYPE html>
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
      return new Uri(file);
    }
  }
}
```
